<script setup>
import { ref, onMounted, computed } from 'vue'
import ProjectCard from '../components/ProjectCard.vue'
import VideoPlayer from '../components/VideoPlayer.vue'
import { getProjects } from '../data/projects'
import { ElButton, ElInput, ElIcon } from 'element-plus'
import { Search } from '@element-plus/icons-vue'
const projects = ref([])
const moreProjects = ref([])

// 页眉相关数据
const navOpen = ref(false)
const isDark = ref(true)
const currentLang = ref('zh')
const searchQuery = ref('')

const logoImage = computed(() => {
  return new URL('../assets/images/lab_logo_icon.png', import.meta.url).href
})


const navPages = [
  { path: '/', title: '首页', tooltip: '主页' },
  { path: '/research', title: '研究', tooltip: '研究项目与成果' },
  { path: '/team', title: '团队', tooltip: '课题组成员' },
  { path: '/hiring', title: '招聘', tooltip: '招聘信息' },
  { path: '/projects', title: '项目', tooltip: '查看项目' },
]

// 引用图片列表 - 使用动态导入
const citationImages = ref([
  new URL('../assets/images/citations/2410_06765.jpg', import.meta.url).href,
  new URL('../assets/images/citations/2410_06554.png', import.meta.url).href,
  new URL('../assets/images/citations/2410_04691.jpg', import.meta.url).href,
  new URL('../assets/images/citations/2407_17011.png', import.meta.url).href,
  new URL('../assets/images/citations/2406_18134.jpg', import.meta.url).href,
  new URL('../assets/images/citations/2404_14122.png', import.meta.url).href,
  new URL('../assets/images/citations/2309_16289.jpg', import.meta.url).href
])

// B 容器走马灯视频资源（来自 @/assets/movies）
const carouselVideos = ref([
  new URL('../assets/movies/test1.mp4', import.meta.url).href,
  new URL('../assets/movies/test2.mp4', import.meta.url).href,
  new URL('../assets/movies/test3.mp4', import.meta.url).href,
])

// 图片模态框
const imageModalVisible = ref(false)
const currentImage = ref('')

// 视频播放控制
/**
 * 说明：本组件统一使用 @/assets/movies 下的视频资源
 * - 物理路径：src/assets/movies
 * - Vite 中 @ 指向 src，因此等价于 @/assets/movies
 * - 这里通过 new URL 的方式构造资源路径，确保构建产物也能正确引用
 */
const videoSrc = new URL('../assets/movies/test1.mp4', import.meta.url).href
const aContainerVideoSrc = new URL('../assets/movies/projects/AI伙伴视频高清.mp4', import.meta.url).href

// 打开图片模态框
const openImageModal = (imageSrc) => {
  currentImage.value = imageSrc
  imageModalVisible.value = true
}


// 语言切换
const switchLanguage = (lang) => {
  currentLang.value = lang
  // 这里可以添加国际化逻辑
  console.log('切换到语言:', lang)
}

// 搜索处理
const handleSearch = () => {
  if (searchQuery.value.trim()) {
    console.log('搜索:', searchQuery.value)
    // 这里可以添加搜索逻辑
  }
}

onMounted(async () => {
  const allProjects = await getProjects()
  projects.value = allProjects.filter(p => p.group !== 'more')
  moreProjects.value = allProjects.filter(p => p.group === 'more')
})
</script>
<template>
  <div class="projects-page">
    <!-- 自定义页眉 -->
    <header class="custom-header">
      <div class="header-content">
        <!-- 主要内容区域 -->
        <div class="main-content">
          <router-link to="/" class="logo-link">
            <span class="logo">
              <img :src="logoImage" alt="logo">
            </span>
          </router-link>
          <nav class="main-nav">
            <router-link to="/" class="nav-item">首页</router-link>
            <router-link to="/research" class="nav-item">论文发表</router-link>
            <router-link to="/team" class="nav-item">研究团队</router-link>
            <router-link to="/projects" class="nav-item">项目</router-link>
            <router-link to="/hiring" class="nav-item">加入我们</router-link>
          </nav>
        </div>
        
        <!-- 功能区域 -->
        <div class="functional-content">
          <!-- 中英文切换 -->
          <div class="language-switch">
            <button 
              class="lang-btn"
              :class="{ active: currentLang === 'zh' }"
              @click="switchLanguage('zh')"
            >
              中
            </button>
            <span class="lang-separator">|</span>
            <button 
              class="lang-btn"
              :class="{ active: currentLang === 'en' }"
              @click="switchLanguage('en')"
            >
              EN
            </button>
          </div>
          
          <!-- 搜索框 -->
          <div class="search-box">
            <el-input
              v-model="searchQuery"
              placeholder="搜索..."
              class="search-input"
              @keyup.enter="handleSearch"
            >
              <template #suffix>
                <el-icon class="search-icon" @click="handleSearch">
                  <Search />
                </el-icon>
              </template>
            </el-input>
          </div>
          
          <!-- 移动端菜单按钮 -->
          <input 
            class="nav-toggle" 
            type="checkbox" 
            aria-label="show/hide nav"
            v-model="navOpen"
          >
        </div>
      </div>
    </header>

    <!-- 主容器 -->
    <main class="main-container">
      <div class="content-wrapper">
        <!-- A容器 - 项目列表 -->
        <section class="a-container">
          <div class="container-header">
            <VideoPlayer 
              :video-src="aContainerVideoSrc"
              video-class="header-background-image"
              :autoplay="true"
              :loop="true"
            />
          </div>
        </section>
        <!-- B容器 - 引用图片展示区 -->
        <section class="b-container">
          <div class="container-header">
            <h2>标题 -- 短视频滑动区</h2>
          </div>
            <div class="container-content">
              <el-carousel 
                :interval="4000" 
                type="card" 
                height="480px"
                indicator-position="outside"
                arrow="hover"
              >
                <el-carousel-item
                  v-for="vid in carouselVideos"
                  :key="vid"
                >
                  <div class="carousel-video-item">
                    <div class="video-wrapper">
                      <video 
                        :src="vid"
                        controls
                        playsinline
                      ></video>
                    </div>
                  </div>
                </el-carousel-item>
              </el-carousel>
            </div>
        </section>



        <!-- C容器 - 额外内容区 -->
        <section class="c-container">
          <!-- <div class="container-header">
            <h2>标题 -- 主链路完整系统功能介绍</h2>
          </div> -->
          <div class="container-content">
            <div class="content-layout-up">
              <div class="container-content-description">
                <h2>标题 -- 主链路完整系统功能介绍</h2>
                <p>
                  这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍。这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍。
                </p>
              </div>
              <div class="container-content-video">
                <VideoPlayer 
                  :video-src="videoSrc"
                />
              </div>
            </div>
            <div class="content-layout-down">
              <div class="container-content-video">
                <VideoPlayer 
                  :video-src="videoSrc"
                />
              </div>
              <div class="container-content-description">
                <h2>标题 -- 主链路完整系统功能介绍</h2>
                <p>
                  这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍。这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍，这里是系统功能介绍。
                </p>
              </div>
            </div>
          </div>
        </section>
      </div>
    </main>
    
    <!-- 图片模态框 -->
    <el-dialog
      v-model="imageModalVisible"
      title="引用图片详情"
      width="80%"
      center
    >
      <div class="modal-image-container">
        <img 
          :src="currentImage" 
          class="modal-image" 
          alt="引用图片详情"
        />
      </div>
    </el-dialog>
    
    <!-- 自定义页脚 -->
    <footer class="projects-footer">
      <!-- SVG Logo -->
      <div class="footer-logo">
        <img src="../assets/images/base/footlogo.svg" alt="EIT Logo" />
      </div>
      
      <div class="footer-content">
        <div class="footer-section">
          <div class="footer-section-title">
            <h3>联系我们</h3>
          </div>
          <div class="footer-section-content">
            <p>邮箱：xyshen@eitech.edu.cn</p>
            <p>|</p>
            <p>地址：浙江省宁波市镇海区庄市街道同心路568号</p>
            <p>|</p>
            <p>
              <img src="@/assets/images/base/github-mark-white.svg" alt="GitHub" style="width: 18px; height: 18px; vertical-align: middle; margin-right: 6px;" />
              Built with Lab Website Template
            </p>
          </div>
        </div>
      </div>

    </footer>
  </div>
</template>



<style scoped lang="scss">
.projects-page {
  min-height: 100vh;
  background: #ffffff;
  display: flex;
  flex-direction: column;
}

// 自定义页眉样式
.custom-header {
  position: relative;
  background: #7a7a7a;
  color: #ffffff;
  z-index: 1;
  padding: 0 0;
  height: 120px;
  box-shadow: var(--shadow);
  position: sticky;
  top: 0;
  z-index: 10;
  // opacity: 0.6;
  font-family: 'PingFang SC';
  font-size: 24px;
  font-weight: 600;
  font-style: Semibold;
  line-height: 30px;
  letter-spacing: 0%;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 0 20px;
}

// 主要内容区域
.main-content {
  display: flex;
  align-items: center;
  gap: 120px;
}

.logo-link {
  display: flex;
  align-items: center;
  text-decoration: none;
}

.logo {
  height: 60px;
}

.logo > * {
  width: unset;
  height: 100%;
}

.main-nav {
  display: flex;
  align-items: center;
  gap: 40px;
  font-family: var(--heading);
}

.nav-item {
  color: #ffffff;
  text-decoration: none;
  font-size: 16px;
  font-weight: 500;
  padding: 8px 0;
  transition: all 0.3s ease;
  position: relative;
}

.nav-item:hover {
  color: #ffffff;
}

.nav-item.router-link-active {
  color: #ffffff;
}

.nav-item.router-link-active::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: #ffffff;
  border-radius: 1px;
}

// 功能区域
.functional-content {
  display: flex;
  align-items: center;
  gap: 60px;
}

// 语言切换
.language-switch {
  display: flex;
  align-items: center;
  gap: 8px;
}

.lang-btn {
  background: transparent;
  border: none;
  color: #ffffff;
  font-size: 14px;
  font-weight: 500;
  padding: 4px 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  opacity: 0.7;
  position: relative;
}

.lang-btn:hover {
  opacity: 1;
}

.lang-btn.active {
  opacity: 1;
  color: #ffffff;
}

.lang-btn.active::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: #ffffff;
  border-radius: 1px;
}

.lang-separator {
  color: #ffffff;
  opacity: 0.5;
  font-size: 14px;
}

// 搜索框
.search-box {
  display: flex;
  align-items: center;
}

.search-input {
  width: 200px;
}

.search-input :deep(.el-input__wrapper) {
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 20px;
  box-shadow: none;
}

.search-input :deep(.el-input__inner) {
  color: #ffffff;
  background: transparent;
}

.search-input :deep(.el-input__inner::placeholder) {
  color: rgba(255, 255, 255, 0.6);
}

.search-input :deep(.el-input__suffix) {
  color: #ffffff;
}

.search-icon {
  cursor: pointer;
  transition: color 0.3s ease;
}

.search-icon:hover {
  color: var(--primary);
}

.nav-toggle {
  display: none;
  position: relative;
  width: 30px;
  height: 30px;
  margin: 0;
  color: #ffffff;
  appearance: none;
  transition: background var(--transition);
  background: transparent;
  border: none;
  cursor: pointer;
}

.nav-toggle:after {
  content: "\f0c9";
  position: absolute;
  left: 50%;
  top: 50%;
  color: #ffffff;
  font-size: 18px;
  font-family: "Font Awesome 6 Free";
  font-weight: 900;
  transform: translate(-50%, -50%);
}

.nav-toggle:checked:after {
  content: "\f00d";
}

// 响应式设计
@media (max-width: 768px) {
  .header-content {
    padding: 0 15px;
  }
  
  .main-content {
    gap: 20px;
  }
  
  .main-nav {
    gap: 20px;
  }
  
  .nav-item {
    font-size: 14px;
  }
}

@media (max-width: 600px) {
  .header-content {
    padding: 0 10px;
  }
  
  .main-content {
    flex-direction: column;
    gap: 15px;
    align-items: flex-start;
  }
  
  .main-nav {
    display: none;
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: #222222;
    flex-direction: column;
    padding: 20px;
    gap: 15px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.3);
    z-index: 100;
  }
  
  .functional-content {
    flex-direction: column;
    gap: 10px;
    align-items: flex-end;
  }
  
  .search-input {
    width: 150px;
  }
  
  .nav-toggle {
    display: block;
  }
  
  .nav-toggle:checked ~ .main-content .main-nav {
    display: flex;
  }
}


// 主容器样式
.main-container {
  flex: 1;
  max-width: 100%;
  // margin: 0 146px;
  // padding: 40px 20px;
  display: flex;
  flex-direction: column;
}

// 内容包装器
.content-wrapper {
  display: flex;
  flex-direction: column;
  gap: 90px;
  width: auto;
  height: 100%;
  margin: 0 146px;
}

.a-container{
  width:100%;
  height: auto;
  padding: 0 0 0 0;
  background: #fff;
  .container-header{
    position: relative;
    width: 100%;
    height: 915px;
    padding: 0 0 0 0;
    background: none;
    overflow: hidden;
    
    .video-wrapper {
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
    }
    
    
    .header-background-image {
      position: absolute;
      top: 0;
      left: 50%;
      transform: translateX(-50%);
      width: auto;
      height: 100%;
      object-fit: cover;
      z-index: 1;
    }
    
    /* 组合标题样式 */
    .title-composed {
      position: relative;
      z-index: 2;
      font-family: 'Inter', sans-serif;
      font-weight: 700;
      font-style: bold;
      font-size: 60px;
      line-height: 100%;
      letter-spacing: 0;
      margin: 0;
      text-align: left;
      display: flex;
      align-items: flex-end;
      gap: 8px;
      padding-left: 88px;
      height: 520px;
  
    }
    .title-left,
    .title-right {
      background: linear-gradient(90deg, #000000 0%, #9747FF 100%);
      -webkit-background-clip: text;
      background-clip: text;
      color: transparent;
      -webkit-text-fill-color: transparent;
      line-height: 1.2;
      margin-bottom: 24px;
    }
    .title-sep {
      color: #7a7a7a;
      font-weight: 600;
      line-height: 1.2;
    }
    p {
      position: relative;
      z-index: 2;
      font-family: 'PingFang SC', 'Microsoft YaHei', 'Helvetica Neue', Arial, sans-serif;
      font-weight: 400;
      font-style: normal;
      font-size: 20px;
      line-height: 100%;
      letter-spacing: 0;
      color: #656565;
      margin: 0;
      text-align: justify;
      padding-left: 88px;
      padding-bottom: 45px;
      width: 40%;
      line-height: 150%;
    }
  }
}
.b-container{
  width:100%;
  height: auto;
  padding: 0 0 0 0;
  background: #fff;
  .container-header{
    padding: 0 0 0 0;
    background: none;
    margin-left: 88px;
    h2 {    
    font-family: 'PingFang SC', 'Microsoft YaHei', 'Helvetica Neue', Arial, sans-serif;
    font-weight: 500;
    font-style: normal;
    font-size: 50px !important;
    line-height: 100%;
    letter-spacing: 0;
    margin-top: 0px;
    margin-bottom: 55px;
    /* leading-trim: NONE;  CSS暂不支持leading-trim，忽略 */
    }
  }


/* 在舞台上但没有被激活的视频添加蒙版 */
.el-carousel__item.is-in-stage:not(.is-active) {
  .video-wrapper {
    position: relative;
    
    &::after {
      content: "";
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: rgba(128, 128, 128, 0.4); /* 灰色蒙版 */
      z-index: 2;
      pointer-events: none;
      border-radius: 8px;
      transition: all 0.3s ease;
    }
  }
}

/* 当前激活的视频 */
.el-carousel__item.is-active {
  .video-wrapper {
    box-shadow: 0 8px 32px 0 rgba(80, 40, 180, 0.18), 0 1.5px 8px 0 rgba(80, 40, 180, 0.10);
    border-radius: 12px;
    position: relative;
    z-index: 3;
    transition: box-shadow 0.3s cubic-bezier(.4,0,.2,1), transform 0.3s cubic-bezier(.4,0,.2,1);
    transform: scale(1.24);
    &::after {
      display: none; /* 激活状态不显示蒙版 */
    }
  }
}
  .carousel-video-item { height: 100%; }
  .video-wrapper { height: 579px; }      /* 改成你需要的高度 */
  .video-wrapper video {
    width: 100%;
    height: 100%;
    max-width: 100%;
    object-fit: contain;                  /* 保持比例，完整显示 */
  }
}

.c-container{
  width:100%;
  height: auto;
  padding: 0 0 0 0;
  background: #fff;
  
  // .container-header{
  //   padding: 0 0 0 0;
  //   background: none;
  //   margin-left: 88px;
  //   h2 {    
  //     font-family: 'PingFang SC', 'Microsoft YaHei', 'Helvetica Neue', Arial, sans-serif;
  //     font-weight: 500;
  //     font-style: normal;
  //     font-size: 50px !important;
  //     line-height: 100%;
  //     letter-spacing: 0;
  //     margin-top: 90px;
  //     margin-bottom: 55px;
  //     /* leading-trim: NONE;  CSS暂不支持leading-trim，忽略 */
  //   }
  // }
  
  .container-content {
    height: 100%;
    // padding-left:88px;
    .content-layout-up,
    .content-layout-down{
      display: flex;
      height: 100%;
      align-items: center;
      .container-content-description{
        // margin-left: 88px;
        flex: 4;
        padding: 0px 100px 0px 88px;
        h2 {    
          font-family: 'PingFang SC', 'Microsoft YaHei', 'Helvetica Neue', Arial, sans-serif;
          font-weight: 500;
          font-style: normal;
          font-size: 50px !important;
          line-height: 100%;
          letter-spacing: 0;
          margin-top: 0px;
          margin-bottom: 45px;
          height:100%;
          flex: 4;
          padding: 30px 0px 0px 0px;
                  
          /* leading-trim: NONE;  CSS暂不支持leading-trim，忽略 */
        }
        p{
          font-family: 'PingFang SC', sans-serif;
          font-weight: 400;
          font-style: normal;
          font-size: 28px;
          /* leading-trim: NONE;  CSS暂不支持leading-trim，忽略 */
          line-height: 50px;
          letter-spacing: 0;
          color: #424242;
          margin-top: 0px;
          text-align: justify;
          
        }
      }
      .container-content-video {
        flex: 6;
        display: flex;
        // align-items: center;
        // justify-content: center;
        // padding: 20px;
        margin-top: 100px;
      }
    }
  }
  .content-layout-up{
    align-items: flex-start !important;
    // padding-right: 88px !important;
    .container-content-description{
      margin-top: 0px !important;
      padding-right: 170px !important;
    }
    .container-content-video{
      margin-right: 88px;
      margin-top: 0px !important;
    }
  }
  .content-layout-down{
    align-items: flex-start !important;
    margin-top: 100px !important;
    .container-content-description{
      padding-left: 170px !important;
      margin-bottom: 100px !important;
      padding-right: 88px !important;
    }
    .container-content-video{
      margin-top: 0px !important;

      margin-bottom: 100px !important;
      margin-left: 88px;
    }
  }
  
  
}





.citation-image-container {
  position: relative;
  width: 100%;
  height: 100%;
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease;
  
  &:hover {
    transform: scale(1.02);
    
    .image-overlay {
      opacity: 1;
    }
  }
}

.carousel-video-item {
  width: 100%;
  height: 100%;
}

.citation-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 12px;
  transition: all 0.3s ease;
}

.image-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.7));
  padding: 20px;
  opacity: 0;
  transition: all 0.3s ease;
}

.image-title {
  color: white;
  font-size: 1.2rem;
  font-weight: 500;
  font-family: "PingFang SC", sans-serif;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
}

.modal-image-container {
  text-align: center;
  
  .modal-image {
    max-width: 100%;
    max-height: 70vh;
    border-radius: 8px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  }
}

.projects-section {
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 20px;
  background: white;
  border-radius: 12px;
  margin-top: 40px;
  
  h2 {
    font-size: 1.8rem;
    color: #1D1D1F;
    margin-bottom: 2rem;
    text-align: center;
    font-family: "PingFang SC", sans-serif;
  }
}

.projects-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  margin-top: 2rem;
}

.project-card {
  background: #f8f9fa;
  padding: 2rem;
  border-radius: 12px;
  border: 1px solid #e9ecef;
  transition: all 0.3s ease;
  
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
    border-color: var(--primary);
  }
  
  h3 {
    font-size: 1.3rem;
    color: var(--primary);
    margin-bottom: 1rem;
    font-family: "PingFang SC", sans-serif;
  }
  
  p {
    color: #666;
    line-height: 1.6;
    margin-bottom: 1.5rem;
  }
}

.project-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  
  .tag {
    background: var(--primary);
    color: white;
    padding: 0.3rem 0.8rem;
    border-radius: 20px;
    font-size: 0.8rem;
    font-weight: 500;
  }
}

// 响应式设计
@media (max-width: 768px) {
  .main-container {
    padding: 20px 15px;
    gap: 30px;
  }
  
  .a-container,
  .b-container,
  .c-container {
    height: auto;
    min-height: 400px;
  }
  
  .c-container {
    .content-layout {
      flex-direction: column;
      height: auto;
    }
    
    // .container-content-description {
    //   flex: none;
    //   padding: 30px 20px;
    //   width: 100%;
    // }
    
    .container-content-video {
      flex: none;
      padding: 20px;
      width: 100%;
      
      video {
        height: 250px;
      }
    }
  }
}

@media (max-width: 480px) {
  .page-header {
    padding: 40px 10px 30px;
    
    .header-content {
      h1 {
        font-size: 1.8rem;
      }
      
      p {
        font-size: 1rem;
      }
    }
  }
  
  .main-container {
    padding: 15px 10px;
    gap: 20px;
  }
  
  .container-header {
    padding: 15px;
    
    h2 {
      font-size: 1.3rem;
    }
  }
  
  .container-content {
    padding: 15px;
  }
}

// 自定义页脚样式
.projects-footer {
  position: relative;
  width: 100%;
  height: 570px;
  color: white;
  margin-top: 80px;
  overflow: hidden;
  font-family: "PingFang SC", sans-serif;
  
  // 背景层
  &::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(0deg, rgba(3, 13, 38, 1) 0%, rgba(36, 6, 76, 1) 100%);
    z-index: 1;
  }
  
  &::after {
    content: "";
    position: absolute;
    top: 1px;
    left: -10px;
    right: 0;
    bottom: 0;
    width: 103%;
    height: 100%;
    background-image: url('@/assets/images/base/footer-image-1.png');
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    opacity: 0.5;
    z-index: 2;
    pointer-events: none;
  }
  
  // Logo
  .footer-logo {
    position: absolute;
    left: 50%;
    bottom: 10px;
    transform: translateX(-50%);
    z-index: 3;
    width: 1500px;
    height: auto;

    img {
      width: 100%;
      height: auto;
      filter: brightness(0) invert(1);
    }
  }
  
  // 内容区域
  .footer-content {
    position: absolute;
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
    width: 100%;
    padding: 0 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    text-align: center;
    z-index: 3;
    
    .footer-section {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      
      .footer-section-title h3 {
        font-weight: 600;
        font-size: 28px;
        line-height: 45px;
        letter-spacing: 0;
        text-align: center;
        margin-bottom: 50px;
        color: #fff;
      }
      
      .footer-section-content {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;
        gap: 60px;
        flex-wrap: wrap;
      }
      
      p {
        color: rgba(255, 255, 255, 0.8);
        font-weight: 400;
        font-size: 24px;
        line-height: 45px;
        letter-spacing: 0;
        margin: 0;
        white-space: nowrap;
      }
    }
  }
}

// 响应式设计
@media (max-width: 768px) {
  .projects-footer {
    height: 500px;
    margin-top: 60px;
    
    .footer-logo {
      width: 150px;
    }
    
    .footer-content {
      padding: 0 15px;
      
      .footer-section {
        .footer-section-title h3 {
          font-size: 1.1rem;
        }
        
        .footer-section-content {
          gap: 30px;
        }
      }
    }
  }
}

@media (max-width: 480px) {
  .projects-footer {
    height: 450px;
    
    .footer-logo {
      width: 120px;
    }
    
    .footer-content {
      .footer-section {
        .footer-section-title h3 {
          font-size: 1rem;
        }
        
        .footer-section-content {
          gap: 20px;
        }
        
        p {
          font-size: 13px;
        }
      }
    }
  }
}
</style>