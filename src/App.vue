<script setup>
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";

const bearer = ref(
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwic3ViIjoxLCJyb2xlcyI6W3siaWQiOjEsIm5hbWUiOiJBRE1JTiJ9XSwiaWQiOjEsIm5hbWUiOm51bGwsIm9yZ0lkIjoxLCJwZXRzIjpbXSwiaWF0IjoxNjkyMTA2NDIxLCJleHAiOjE2OTIxNjY0MjF9.y5MzxSeGYSQ9kjBxHe_ZUmipD5vnwv9RSS-CsSDYag4"
);
let socketOptions = {
  transportOptions: {
    polling: {
      extraHeaders: {
        Authorization: bearer.value,
      },
    },
  },
};

let socket = io("http://localhost:3001", socketOptions);
// const socket2 = io("http://localhost:3003", socketOptions2);

const rooms = ref([]);
const roomName = ref("");
const currentRoom = ref(null);
const currentUsers = ref(null);
const messages = ref([]);
const messageText = ref("");
const joined = ref(false);
const username = ref("");
const typingDisplay = ref("");

const changeUser = () => {
  socketOptions = {
    transportOptions: {
      polling: {
        extraHeaders: {
          Authorization: bearer.value,
        },
      },
    },
  };
  socket = io("http://localhost:3001", socketOptions);
};

const findAllRooms = () => {
  socket.emit("findAllRooms", {}, (response) => {
    rooms.value = response;
  });
};

const newRoom = () => {
  socket.emit("createRoom", { name: roomName.value }, (room) => {
    roomName.value = "";
  });
};

const join = (room, id) => {
  socket.emit("joinRoom", { id: id }, (name) => {
    joined.value = true;
    // would normally be removed if i added authentication :p
    username.value = name;
    currentRoom.value = room;
    getRoomMessages(id);
  });
};

const getRoomMessages = (roomId) => {
  socket.emit("findAllRoomMessages", { roomId: roomId }, (response) => {
    console.log("hi");
    console.log(response);
    messages.value = response;
  });
};

const sendMessage = (room) => {
  socket.emit(
    "createMessage",
    { text: messageText.value, room: room },
    (response) => {
      messageText.value = "";
    }
  );
};

let timeout;
const emitTyping = () => {
  console.log(socket);
  socket.emit("typing", { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit("typing", { isTyping: false });
  }, 2000);
};

onBeforeMount(() => {
  findAllRooms();

  socket.on("message", (message) => {
    messages.value.push(message);
    console.log(messages.value);
  });

  socket.on("room", (room) => {
    rooms.value.push(room);
  });

  socket.on("typing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`;
    } else {
      typingDisplay.value = "";
    }
  });
});
</script>

<template class="flex-pos">
  <div class="row">
    <div class="col-12">
      <form @submit.prevent="changeUser">
        <label> change user? </label>
        <input v-model="bearer" />
        <button type="submit">Send</button>
      </form>
    </div>
    <div class="col-12">
      <div class="rooms">
        <div v-for="room in rooms" :key="room.id">
          <div class="card" style="width: 18rem">
            <div class="card-body">
              <h5 class="card-title">{{ room.name }}</h5>
              <h5 class="card-text">Current users:</h5>
              <div v-for="user in room.users" :key="user.id">
                <h5 class="card-text">{{ user.username }}</h5>
              </div>

              <p class="card-text">
                Join the room and get access to messaging with current users
              </p>
              <!-- <div v-if=""> -->
              <a @click="join(room, room.id)" class="btn btn-primary">Join</a>
              <!-- </div> -->
            </div>
          </div>
        </div>
        <!-- Create a new room -->
        <form @submit.prevent="newRoom">
          <label> New room: </label>
          <input v-model="roomName" />
          <button type="submit">Create</button>
        </form>
        <!-- // Create a new room -->
      </div>
    </div>
  </div>
  <div class="chat">
    <div v-if="!joined">
      <label> No room joined! </label>
    </div>
    <div class="chat-container" v-else>
      <h5 class="title cap">{{ currentRoom.name }}</h5>
      <div class="messages-container">
        <div
          class="mt-3 mb-1"
          v-bind:class="message.user.username !== username ? 'al-right' : null"
          v-for="message in messages"
          :key="message.user?.id"
        >
          <span class="chat-bg"
            ><span class="cap">[{{ message.user.username }}]:</span>
            {{ message.text }}</span
          >
        </div>
      </div>

      <div v-if="typingDisplay">{{ typingDisplay }}</div>

      <hr />

      <div class="message-input">
        <form @submit.prevent="sendMessage(currentRoom)">
          <label>Message: </label>
          <input v-model="messageText" @input="emitTyping" />

          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chat-bg {
  padding: 5px;
  margin-bottom: 10px;
  background-color: rgb(48, 123, 236);
  color: white;
  border-radius: 10px;
}
.al-right {
  text-align: right;
}
.cap {
  text-transform: capitalize;
}
.title {
  text-align: center;
}
.chat {
  padding: 20px;
  height: 100vh;
}
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  max-width: 80%;
}

.messages-container {
  max-height: 50%;
  overflow-y: scroll;
  flex: 1;
}
</style>
