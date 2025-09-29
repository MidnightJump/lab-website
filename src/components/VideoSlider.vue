<template>
  <div class="video-slider-container">
    <!-- 标题区域 -->
    <div class="slider-title">
      <h2>标题 -- 短视频滑动区</h2>
    </div>
    
    <!-- 视频滑动区域 -->
    <div class="video-slider">
      <div class="video-cards-container" :style="{ transform: `translateX(${translateX}px)` }">
        <!-- 左侧视频卡片 -->
        <div class="video-card video-card-small" :class="{ active: activeIndex === 0 }">
          <div class="video-area">
            <video 
              :ref="el => videoRefs[0] = el"
              class="video-player"
              :poster="videos[0].poster"
              preload="metadata"
              @click="playVideo(0)"
              @ended="onVideoEnded(0)"
            >
              <source :src="videos[0].src" type="video/mp4">
              您的浏览器不支持视频播放。
            </video>
            <button class="play-button" @click="playVideo(0)">
              <img 
                v-if="!playingStates[0]"
                :src="playIconUrl"
                alt="播放"
                style="width: 48px; height: 48px;"
              />
              <img 
                v-else
                :src="pauseIconUrl"
                alt="暂停"
                style="width: 48px; height: 48px;"
              />
            </button>
            <div class="video-overlay" v-if="!playingStates[0]" @click="playVideo(0)">
              <div class="video-title">{{ videos[0].title }}</div>
            </div>
          </div>
        </div>
        
        <!-- 中间视频卡片（主要） -->
        <div class="video-card video-card-large" :class="{ active: activeIndex === 1 }">
          <div class="video-area">
            <video 
              :ref="el => videoRefs[1] = el"
              class="video-player"
              :poster="videos[1].poster"
              preload="metadata"
              @click="playVideo(1)"
              @ended="onVideoEnded(1)"
            >
              <source :src="videos[1].src" type="video/mp4">
              您的浏览器不支持视频播放。
            </video>
            <button class="play-button" @click="playVideo(1)">
              <img 
                v-if="!playingStates[1]"
                :src="playIconUrl"
                alt="播放"
                style="width: 48px; height: 48px;"
              />
              <img 
                v-else
                :src="pauseIconUrl"
                alt="暂停"
                style="width: 48px; height: 48px;"
              />
            </button>
            <div class="video-overlay" v-if="!playingStates[1]" @click="playVideo(1)">
              <div class="video-title">{{ videos[1].title }}</div>
            </div>
          </div>
        </div>
        
        <!-- 右侧视频卡片 -->
        <div class="video-card video-card-small" :class="{ active: activeIndex === 2 }">
          <div class="video-area">
            <video 
              :ref="el => videoRefs[2] = el"
              class="video-player"
              :poster="videos[2].poster"
              preload="metadata"
              @click="playVideo(2)"
              @ended="onVideoEnded(2)"
            >
              <source :src="videos[2].src" type="video/mp4">
              您的浏览器不支持视频播放。
            </video>
            <button class="play-button" @click="playVideo(2)">
              <img 
                v-if="!playingStates[2]"
                :src="playIconUrl"
                alt="播放"
                style="width: 48px; height: 48px;"
              />
              <img 
                v-else
                :src="pauseIconUrl"
                alt="暂停"
                style="width: 48px; height: 48px;"
              />
            </button>
            <div class="video-overlay" v-if="!playingStates[2]" @click="playVideo(2)">
              <div class="video-title">{{ videos[2].title }}</div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- 分页指示器 -->
      <div class="pagination-indicator">
        <div class="indicator-dots">
          <span 
            v-for="(dot, index) in 6" 
            :key="index" 
            class="dot"
            :class="{ active: index === 2 }"
          ></span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'

// 响应式数据
const activeIndex = ref(1) // 默认中间卡片为活跃状态
const playingStates = ref([false, false, false]) // 播放状态

// 图标URL
const playIconUrl = new URL('../assets/images/projects/video-play.svg', import.meta.url).href
const pauseIconUrl = new URL('../assets/images/projects/video-pause.svg', import.meta.url).href

// 计算属性：计算滑动距离
const translateX = computed(() => {
  const cardWidth = 900 // 小卡片宽度
  const gap = 35 // 卡片间距
  const largeCardWidth = 1030 // 大卡片宽度
  
  if (activeIndex.value === 0) {
    return 0
  } else if (activeIndex.value === 1) {
    return -(cardWidth + gap)
  } else {
    return -(cardWidth + gap + largeCardWidth + gap)
  }
})

// 播放/暂停视频
const playVideo = (index) => {
  playingStates.value[index] = !playingStates.value[index]
  
  // 如果播放其他视频，暂停其他视频
  if (playingStates.value[index]) {
    playingStates.value.forEach((state, i) => {
      if (i !== index) {
        playingStates.value[i] = false
      }
    })
  }
}

// 点击卡片切换活跃状态
const setActiveCard = (index) => {
  activeIndex.value = index
}
</script>

<style scoped lang="scss">
.video-slider-container {
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
  padding: 90px 0 0 0;
  background: #f5f5f5;
  min-height: 700px;
}

.slider-title {
  margin-bottom: 55px;
  padding-left: 37px;
  
  h2 {
    font-size: 50px;
    color: #1D1D1F;
    font-weight: 600;
    margin: 0;
    font-family: "PingFang SC", sans-serif;
  }
}

.video-slider {
  position: relative;
  overflow: hidden;
  padding-left: 37px;
}

.video-cards-container {
  display: flex;
  gap: 35px;
  transition: transform 0.5s ease-in-out;
}

.video-card {
  flex-shrink: 0;
  border-radius: 12px;
  background: #e0e0e0;
  position: relative;
  cursor: pointer;
  transition: all 0.3s ease;
  
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  }
  
  &.active {
    box-shadow: 0 15px 40px rgba(0, 0, 0, 0.15);
  }
}

.video-card-small {
  width: 900px;
  height: 506px;
}

.video-card-large {
  width: 1030px;
  height: 579px;
}

.video-area {
  width: 100%;
  height: 100%;
  position: relative;
  border-radius: 12px;
  overflow: hidden;
}

.video-placeholder {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 24px;
  font-weight: 500;
  font-family: "PingFang SC", sans-serif;
}

.play-button {
  position: absolute;
  top: 40px;
  right: 30px;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.7);
  border: none;
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  z-index: 10;
  
  &:hover {
    background: rgba(0, 0, 0, 0.9);
    transform: scale(1.1);
  }
  
  .el-icon {
    font-size: 24px;
  }
}

.pagination-indicator {
  margin-top: 40px;
  display: flex;
  justify-content: center;
  padding-left: 37px;
}

.indicator-dots {
  display: flex;
  gap: 22px;
  align-items: center;
  
  .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #ccc;
    transition: all 0.3s ease;
    
    &.active {
      background: #007acc;
      transform: scale(1.2);
    }
    
    &:first-child,
    &:last-child {
      width: 12px;
      height: 12px;
    }
  }
}

// 响应式设计
@media (max-width: 1200px) {
  .video-slider-container {
    padding: 60px 20px 0 20px;
  }
  
  .slider-title {
    padding-left: 0;
    margin-bottom: 40px;
    
    h2 {
      font-size: 36px;
    }
  }
  
  .video-slider {
    padding-left: 0;
  }
  
  .video-card-small {
    width: 300px;
    height: 200px;
  }
  
  .video-card-large {
    width: 350px;
    height: 250px;
  }
  
  .video-cards-container {
    gap: 20px;
  }
  
  .play-button {
    width: 40px;
    height: 40px;
    top: 20px;
    right: 20px;
    
    .el-icon {
      font-size: 16px;
    }
  }
  
  .video-placeholder {
    font-size: 16px;
  }
}

@media (max-width: 768px) {
  .video-slider-container {
    padding: 40px 15px 0 15px;
  }
  
  .slider-title h2 {
    font-size: 28px;
  }
  
  .video-card-small {
    width: 250px;
    height: 150px;
  }
  
  .video-card-large {
    width: 280px;
    height: 180px;
  }
  
  .video-cards-container {
    gap: 15px;
  }
  
  .play-button {
    width: 35px;
    height: 35px;
    top: 15px;
    right: 15px;
    
    .el-icon {
      font-size: 14px;
    }
  }
  
  .video-placeholder {
    font-size: 14px;
  }
  
  .indicator-dots {
    gap: 15px;
    
    .dot {
      width: 6px;
      height: 6px;
      
      &:first-child,
      &:last-child {
        width: 8px;
        height: 8px;
      }
    }
  }
}
</style>
