<template>
  <div class="camera-overlay">
    <video ref="videoElement" autoplay></video>
    <button @click="capturePhoto" class="btn btn-success">Capturar</button>
    <button @click="$emit('close')" class="btn btn-danger">Cerrar</button>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue';

export default {
  emits: ['close'],
  setup() {
    const videoElement = ref(null);
    let stream = null;

    const startCamera = async () => {
      try {
        stream = await navigator.mediaDevices.getUserMedia({ video: true });
        videoElement.value.srcObject = stream;
      } catch (error) {
        console.error('Error al acceder a la cámara:', error);
      }
    };

    const capturePhoto = () => {
      const canvas = document.createElement('canvas');
      canvas.width = videoElement.value.videoWidth;
      canvas.height = videoElement.value.videoHeight;
      const ctx = canvas.getContext('2d');
      ctx.drawImage(videoElement.value, 0, 0);
      const photoUrl = canvas.toDataURL('image/png');
      console.log('Foto capturada:', photoUrl);
    };

    onMounted(startCamera);
    onUnmounted(() => {
      if (stream) {
        stream.getTracks().forEach(track => track.stop());
      }
    });

    return { videoElement, capturePhoto };
  },
};
</script>

<style>
.camera-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
</style>
