<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from "vue";
import videojs from "video.js";
import "videojs-youtube";
import Player = videojs.Player;
import Scrubber from "./Scrubber.vue";

type VideoData = {
  src: any
  duration: number
}

const videoRef = ref(null);
let videoIdx = ref(0);
const timeRef = ref({ current: 0, total: 0 });
const progress = ref(0);
const formattedTimeRef = computed(() => {
  return { current: padTime(timeRef.value.current), total: padTime(timeRef.value.total) };
});

const padTime = (seconds: number): string => {
  const min = Math.floor(seconds / 60);
  const sec = seconds % 60;
  return `${min}:${sec.toString().padStart(2, "0")}`;
};

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
  techOrder: ["html5", "youtube"],
  autoplay: false,
  controls: false
};

let player: Player;
let videoData: {
  videos: Array<VideoData>
} = { videos: [] };

onMounted(async () => {
  if (videoRef.value) {
    player = videojs(videoRef.value, baseOptions, () => {
    });
  }
  videoData.videos = await loadDurations(props.options.videos);
  setupPlayerEvents();
  updateTimeRef();
});

const setupPlayerEvents = () => {
  player.src(videoData.videos[videoIdx.value].src);
  player.on("ended", () => {
    videoIdx.value++;
    if (videoIdx.value >= videoData.videos.length) videoIdx.value = 0;
    player.src(videoData.videos[videoIdx.value].src);
    player.play();
  });
  player.on("timeupdate", () => {
    updateTimeRef();
  });
};

const updateTimeRef = () => {
  if (!videoData.videos.length) {
    timeRef.value = { current: 0, total: 0 };
    return;
  }
  let current = player.currentTime();
  if (videoIdx.value > 0) {
    current += videoData.videos
      .slice(0, videoIdx.value)
      .reduce((dur, video) => dur + video.duration, 0);
  }
  current = current.toFixed(0);
  console.log("vidx", videoIdx.value);
  console.log("current time", current);
  const total = parseInt(
    videoData.videos.reduce((dur, video) => dur + video.duration, 0).toString()
  );
  timeRef.value = { current, total };
  progress.value = (current / total) * 100;
};

const loadDurations = async (videos: Array<{ src: string; type: string }>): Promise<Array<VideoData>> => {
  const elVid = document.createElement("video");
  elVid.style.display = "none";
  document.body.appendChild(elVid);
  const dataPlayer = videojs(elVid, { ...baseOptions, controls: true });
  const results = [];
  for (let i = 0; i < videos.length; i++) {
    const src = videos[i];
    const duration = await getDuration(dataPlayer, src);
    results.push({ src, duration });
  }
  dataPlayer.dispose();
  elVid.remove();
  return results;
};

const getDuration = async (dataPlayer: Player, videoSource: { src: string; type: string }): Promise<number> => {
  return new Promise((resolve) => {
    dataPlayer.src(videoSource);
    dataPlayer.one("loadedmetadata", () => {
      resolve(dataPlayer.duration());
    });
  });
};

onBeforeUnmount(() => {
  if (player) {
    player.dispose();
  }
});

const updateTime = (time: any) => {
  player.off("timeupdate");
  player.pause();
  time = parseInt(((time / 100) * timeRef.value.total).toString());
  timeRef.value.current = time;
  let curTime = 0;
  let curIdx: number;
  for (curIdx = 0; curIdx < videoData.videos.length; curIdx++) {
    curTime += videoData.videos[curIdx].duration;
    if (curTime >= time) break;
  }
  timeRef.value.current = time;
  console.log("idx", curIdx);
  console.log("time", time);
  let durationToVideo = (curIdx === 0) ? time : videoData.videos.slice(0, curIdx).reduce((dur, video) => dur + video.duration, 0);
  durationToVideo = parseInt(durationToVideo.toString());
  let offset = (curIdx === 0) ? time : time - videoData.videos.slice(0, curIdx).reduce((dur, video) => dur + video.duration, 0);
  offset = parseInt(offset.toString());
  console.log("durationUpTo", durationToVideo);
  console.log("videoIdx", videoIdx.value);
  if (curIdx !== videoIdx.value) {
    videoIdx.value = curIdx;
    player.src(videoData.videos[curIdx].src);
  }
  console.log("offset", offset);
  player.currentTime(offset);
  player.play()
  player.on("timeupdate", () => {
    updateTimeRef();
  });
};

const play = () => {
  player.play();
};

const pause = () => {
  player.pause();
};
</script>
<template>
  <div class="wrapper">
    <video class="video-wrapper video-js" width="640" height="264" ref="videoRef"></video>
    <scrubber :timeRef="timeRef" :progress="progress" @update="updateTime" @play="play" @pause="pause"></scrubber>
  </div>
  <button @click="play">play</button>
  <button @click="pause">pause</button>
</template>
<style lang="scss">
.wrapper {
  position: relative;
  width: 640px;
}
</style>
