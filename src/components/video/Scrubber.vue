<script setup lang="ts">
import { computed, ref, onMounted, watchEffect } from 'vue'

const emit = defineEmits(['update', 'play', 'pause'])
const trackRef = ref()
const progressRef = ref(0)
const scrubProgress = ref(0)
const scrubbing = ref(false)
const seekTime = ref('')
const props = defineProps({
  timeRef: {
    type: Object,
    default: () => ({
      current: 0,
      total: 0,
    }),
  },
  progress: {
    type: Number,
    default: () => 0,
  },
  playStatus: {
    type: String,
    default: () => 'stopped'
  }
})

let wrapperWidth = 0

const formattedTimeRef = computed(() => {
  return { current: padTime(props.timeRef.current), total: padTime(props.timeRef.total) }
})

watchEffect(() => {
  if (!scrubbing.value) {
    progressRef.value = props.progress
  }
})

onMounted(() => {
  if (trackRef.value) {
    trackRef.value.addEventListener('click', getClickPosition, false)
    trackRef.value.addEventListener('mousemove', getDragPosition)
  }
})
const getClickPosition = (e: any) => {
  let target = e.target
  if (target.nodeType == 3) target = target.parentNode
  wrapperWidth = wrapperWidth || target.offsetWidth
  let seekWidth = e.offsetX
  scrubbing.value = false
  progressRef.value = (seekWidth / wrapperWidth) * 100
  scrubProgress.value = progressRef.value
  emit('update', progressRef.value)
}
const getDragPosition = (e: any) => {
  if (e.buttons === 1) {
    scrubbing.value = true
    let target = e.target
    if (target.nodeType == 3) target = target.parentNode
    wrapperWidth = wrapperWidth || target.offsetWidth
    let seekWidth = e.offsetX
    scrubProgress.value = (seekWidth / wrapperWidth) * 100
    if (scrubProgress.value > 100) scrubProgress.value = 100;
    else if (scrubProgress.value < 0) scrubProgress.value = 0;
    seekTime.value = padTime(Math.floor((scrubProgress.value / 100) * props.timeRef.total))
  } else {
    scrubbing.value = false
  }
}

const padTime = (seconds: number): string => {
  const min = Math.floor(seconds / 60)
  const sec = seconds % 60
  return `${min}:${sec.toString().padStart(2, '0')}`
}

const handleControl = () => {
  if (props.playStatus !== 'playing')
    emit('play', true)
  else emit('pause', true)
}
</script>

<template>
  <div class="controls">
    <div class="hide">
      <div class="control-options" :class="{playing: playStatus === 'playing', paused: playStatus !== 'playing'}" @click="handleControl"></div>
      <span class="time-current">{{ formattedTimeRef.current }}</span>
      <div class="scrubber">
        <div class="track" ref="trackRef">
          <div class="slider" :style="'width:' + (scrubbing ? scrubProgress : progressRef) + '%'">
            <span class="scrub-value" v-if="scrubbing">{{ seekTime }}</span>
          </div>
        </div>
      </div>
      <span class="time-total">{{ formattedTimeRef.total }}</span>
    </div>
  </div>
</template>
<style scoped lang="scss">
.controls {
  height: 25px;
  width: 100%;
  display: flex;
  position: absolute;
  bottom: 0;
  left: 0;
  background: RGBA(44, 62, 80, 0.4);

  .control-options {
    text-align: center;
    width: 25px;
    height: 25px;
    display: block;
    position: absolute;
    left: 3px;
    line-height: 25px;
    top: 0;
    font-size: 14px;
    cursor: pointer;
    &.paused::before {
      content: '▶️';
    }

    &.playing::before {
        content: '⏸';
    }
  }

  .hide {
    display: flex;
    color: #fff;

    .time-current {
      position: absolute;
      line-height: 14px;
      top: calc(50% - 7px);
      left: 30px;
    }

    .time-total {
      position: absolute;
      line-height: 14px;
      top: calc(50% - 7px);
      right: 10px;
    }

    .control-options {
    }

    .scrubber {
      background-color: RGBA(44, 62, 80, 0.5);

      .track {
        width: 78%;
        top: 45%;
        position: absolute;
        margin-left: 0;
        left: 12%;
        height: 5px;
        background-color: #c7d8e7;
        border-radius: 8px;

        .slider {
          height: 100%;
          width: 0;
          position: relative;
          background-color: #2c3e50;

          &::after {
            right: 0;
            background-color: #5f7081;
            content: '\f111';
            height: 10px;
            width: 10px;
            position: absolute;
            top: calc(50% - 5px);
            color: transparent;
            border-radius: 8px;
          }

          .scrub-value {
            background-color: RGBA(150, 150, 150, 0.9);
            position: absolute;
            right: -12px;
            top: -25px;
            padding: 2px 3px;
            border-radius: 5px;
            font-size: 12px;
            line-height: 12px;
            user-select: none;
          }
        }
      }
    }
  }

  &:hover {
    .hide {
      display: flex;
      width: 100%;
      height: 100%;
    }
  }
}
</style>
