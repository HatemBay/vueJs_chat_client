<script setup>
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";

const socketOptions = {
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

const socket = io("http://localhost:3001", socketOptions);
// const socket2 = io("http://localhost:3003", socketOptions2);

const messages = ref([]);
const messageText = ref("");
const joined = ref(false);
const name = ref("");
const typingDisplay = ref("");

onBeforeMount(() => {
  socket.emit("findAllMessages", {}, (response) => {
    messages.value = response;
  });

  socket.on("message", (message) => {
    messages.value.push(message);
  });

  socket.on("typing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`;
    } else {
      typingDisplay.value = "";
    }
  });
});

const join = () => {
  socket.emit("join", { name: name.value }, () => {
    joined.value = true;
  });
};

const sendMessage = () => {
  socket.emit("createMessage", { text: messageText.value }, (response) => {
    messageText.value = "";
  });
};

let timeout;
const emitTyping = () => {
  socket.emit("typing", { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit("typing", { isTyping: false });
  }, 2000);
};
</script>

<template>
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
          [{{ name }}]: {{ message.text }}
        </div>
      </div>

      <div v-if="typingDisplay">{{ typingDisplay }}</div>

      <hr />

      <div class="message-input">
        <form @submit.prevent="sendMessage">
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
