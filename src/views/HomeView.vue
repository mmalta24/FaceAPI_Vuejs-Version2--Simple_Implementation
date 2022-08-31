<template>
  <div class="home">
    <video autoplay class="feed"></video>
    <br>
    <button @click="checkEmotion">Clica</button>
    <p>{{result}}</p>
  </div>
</template>

<script>
// @ is an alias to /src
import * as faceapi from 'face-api.js';

export default {
  name: 'HomeView',

  data() {
    return {
      result: 'Experimenta'
    }
  },

  methods: {
    async init() {
      if('mediaDevices' in navigator && 'getUserMedia' in navigator.mediaDevices){
        navigator.mediaDevices.getUserMedia({video:true}).then(stream=>{
          const videoPlayer=document.querySelector("video");
          videoPlayer.srcObject=stream;
          videoPlayer.play();
        })
      }
      else{
        alert('Cannot get mediaDevices')
      }
    
    },

    async initIA(){
     await faceapi.nets.tinyFaceDetector.loadFromUri('models')
     await faceapi.nets.faceLandmark68Net.loadFromUri('models')
     await faceapi.nets.faceRecognitionNet.loadFromUri('models')
     await faceapi.nets.faceExpressionNet.loadFromUri('models')

    },

    async checkEmotion(){
      const video=document.querySelector("video")
      
        const detections=await faceapi.detectAllFaces(video,new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceExpressions()
        if (detections.length==0){
          this.result='Resultado: Afasta-te mais da câmera'
        }
        else{
          let result=Object.keys(detections[0].expressions).reduce(function(a, b){ return detections[0].expressions[a] > detections[0].expressions[b] ? a : b })
          this.result='Resultado: '+result;
        }
  
      
    }
  },

  beforeMount () {
    this.initIA().then(()=>{
      this.init();
    })
    
  },
}
</script>

<style scoped>
video
{
    transform: rotateY(180deg);
    -webkit-transform:rotateY(180deg); /* Safari and Chrome */
    -moz-transform:rotateY(180deg); /* Firefox */
}
</style>
