<script setup lang="ts">
import { ref } from 'vue'
import VideoPlayer from './components/video/VideoPlayer.vue'
const videos = [
      {
        // src: 'https://www.youtube.com/watch?v=xjS6SftYQaQ',
        src: 'https://www.youtube.com/watch?v=7CVtTOpgSyY',
        type: 'video/youtube'
      },
      {
        src: 'https://d2zihajmogu5jn.cloudfront.net/bipbop-advanced/bipbop_16x9_variant.m3u8',
        type: 'application/x-mpegURL',
      }
]
const options = ref({
  videos
})
</script>

<template>
  <VideoPlayer :options="options"></VideoPlayer>
</template>

<style scoped>
</style>
