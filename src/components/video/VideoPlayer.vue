<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from "vue";
import videojs from "video.js";
import "videojs-youtube";
import Player = videojs.Player;

type VideoData = {
  src: any,
  duration: number
}

const videoRef = ref(null);
let videoIdx = ref(0);
const timeRef = ref({ current: 0, total: 0 })

const formattedTimeRef = computed(() => {
  return { current: padTime(timeRef.value.current), total: padTime(timeRef.value.total) }
})

const padTime = (seconds: number): string => {
  const min = Math.floor(seconds / 60)
  const sec = seconds % 60
  return `${min}:${sec.toString().padStart(2, '0')}`
}

const props = defineProps({
  options: {
    type: Object,
    default() {
      return {
        videos: []
      };
    }
  }
});

const baseOptions = {
  techOrder: ['html5', 'youtube'],
  autoplay: false,
  controls: true
}

let player: Player;
let videoData: {
  videos: Array<VideoData>
} = {videos: []};

onMounted(async () => {
  if (videoRef.value) {
    player = videojs(videoRef.value, baseOptions, () => {
    });
  }
  videoData.videos = await loadDurations(props.options.videos)
  setupPlayerEvents()
  updateTimeRef()
});

const setupPlayerEvents = () => {
  player.src(videoData.videos[videoIdx.value].src)
  player.on('ended', () => {
    videoIdx.value++;
    if (videoIdx.value >= videoData.videos.length) videoIdx.value = 0
    player.src(videoData.videos[videoIdx.value].src)
    player.play()
  })
  player.on('timeupdate', () => {
    updateTimeRef()
  })
}

const updateTimeRef = () => {
  if (!videoData.videos.length) {
    timeRef.value = { current: 0, total: 0}
    return
  }
  let current = player.currentTime()
  if (videoIdx.value > 0) {
    current += videoData.videos.slice(0, videoIdx.value - 1).reduce((dur, video) => dur + video.duration, videoData.videos[0].duration)
  }
  current = parseInt(current.toString())
  const total = parseInt(
    videoData.videos.reduce(
      (dur, video) =>
        dur + video.duration, videoData.videos[0].duration
    ).toString())
  timeRef.value = { current, total }
}

const loadDurations = async (videos: Array<{src: string, type: string}>): Promise<Array<VideoData>> => {
  const elVid = document.createElement('video')
  elVid.style.display = 'none'
  document.body.appendChild(elVid)
  const dataPlayer = videojs(elVid, { ...baseOptions, controls: true })
  const results = []
  for (let i = 0; i < videos.length; i++) {
    const src = videos[i]
    const duration = await getDuration(dataPlayer, src)
    results.push({src, duration})
  }
  dataPlayer.dispose()
  elVid.remove()
  return results
}

const getDuration = async (dataPlayer: Player, videoSource: {src: string, type: string}): Promise<number> => {
  return new Promise((resolve) => {
    dataPlayer.src(videoSource)
    dataPlayer.one('loadedmetadata', () => {
      resolve(dataPlayer.duration());
    })
  })
}

onBeforeUnmount(() => {
  if (player) {
    player.dispose();
  }
});

const play = () => {
  player.play()
}

const pause = () => {
  player.pause()
}
</script>
<template>
  <video class="video-wrapper video-js" width="640" height="264" ref="videoRef"></video>
  <div>
    <span>{{ formattedTimeRef.current }} - {{ formattedTimeRef.total }}</span>
  </div>
  <button @click="play">play</button><button @click="pause">pause</button>
</template>
<style lang="scss">
</style>
