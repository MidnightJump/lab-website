<template>
  <div class="video-player-wrapper">
    <video 
      ref="videoRef"
      :src="videoSrc"
      :class="videoClass"
      :autoplay="autoplay"
      :loop="loop"
      :muted="muted"
      :style="{ objectFit: objectFit }"
      playsinline
      @click="onVideoClick"
      @ended="onVideoEnded"
    ></video>
    <el-button
      v-show="showControls"
      class="custom-play-button"
      type="primary"
      circle
      @click.stop="toggleVideo"
    >
      <img 
        :src="isPlaying ? pauseIconUrl : playIconUrl"
        alt="播放/暂停"
        style="width: 56px; height: 56px;"
      />
    </el-button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'
import { ElButton } from 'element-plus'

// Props
const props = defineProps({
  videoSrc: {
    type: String,
    required: true
  },
  videoClass: {
    type: String,
    default: ''
  },
  autoplay: {
    type: Boolean,
    default: false
  },
  loop: {
    type: Boolean,
    default: false
  },
  muted: {
    type: Boolean,
    default: false
  },
  objectFit: {
    type: String,
    default: 'cover' // 'cover' 或 'contain'
  }
})

// 响应式数据
const videoRef = ref(null)
const isPlaying = ref(props.autoplay) // 根据autoplay属性初始化播放状态
const showControls = ref(true) // 控制图标显示/隐藏
const hideControlsTimer = ref(null) // 隐藏控制器的定时器

// 图标URL
const playIconUrl = new URL('../assets/images/projects/video-play.svg', import.meta.url).href
const pauseIconUrl = new URL('../assets/images/projects/video-pause.svg', import.meta.url).href

// 显示控制器
const showControlsTemporarily = () => {
  showControls.value = true
  
  // 清除之前的定时器
  if (hideControlsTimer.value) {
    clearTimeout(hideControlsTimer.value)
  }
  
  // 如果视频正在播放，1.5秒后隐藏控制器
  if (isPlaying.value) {
    hideControlsTimer.value = setTimeout(() => {
      showControls.value = false
    }, 1500)
  }
}

// 切换视频播放状态
const toggleVideo = () => {
  console.log('toggleVideo called, videoRef:', videoRef.value)
  if (videoRef.value) {
    const video = videoRef.value
    console.log('Video element found, paused:', video.paused)
    try {
      if (video.paused) {
        console.log('Starting video playback...')
        const p = video.play()
        if (p && typeof p.then === 'function') {
          p.then(() => { 
            console.log('Video started playing')
            isPlaying.value = true
            showControlsTemporarily() // 播放后显示控制器
          }).catch((e) => {
            console.warn('视频播放失败:', e)
            isPlaying.value = false
          })
        } else {
          console.log('Video started playing (sync)')
          isPlaying.value = true
          showControlsTemporarily() // 播放后显示控制器
        }
      } else {
        console.log('Pausing video...')
        video.pause()
        isPlaying.value = false
        showControls.value = true // 暂停时显示控制器
        // 清除隐藏定时器
        if (hideControlsTimer.value) {
          clearTimeout(hideControlsTimer.value)
        }
      }
    } catch (e) {
      console.warn('视频控制失败:', e)
      isPlaying.value = false
    }
  } else {
    console.warn('Video element not found')
  }
}

// 视频播放结束
const onVideoEnded = () => {
  isPlaying.value = false
  showControls.value = true // 播放结束时显示控制器
  // 清除隐藏定时器
  if (hideControlsTimer.value) {
    clearTimeout(hideControlsTimer.value)
  }
}

// 视频点击事件
const onVideoClick = () => {
  console.log('Video clicked')
  showControlsTemporarily() // 点击视频时显示控制器
  toggleVideo() // 切换播放状态
}

// 组件挂载时初始化
onMounted(async () => {
  console.log('VideoPlayer mounted, videoRef:', videoRef.value)
  
  // 等待DOM更新
  await nextTick()
  
  console.log('After nextTick, videoRef:', videoRef.value)
  
  // 如果设置了autoplay，初始化时显示控制器
  if (props.autoplay) {
    showControlsTemporarily()
  }
  
  // 确保视频元素存在后，添加事件监听器
  if (videoRef.value) {
    const video = videoRef.value
    console.log('Video element ready:', video)
    
    // 监听视频播放事件
    video.addEventListener('play', () => {
      console.log('Video play event fired')
      isPlaying.value = true
    })
    
    // 监听视频暂停事件
    video.addEventListener('pause', () => {
      console.log('Video pause event fired')
      isPlaying.value = false
    })
  } else {
    console.warn('Video element not found after nextTick')
  }
})

// 组件卸载时清理定时器
onUnmounted(() => {
  if (hideControlsTimer.value) {
    clearTimeout(hideControlsTimer.value)
  }
})

// 暴露方法给父组件
defineExpose({
  toggleVideo,
  videoRef
})
</script>

<style scoped lang="scss">
.video-player-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  border-radius: 16px;
  
  video {
    width: 100%;
    height: 100%;
    max-width: 100%;
    object-fit: cover;
  }
  
  .custom-play-button {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 56px;
    height: 56px;
    border-radius: 50%;
    border: none !important;
    box-shadow: none !important;
    --el-button-bg-color: transparent;
    --el-button-hover-bg-color: transparent;
    --el-button-active-bg-color: transparent;
    --el-button-border-color: transparent;
    color: white;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.3s ease;
    z-index: 5;
    opacity: 1;
    
    &:hover {
      background: transparent;
      transform: translate(-50%, -50%) scale(1.05);
    }
  }
}
</style>
