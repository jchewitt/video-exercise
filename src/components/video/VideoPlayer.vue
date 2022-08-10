<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from 'vue'
import videojs from 'video.js'
import 'videojs-youtube'
import Player = videojs.Player
import Scrubber from './Scrubber.vue'

type VideoData = {
  src: any
  duration: number
}

const videoRef = ref(null)
const videoIdx = ref(0)
const timeRef = ref({ current: 0, total: 0 })
const progress = ref(0)
const playStatus = ref('stopped')

const props = defineProps({
  options: {
    type: Object,
    default() {
      return {
        videos: [],
      }
    },
  },
})

const baseOptions = {
  techOrder: ['html5', 'youtube'],
  autoplay: false,
  controls: false,
}

let player: Player
let videoData: {
  videos: Array<VideoData>
} = { videos: [] }

onMounted(async () => {
  if (videoRef.value) {
    player = videojs(videoRef.value, baseOptions, () => {})
  }
  videoData.videos = await loadDurations(props.options.videos)
  setupPlayerEvents()
})

onBeforeUnmount(() => {
  if (player) {
    player.dispose()
  }
})

const setupPlayerEvents = () => {
  player.src(videoData.videos[videoIdx.value].src)
  player.on('ended', () => {
    videoIdx.value++
    if (videoIdx.value >= videoData.videos.length) videoIdx.value = 0
    player.src(videoData.videos[videoIdx.value].src)
    player.play()
  })
  player.on('timeupdate', () => {
    updateTimeRef()
  })
  player.on('play', () => {
    playStatus.value = 'playing'
  })
  player.on('pause', () => {
    playStatus.value = 'paused'
  })
}

const updateTimeRef = (offset?: number) => {
  if (!videoData.videos.length) {
    timeRef.value = { current: 0, total: 0 }
    return
  }
  const current = sumDurations(videoData.videos.slice(0, videoIdx.value), offset || player.currentTime())
  const total = sumDurations(videoData.videos)
  timeRef.value = { current, total }
  progress.value = (current / total) * 100
}

const sumDurations = (videos: Array<VideoData>, beginningDuration: number = 0): number => {
  if (videos.length) {
    beginningDuration += videos.reduce((dur, video) => dur + video.duration, 0)
  }
  return parseInt(beginningDuration.toFixed(0))
}

const loadDurations = async (videos: Array<{ src: string; type: string }>): Promise<Array<VideoData>> => {
  const elVid = document.createElement('video')
  elVid.style.display = 'none'
  document.body.appendChild(elVid)
  const dataPlayer = videojs(elVid, { ...baseOptions, controls: true })
  const results = []
  for (let i = 0; i < videos.length; i++) {
    const src = videos[i]
    await loadVideoSource(dataPlayer, src)
    results.push({ src, duration: dataPlayer.duration() })
  }
  dataPlayer.dispose()
  elVid.remove()
  return results
}

const updateTime = async (time: any) => {
  time = parseInt(((time / 100) * timeRef.value.total).toString())
  let curTime = 0
  let curIdx: number
  for (curIdx = 0; curIdx < videoData.videos.length; curIdx++) {
    curTime += videoData.videos[curIdx].duration
    if (curTime >= time) break
  }
  let offset = curIdx === 0 ? time : time - sumDurations(videoData.videos.slice(0, curIdx))
  if (curIdx !== videoIdx.value) {
    videoIdx.value = curIdx
    updateTimeRef(offset)
    await loadVideoSource(player, videoData.videos[curIdx].src)
  }
  player.currentTime(offset.toString())
  player.play()
}

const loadVideoSource = async (player: Player, source: any) => {
  return new Promise((resolve) => {
    player.src(source)
    player.one('loadedmetadata', () => {
      resolve(player)
    })
  })
}

const play = () => {
  player.play()
}

const pause = () => {
  player.pause()
}
</script>
<template>
  <div class="wrapper">
    <video class="video-wrapper video-js" width="640" height="264" ref="videoRef"></video>
    <scrubber
      :timeRef="timeRef"
      :progress="progress"
      :playStatus="playStatus"
      @update="updateTime"
      @play="play"
      @pause="pause"
    ></scrubber>
  </div>
</template>
<style lang="scss">
.wrapper {
  position: relative;
  width: 640px;
}
</style>
