<template>
  <div class="panel">
    <div class="messages">
      <div class="inner">
        <div v-for="(message, index) in messages" :key="index" class="message">
          <div v-if="message.uid === uid" class="user-self">
            You
            <div class="fon">
              <div class="text">{{ message.text }}</div>
            </div>
          </div>
          <div v-else class="user-them">
            Them
            <div class="fon">
              <div class="text">{{ message.text }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <form @submit.prevent="sendMessage">
      <input v-model="text" />
      <button>Send</button>
    </form>
  </div>
</template>

<script setup>
import AgoraRTM from 'agora-rtm-sdk';
import { v4 as uuidv4 } from 'uuid';
import { ref, onMounted } from 'vue';

const APP_ID = '452f99a0814b44d29d9a446ec20356fc';
const CHANEL = 'wdj';
const client = AgoraRTM.createInstance(APP_ID);
const uid = uuidv4();
let channel;
const text = ref('');
const messages = ref([]);

const appendMessage = async (message) => {
  messages.value.push(message);
};

onMounted(async () => {
  await client.login({ uid, token: null });
  channel = await client.createChannel(CHANEL);
  await channel.join();
  channel.on('ChannelMessage', (message, peerID) => {
    appendMessage({ text: message.text, uid: peerID });
  });
});

function sendMessage() {
  if (text.value === '') return;
  channel.sendMessage({ text: text.value, type: 'text' });
  appendMessage({ text: text.value, uid });
  text.value = '';
}
</script>

<style>
body {
  position: fixed;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.panel {
  color: white;
  background-image: url(../public/fon.jpg);
  background-size: cover;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  height: 90vh;
  width: 100vw;
  /* height: 844px;
  width: 390px; */
}

.inner {
  display: flex;
  flex-direction: column;
  gap: 5px;
  width: auto;
  margin-top: 5px;
}

.messages {
  width: 100%;
  overflow-y: scroll;
}

.message {
  /* display: flex; */
  /* justify-content: space-between; */
  margin: 0 5px 0 5px;
}

.user-self {
  color: red;
  display: flex;
  justify-content: space-between;
}

.user-them {
  color: rgb(255, 255, 0);
  display: flex;
  flex-direction: row-reverse;
  justify-content: space-between;
}

.fon {
  display: inline-block;
  width: auto;
  padding: 10px;
  background-color: rgba(77, 77, 77, 0.9);
  border-radius: 10px 0 10px 10px;
}

.text {
  display: flex;
  flex-wrap: wrap;
  background: 150%;
  color: white;
}

form {
  position: relative;
  display: flex;
  width: 100%;
  margin-bottom: 20px;
}

input {
  width: 100%;
  border: 2px solid white;
  border-radius: 10px;
  color: white;
  background: none;
  outline: none;
  height: 30px;
  font-size: 20px;
}

button {
  border: 2px solid white;
  border-radius: 10px;
  color: white;
  background: none;
  outline: none;
  font-size: 20px;
}
</style>
