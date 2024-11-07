<template>
  <div>
    <h2>Video Call</h2>
    <div>
      <video ref="localVideo" autoplay muted playsinline></video>
      <video ref="remoteVideo" autoplay playsinline></video>
    </div>
    <div>
      <button @click="startCall">Start Call</button>
      <input v-model="remotePeerId" placeholder="Enter Remote Peer ID" />
      <button @click="connectToPeer">Connect to Peer</button>
    </div>
    <p v-if="myPeerId">Your Peer ID: {{ myPeerId }}</p>
  </div>
</template>

<script setup>
import Peer from "peerjs";

const peer = ref(null); // объект PeerJS
const myPeerId = ref(""); // локальный Peer ID
const remotePeerId = ref(""); // ID удаленного пользователя
const localStream = ref(null); // локальный медиапоток
const call = ref(null); // объект звонка
const localVideo = ref(null); // ссылка на видеоэлемент для локального видео
const remoteVideo = ref(null); // ссылка на видеоэлемент для удаленного видео

// Инициализация PeerJS
const initPeer = () => {
  peer.value = new Peer();

  // Получаем ID после успешного подключения к серверу
  peer.value.on("open", (id) => {
    myPeerId.value = id;
    console.log(`My Peer ID: ${id}`);
    alert(`Your Peer ID is: ${id}`);
  });

  // Обработка входящих вызовов
  peer.value.on("call", (incomingCall) => {
    answerCall(incomingCall);
  });
};

// Получаем доступ к камере и микрофону
const getLocalStream = async () => {
  try {
    localStream.value = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true,
    });
    localVideo.value.srcObject = localStream.value;
  } catch (error) {
    console.error("Error accessing media devices.", error);
  }
};

// Начало вызова к удаленному ID
const startCall = () => {
  if (!remotePeerId.value) {
    alert("Please enter a Peer ID to call");
    return;
  }

  // Создаем вызов к удаленному ID
  call.value = peer.value.call(remotePeerId.value, localStream.value);
  call.value.on("stream", (remoteStream) => {
    remoteVideo.value.srcObject = remoteStream;
  });
  call.value.on("close", () => {
    console.log("Call ended");
  });
};

// Ответ на входящий вызов
const answerCall = (incomingCall) => {
  incomingCall.answer(localStream.value);
  incomingCall.on("stream", (remoteStream) => {
    remoteVideo.value.srcObject = remoteStream;
  });
  call.value = incomingCall;
};

// Подключение к удаленному ID
const connectToPeer = () => {
  if (!remotePeerId.value) {
    alert("Please enter the remote peer ID");
    return;
  }
  startCall();
};

// Очистка ресурсов при уничтожении компонента
onBeforeUnmount(() => {
  if (call.value) call.value.close();
  if (peer.value) peer.value.destroy();
  if (localStream.value) {
    localStream.value.getTracks().forEach((track) => track.stop());
  }
});

// Инициализация при монтировании
onMounted(() => {
  initPeer();
  getLocalStream();
});
</script>

<style scoped>
video {
  width: 45%;
  margin: 5px;
  border: 1px solid #ddd;
  border-radius: 8px;
  transform: scaleX(-1);
}
</style>
