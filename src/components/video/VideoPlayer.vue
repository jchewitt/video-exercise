<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, defineProps } from 'vue'
import videojs from 'video.js'
import 'videojs-youtube'
import Player = videojs.Player

const videoRef = ref(null)
const props = defineProps({
  options: {
    type: Object,
    default() {
      return {}
    },
  },
})
let player: Player

onMounted(() => {
  videojs.registerPlugin
  player = videojs(videoRef.value, props.options, () => {
    player.log('onPlayerReady', this)
  })
})

onBeforeUnmount(() => {
  if (player) {
    player.dispose()
  }
})
</script>
<template>
  <video ref="videoRef"></video>
</template>
