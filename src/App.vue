<script setup>
import AgoraRTM from 'agora-rtm-sdk';
import { v4 as uuidv4 } from 'uuid';
import { ref, onMounted, nextTick } from 'vue';

const APP_ID = '7c12fd1b7f9745ab8517fe4695805fe6';
const CHANNEL_NAME = 'agora-rtm-demo';

const client = AgoraRTM.createInstance(APP_ID);
const uid = uuidv4();
let channel;
const text = ref('');
const messages = ref([]);
const messagesRef = ref(null);



const appendMessage = async (message) => {
  messages.value.unshift(message);
  await nextTick();
  messagesRef.value.scrollTop = messagesRef.value.scrollHeight;
}


onMounted(async () => {
  await client.login({ uid, token: null});
  channel = await client.createChannel(CHANNEL_NAME);
  await channel.join();
  channel.on('ChannelMessage', (message, peerId) => {
    appendMessage({ text: message.text, uid: peerId})
  })
});



function sendMessage() {
  if (text.value === '') return;
  channel.sendMessage({ text: text.value, type: 'text' });
  appendMessage({ text: text.value, uid: uid})
  text.value = '';
}

</script>

<template>
  <div class="panel">
    <div class="messages" ref="messagesRef">
      <div class="inner">
        <div v-for="(message, index) in messages" :key="index" class="message">
          <div v-if="message.uid === uid" class="user-self">
            You: &nbsp;
          </div>
          <div v-else class="user-two">
            Surmur: &nbsp;
          </div>
          <div class="text">{{ message.text }}</div>
        </div>
      </div>
    </div>
    <form @submit.prevent="sendMessage">
          <input v-model="text"/>
          <button>+</button>
    </form>
  </div>
</template>

<style>
body {
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(
    90deg,
    rgb(171, 211, 243) 0%,
    rgb(127, 165, 161) 53%,
    rgb(103, 103, 103) 100%
  );
  height: 100vh;
  overflow: hidden;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  width: 650px;
}

.panel {
  display: flex;
  flex-direction: column;
  justify-content: space-between; /* Keep the form at the bottom */
  padding: 20px;
  background: rgba(255, 255, 255, 0.7);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  backdrop-filter: blur(4px);
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.18);
  width: 80%;
  height: 80%;
  overflow: hidden;
}

.messages {
  flex: 1;
  overflow-y: auto;
  background-color: white;
  border-top-left-radius: 5px;
  border-top-right-radius: 5px;
}

.inner {
  padding: 10px;
  display: flex;
  flex-direction: column-reverse;
}

.message {
  text-align: left;
  display: flex;
  margin-bottom: 6px;
  align-self: flex-start;
}

.user-self {
  color: green;
}

.user-two {
  color: red;
}

form {
  display: flex;
  align-items: center; 
}

input {
  flex: 1;
  border: none;
  padding: 8px;
  border-radius: 1px;
  border-color: aliceblue;
  outline: none;
  background-color: rgb(255, 255, 255);
}

button {
  border: none;
  outline: none;
  background-color: transparent;
  font-size: 24px;
  cursor: pointer;

}

.text {
  font-weight: 500;
  color: #080808;
}


</style>
