<template>
  <audio :src="songSrc" ref="audioRef" autoplay @play="playSong" @ended="ended"></audio>
  <input type="file" @change="inputChange" ref="fileRef" accept=".mp3" style="display: none;">
  <div id="upload" @click="upload">
    <div id="btn">点击上传</div>
    <span>{{ name }}</span>
  </div>
  <canvas id="canvas" ref="canvas"></canvas>
  <button @click="controllSong" class="controll_btn">
    <i :class="['iconfont', `icon-${isPlay ? 'zanting' : 'bofang'}`]"></i>
  </button>
  <div id="switch">
    <div id="bar" @click="changeState('bar')" :class="stateBtn === 'bar' ? 'drawing' : ''">柱形</div>
    <div id="wave" @click="changeState('wave')" :class="stateBtn === 'wave' ? 'drawing' : ''">波形</div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const fileRef = ref<any>()
const audioRef = ref<any>()
const songSrc = ref<string>()
const ac = ref<any>()
const canvas = ref<any>()
const isPlay = ref<Boolean>(false)
const stateBtn = ref<string>('bar')
const name = ref<String>('')
const analyser = ref<any>()
const bufferLength = ref<any>()
const dataArray = ref<any>()
const ctx = ref<any>()

const upload = () => {
  fileRef.value.click()
}

const inputChange = () => {
  const url = fileRef.value.files[0]
  name.value = url.name
  songSrc.value = URL.createObjectURL(url)
  isPlay.value = true
}

const ended = () => {
  isPlay.value = false
}

const controllSong = () => {
  if (songSrc.value) {
    isPlay.value = !isPlay.value
    if (isPlay.value) {
      audioRef.value?.play()
    } else {
      audioRef.value?.pause()
    }
  }
}

const playSong = () => {
  if (!bufferLength.value) {
    initAudio()
  }
}

const changeState = (state: string) => {
  stateBtn.value = state
}

const initAudio = () => {
  ac.value = new window.AudioContext()
  analyser.value = ac.value.createAnalyser()
  analyser.value.fftSize = 512
  bufferLength.value = analyser.value.fftSize / 2
  const source = ac.value.createMediaElementSource(audioRef.value)
  source.connect(analyser.value)
  analyser.value.connect(ac.value.destination)
  bufferLength.value = analyser.value.frequencyBinCount
  isPlay.value = true
  initCanvas()
}

const initCanvas = () => {
  canvas.value.width = canvas.value.clientWidth
  canvas.value.height = canvas.value.clientHeight
  ctx.value = canvas.value?.getContext('2d')
  requestAnimationFrame(renderCanavs)
}

const renderCanavs = () => {
  dataArray.value = new Uint8Array(bufferLength.value)
  analyser.value.getByteFrequencyData(dataArray.value)
  ctx.value.clearRect(0, 0, canvas.value.width, canvas.value.height)
  if (stateBtn.value === 'bar') {
    drawBar()
  } else {
    drawWave()
  }
  requestAnimationFrame(renderCanavs)
}

const drawWave = () => {
  ctx.value.lineWidth = 2
  ctx.value.strokeStyle = 'plum'
  ctx.value.beginPath()
  const sliceWidth = canvas.value.width / bufferLength.value
  dataArray.value.forEach((item: any, index: number) => {
    const y = canvas.value.height / 2 - item
    if (index === 0) {
      ctx.value.moveTo(sliceWidth * index, y)
    } else {
      ctx.value.lineTo(sliceWidth * index, y)
    }
  })
  ctx.value.lineTo(canvas.value.width, canvas.value.height / 2)
  ctx.value.stroke()
}

const drawBar = () => {
  const barWidth = canvas.value.width / bufferLength.value * 2.3
  dataArray.value.forEach((item: any, index: number) => {
    const barHeight = item
    const r = 50
    const g = 250 * (index / bufferLength.value)
    const b = barHeight + 25 * (index / bufferLength.value)
    ctx.value.fillStyle = `rgba(${r}, ${g}, ${b}, .7)`
    ctx.value.fillRect(barWidth * (index + 1), canvas.value.height / 2 - barHeight, barWidth, barHeight)
    ctx.value.fillRect(barWidth * (index + 1), canvas.value.height / 2, barWidth, barHeight)
  })
}
</script>

<style lang="scss">
$color: skyblue;
#upload {
  display: flex;
  align-items: center;
  position: absolute;
  top: 20%;
  left: 2%;
  #btn {
    position: relative;
    text-align: center;
    width: 100px;
    height: 40px;
    line-height: 40px;
    margin-right: 10px;
    cursor: pointer;
    color: $color;
    transition: all 0.3 ease-in-out;
    z-index: 2;
    &::before,
    &::after {
      content: "";
      position: absolute;
      width: 10px;
      height: 10px;
      border: 2px solid $color;
      transition: all 0.3s ease-in-out 0.3s;
    }
    &::before {
      top: 0;
      left: 0;
      border-bottom: 0;
      border-right: 0;
    }
    &::after {
      right: 0;
      bottom: 0;
      border-top: 0;
      border-left: 0;
    }
    &:hover {
      background-color: $color;
      color: #fff;
      box-shadow: 0 0 50px $color;
      transition-delay: 0.4s;
      &::before,
      &::after {
        width: 98px;
        height: 38px;
        transition-delay: 0s;
      }
    }
  }
  span {
    color: whitesmoke;
    font-weight: bolder;
    font-family: 'Simsun';
  }
}

#canvas {
  width: 100%;
  height: 99%;
}

.controll_btn {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  cursor: pointer;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  color: #2376b7;
  border: 2px solid #2376b7;
  background-color: rgba(0, 0, 0, 0);
  box-shadow: 0 0 15px #2376b7;
  z-index: 2;
}

#switch {
  position: absolute;
  top: 10px;
  right: 10px;
  display: flex;
  font-family: 'Simsun';
  #bar,
  #wave {
    background: rgba(255, 255, 255, .7);
    cursor: pointer;
    color: #000;
    padding: 2px 10px;
    box-shadow: 0 1px 2px gray, inset 0 2px 5px gray;
    font-style:oblique;
    &:active {
      box-shadow: inset 0 5px 5px gray;
    }
  }
  #bar {
    border-radius: 10px 0 0 10px;
  }
  #wave {
    border-radius: 0 10px 10px 0;
  }
}

.drawing {
  color: #fff !important;
  background: rgba(182, 43, 204, 0.7) !important;
}
</style>
