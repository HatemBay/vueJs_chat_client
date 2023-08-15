<script setup>
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";

const socketOptions3 = {
  transportOptions: {
    polling: {
      extraHeaders: {
        Authorization:
          "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwic3ViIjoxLCJyb2xlcyI6W3siaWQiOjEsIm5hbWUiOiJBRE1JTiJ9XSwiaWQiOjEsIm5hbWUiOm51bGwsIm9yZ0lkIjoxLCJwZXRzIjpbXSwiaWF0IjoxNjkyMDE3OTQzLCJleHAiOjE2OTIwNzc5NDN9.BkVt_s4i-xw87UMQr2JrDrSqmWvgllC5BS55YWmgLCM", // 'Bearer h93t4293t49jt34j9rferek...'
      },
    },
  },
};
const socketOptions2 = {
  transportOptions: {
    polling: {
      extraHeaders: {
        Authorization:
          "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluMyIsInN1YiI6MTYsInJvbGVzIjpbeyJpZCI6MSwibmFtZSI6IkFETUlOIn1dLCJpZCI6MTYsIm5hbWUiOm51bGwsIm9yZ0lkIjoxLCJwZXRzIjpbXSwiaWF0IjoxNjkyMDI4MTM0LCJleHAiOjE2OTIwODgxMzR9.xzhiYcYD6UcIFSJ16Kjc9UhYDsOitZhO4eAz2oQ2tsI",
      },
    },
  },
};
const socketOptions = {
  transportOptions: {
    polling: {
      extraHeaders: {
        Authorization:
          "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwic3ViIjoxLCJyb2xlcyI6W3siaWQiOjEsIm5hbWUiOiJBRE1JTiJ9XSwiaWQiOjEsIm5hbWUiOm51bGwsIm9yZ0lkIjoxLCJwZXRzIjpbXSwiaWF0IjoxNjkyMTA2NDIxLCJleHAiOjE2OTIxNjY0MjF9.y5MzxSeGYSQ9kjBxHe_ZUmipD5vnwv9RSS-CsSDYag4",
      },
    },
  },
};

const socket = io("http://localhost:3001", socketOptions);
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

<template>
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

  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label> what's thy name? </label>
        <input v-model="name" />
        <button type="submit">Send</button>
      </form>
    </div>
    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages" :key="message.user?.id">
          [{{ username }}]: {{ message.text }}
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
.chat {
  padding: 20px;
  height: 100vh;
}
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.messages-container {
  flex: 1;
}
</style>
