<script setup>
import { ref } from 'vue'
import { useAVLine } from 'vue-audio-visual'
import axios from 'axios'

const player = ref(null)
const canvas = ref(null)
let mySource = ref(null)
let action = ref('')
let output = ref('')

useAVLine(player, canvas, { src: mySource, canvHeight: 300, canvWidth: 1000, barColor: 'lime' })

const runSpeechRecognition = () => {
  var SpeechRecognition = SpeechRecognition || webkitSpeechRecognition;
  var recognition = new SpeechRecognition();

  recognition.onstart = () => { action.value = "*Escuchando tu pregunta...*" };
  
  recognition.onspeechend = () => {
    action.value = "*dejando de escuchar*";
    recognition.stop();
  }
  
  recognition.onresult = async (event) => {
    var transcript = event.results[0][0].transcript;
    output.value = transcript

    try {
      let res = await axios.post('http://localhost:4001/api/text-to-audio-file', {
        text: event.results[0][0].transcript
      })

      if (res.data) {
        mySource.value = '/voice/' + res.data + '.mp3'
        setTimeout(() => { player.value.play() }, 500)
      }
    } catch (err) {
      console.log(err)
    }
  };
  recognition.start();
}

const pauseAudio = () => {
  player.value.pause();
}
</script>

<template>
  <div>
    <audio 
      id="player" 
      ref="player" 
      :src="mySource" 
      type="audio/mpeg" 
      controls 
      hidden
    ></audio>

    <canvas ref="canvas" />
  </div>
  
  <div class="p-12 flex justify-evenly items-center">
    <button class="btn btn-outline btn-accent " type="button" @click="runSpeechRecognition()">Hacele una pregunta a Lupe</button>
    <button class="btn btn-outline btn-accent" type="button" @click="pauseAudio()">&#x2016;</button>
  </div>

  <div class="chat chat-start pl-12 pt-10" v-if="output">
    <div class="chat-image avatar">
      <div class="w-10 rounded-full">
        <img src="https://media.licdn.com/dms/image/D4D03AQGWzYhWa23HWA/profile-displayphoto-shrink_800_800/0/1683653155724?e=1698278400&v=beta&t=uoN7QSvS1m7d1uQoksaL2Vf87UdI-YzRh7q7Pskcl-U" />
      </div>
    </div>
    <div class="chat-header">
      Valentino Ghitti
    </div>
    <div class="chat-bubble" >{{ output }}</div>
  </div>

  <div class="chat chat-end pr-12 pb-12" v-if="action">
    <div class="chat-image avatar">
      <div class="w-10 rounded-full">
        <img src="https://vader.news/__export/1683381960183/sites/gadgets/img/2023/05/05/terminator-home.jpg_831511827.jpg" />
      </div>
    </div>
    <div class="chat-header">
      LupeGPT
    </div>
    <div class="chat-bubble" ><b>{{ action }}</b></div>
  </div>
</template>

<style>
body {
  background-color: rgb(23, 23, 23);
}

canvas {
  padding: 0;
  margin: auto;
  display: block;
  width: 800px;
  position: absolute;
  top: 100;
  bottom: 0;
  left: 0;
  right: 0;
}

</style>