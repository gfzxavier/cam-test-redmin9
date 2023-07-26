<template>
  <div id="app">
    Visão da câmera
    <CameraRecordingFm/>
  </div>
</template>

<script>
import CameraRecordingFm from './components/CameraRecordingFm.vue'

export default {
  name: 'App',
  components: {
    CameraRecordingFm
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
