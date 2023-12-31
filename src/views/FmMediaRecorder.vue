<template>
  <div class="camera-recording-fm">
    <div class="video-and-info">
      <video id="video" playsinline autoplay loop></video>
    </div>
    <div class="buttons">
      <button @click="toggleRecord">{{ buttonText }}</button>
      <button @click="changeCam">Alternar câmera</button>
    </div>
    <div class="mimes-menu">
      <button @click="changeMime(false)">-</button>
      Nº{{usedMimeIndex + 1}}
      <button @click="changeMime(true)">+</button>
    </div>
    <p>
      <span>
        MIME:<br>
      </span>
      {{ usedMime }}
    </p>
  </div>
</template>

<script>
export default {
  name: 'CameraRecordingFm',
  data() {
    return {
      
      maxWidth: 720,
      maxHeight: 1280,
      isMobile: true,
      cameraConfig: null,
      
      videoTagNode: null,

      stream: null,
      isRecording: false,
      mediaRecorder: null,
      recordedBlobs: [],

      usedMimeIndex: 0,

      mimesList: [
        'video/webm;codecs=vp8',
        'video/webm;codecs=vp9',
        'video/webm;codecs=vp8.0',
        'video/webm;codecs=vp9.0',
        'video/webm;codecs=h264',
        'video/webm;codecs=H264',
        'video/webm;codecs=avc1',
        'video/webm;codecs=vp8,opus',
        'video/webm;codecs=VP8,OPUS',
        'video/webm;codecs=vp9,opus', 
        'video/webm;codecs=vp8,vp9,opus',
        'video/webm;codecs=h264,opus',
        'video/webm;codecs=h264,vp9,opus',
        'video/x-matroska;codecs=avc1'
      ]
    }
  },
  beforeMount(){
    this.verifyDevice()
  },
  mounted(){
    this.videoTagNode = document.querySelector('#video')
    this.setCameraConfig()
    this.initCamera()
  },
  computed: {
    buttonText() {
      if(!this.isRecording) {
        return 'Gravar' 
      }
      return 'Parar gravação'
    },
    usedMime() {
      return this.mimesList[this.usedMimeIndex]
    }
  },
  methods: {
    async initCamera() {
      try {
        this.stream = await navigator.mediaDevices.getUserMedia(
          this.cameraConfig
        )
        this.videoTagNode.srcObject = this.stream
      } catch (error) {
        console.error('navigator.getUserMedia error:', error)
      }
    },
    async stopCamera() {
      const track = await this.getVideoTrack()
      await track.stop()
    },
    getVideoTrack() {
      let trackReturn
      this.videoTagNode.srcObject.getTracks().forEach(function (track) {
        if (track.kind === 'video') trackReturn = track
      })
      return trackReturn
    },
    setCameraConfig() {
      this.cameraConfig = {
        audio: false,
        video: {
          width: {
            max: this.maxWidth
          },
          height: {
            max: this.maxHeight
          },
          facingMode: this.frontalCamera ? 'user' : 'environment',
          aspectRatio: this.isMobile ? 16 / 9 : 9 / 16
        },
      }
    },
    verifyDevice() {
      const userAgent = window.navigator.userAgent
      const userAgentData = window.navigator.userAgentData
      if (
        (userAgentData != undefined && userAgentData.mobile) ||
        userAgent.match(/Mobile/i) ||
        userAgent.match(/IPhone/i)
      ) {
        this.isMobile = true
        return
      }
      this.isMobile = false
    },
    toggleRecord() {
      if(this.isRecording) {
        this.stopRecording()
        this.isRecording = !this.isRecording
        return
      }
      this.isRecording = true
      this.mediaRecorder = new MediaRecorder(this.stream, {
        mimeType: this.usedMime
      });
      this.mediaRecorder.start()
      console.log(this.mediaRecorder)
      this.mediaRecorder.ondataavailable = event => {
        console.log(this.mediaRecorder.mimeType)
        if (event.data && event.data.size > 0) {
          this.recordedBlobs.push(event.data)
          this.playRecordedVideo()
        }
      }
    },
    async changeCam() {
      this.frontalCamera = !this.frontalCamera
      await this.stopCamera()
      await this.setCameraConfig()
      this.initCamera()
    },
    playRecordedVideo() {
      const superBuffer = new Blob(this.recordedBlobs)
      this.stopCamera()
      this.videoTagNode.src = null
      this.videoTagNode.srcObject = null
      this.videoTagNode.src = window.URL.createObjectURL(superBuffer)
      this.videoTagNode.controls = true
      this.videoTagNode.play()
      console.log(superBuffer)
    },
    stopRecording() {
      this.mediaRecorder.stop()
    },
    changeMime(isNext){
      if(isNext && this.usedMimeIndex + 1 < this.mimesList.length) {
        this.usedMimeIndex ++
      } else if(!isNext && this.usedMimeIndex > 0) {
        this.usedMimeIndex --
      }
      this.mediaRecorder = null
      this.stream = null
      this.recordedBlobs = []
      this.setCameraConfig()
      this.initCamera()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.camera-recording-fm {
  display: flex;
  flex-direction: column;
  justify-content: center;
  max-width: 300px;
  margin: 0 auto;
  gap: 20px;
}
.buttons {
  margin: 0 auto;
}
.video-and-info{
  display: flex;
  justify-content: center;
}
span {
  font-weight: 700;
}
</style>
