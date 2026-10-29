<template>
  <div class="settings-container">
    <!-- 左侧菜单栏 -->
    <div class="sidebar">
      <div :class="['menu-item', { active: activeTab === 'profile' }]" @click="activeTab = 'profile'">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg>
        个人信息
      </div>
      <div :class="['menu-item', { active: activeTab === 'footprint' }]" @click="activeTab = 'footprint'">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M4 16v-2.38C4 11.5 2.97 10.5 3 8c.03-2.72 1.49-6 4.5-6C9.37 2 10 3.8 10 5.5c0 3.11-2 5.66-2 8.68V16a2 2 0 1 1-4 0Z"/>
          <path d="M20 20v-2.38c0-2.12 1.03-3.12 1-5.62-.03-2.72-1.49-6-4.5-6C14.63 6 14 7.8 14 9.5c0 3.11 2 5.66 2 8.68V20a2 2 0 1 0 4 0Z"/>
          <path d="M16 17h4"/>
          <path d="M4 13h4"/>
        </svg>
        个人足迹
      </div>
      <div :class="['menu-item', { active: activeTab === 'favorites' }]" @click="activeTab = 'favorites'">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path></svg>
        我的关注
      </div>
    </div>

    <!-- 右侧内容区 -->
    <div class="content">
      <!-- 1. 个人信息模块 -->
      <div v-if="activeTab === 'profile'" class="panel">
        <div class="panel-header">
          <h2>个人信息</h2>
          <button class="btn edit-btn" @click="openEditModal">修改个人信息</button>
        </div>
        
        <div class="profile-display">
        <!-- 头像 (外面套一个圆圈 div) -->
        <div class="avatar-circle">
            <img :src="userInfo.avatar" class="avatar" alt="头像" />
        </div>
          <!-- 昵称和座右铭 -->
          <div class="info-section">
            <h3 class="nickname">{{ userInfo.nickname }}</h3>
            <p class="motto">{{ userInfo.motto || '这个人很懒，还没有写座右铭哦~' }}</p >
          </div>
        </div>
        
        <!-- 个人相册 -->
        <h3 class="section-title">个人相册</h3>
        <div class="album">
          <div v-for="(img, index) in userInfo.album" :key="index" class="photo-wrap">
            <img :src="img" class="photo" alt="相册照片" />
          </div>
          <!-- 隐藏的相册文件输入框 -->
            <input type="file" id="album-upload" accept="image/*" class="hidden-file-input" @change="uploadPhoto" />
            <!-- 自定义的相册上传按钮 -->
            <label for="album-upload" class="btn upload-btn">+ 上传照片</label>
        </div>
      </div>

      <!-- 2. 个人足迹模块 -->
      <div v-if="activeTab === 'footprint'" class="panel">
        <h2>个人足迹</h2>
        <p class="desc">点击地图上的图钉，查看打卡相册：</p >
        
        <!-- 模拟中国地图区域 -->
        <div class="map-container">
          <div class="map-bg-text">中国地图区域 (待接入地图组件)</div>
          
          <div class="pin" style="top: 30%; left: 70%;" @click="showFootprintPhotos('北京')">
            <svg viewBox="0 0 24 24" fill="#e74c3c" width="30" height="30"><path d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zm0 9.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z"/></svg>
            <span class="pin-label">北京</span>
          </div>

          <div class="pin" style="top: 55%; left: 40%;" @click="showFootprintPhotos('四川')">
            <svg viewBox="0 0 24 24" fill="#e74c3c" width="30" height="30"><path d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zm0 9.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z"/></svg>
            <span class="pin-label">四川</span>
          </div>
        </div>

        <div v-if="currentFootprintPhotos.length > 0" class="album" style="margin-top: 20px;">
          <h3>{{ currentLocation }} - 打卡照片：</h3>
          <div class="photo-wrap" v-for="(img, index) in currentFootprintPhotos" :key="index">
            <img :src="img" class="photo" />
          </div>
        </div>
      </div>

      <!-- 3. 我的关注模块 -->
      <div v-if="activeTab === 'favorites'" class="panel">
        <h2>我的关注</h2>
        <p class="desc">您收藏的想去的地方：</p >
        <div class="favorite-grid">
          <div v-for="(item, index) in favorites" :key="index" class="fav-card">
            ❤️ {{ item }}
          </div>
        </div>
      </div>
    </div>

    <!-- 修改个人信息弹窗 -->
    <div v-if="showEditModal" class="modal-overlay" @click.self="showEditModal = false">
      <div class="modal-content">
        <h3>修改个人信息</h3>
        <div class="form-group avatar-upload-group">
        <label>修改头像：</label>
        <div class="upload-area">
            <!-- 预览选中的头像 -->
            <img v-if="editForm.avatar" :src="editForm.avatar" class="avatar-preview" />
            <!-- 隐藏的原生文件上传框 -->
            <input type="file" id="avatar-upload" accept="image/*" @change="handleAvatarUpload" class="hidden-file-input" />
            <!-- 自定义的漂亮按钮 -->
            <label for="avatar-upload" class="btn upload-btn-small">上传本地图片</label>
        </div>
        </div>
        <div class="form-group">
          <label>昵称：</label>
          <input v-model="editForm.nickname" class="input-box" />
        </div>
        <div class="form-group">
          <label>座右铭：</label>
          <input v-model="editForm.motto" class="input-box" placeholder="写一句旅行宣言" />
        </div>
        <div class="modal-actions">
          <button class="btn cancel-btn" @click="showEditModal = false">取消</button>
          <button class="btn save-btn" @click="saveUserInfo">保存修改</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 这里是解决报错的关键：定义 activeTab 和 showEditModal
const activeTab = ref('profile')
const showEditModal = ref(false)

// 用户信息
const userInfo = ref({
  avatar: 'https://images.unsplash.com/photo-1535713875002-d1d0cf377fde?w=150&q=80',
  nickname: '旅游达人小王',
  motto: '去拥抱陌生，去期待惊喜。',
  album: [
    'https://images.unsplash.com/photo-1584646098378-0874589d76b1?w=200&q=80',
    'https://images.unsplash.com/photo-1544785346-973bd66a54e1?w=200&q=80'
  ]
})

const editForm = ref({})

const openEditModal = () => {
  editForm.value = { ...userInfo.value }
  showEditModal.value = true
}
// 处理本地图片上传并转为 Base64
const handleAvatarUpload = (event) => {
  const file = event.target.files[0];
  if (!file) return;

  // 限制图片大小不能超过 2MB (防止 localStorage 内存爆掉)
  if (file.size > 2 * 1024 * 1024) {
    alert("图片太大了，请选择小于 2MB 的图片！");
    return;
  }

  const reader = new FileReader();
  reader.onload = (e) => {
    // e.target.result 就是转换后的 Base64 图片数据
    editForm.value.avatar = e.target.result;
  };
  reader.readAsDataURL(file);
};
const saveUserInfo = () => {
  userInfo.value = { ...editForm.value }
  localStorage.setItem('userInfo', JSON.stringify(userInfo.value))
  showEditModal.value = false
}

const uploadPhoto = (event) => {
  const file = event.target.files[0];
  if (!file) return;

  // 限制不能超过 2MB (防止 localStorage 爆掉)
  if (file.size > 2 * 1024 * 1024) {
    alert("照片太大了，请选择小于 2MB 的照片！");
    event.target.value = ''; 
    return;
  }

  const reader = new FileReader();
  reader.onload = (e) => {
    // 把读取到的 Base64 图片数据推入相册数组
    userInfo.value.album.push(e.target.result);
    
    // 存入 localStorage 以便刷新后还在
    try {
      localStorage.setItem('userInfo', JSON.stringify(userInfo.value));
    } catch (error) {
      alert("您的浏览器存储空间已满，无法保存更多照片啦！");
      userInfo.value.album.pop(); 
    }
    
    // 清空 input 的值，允许下次上传同一张图
    event.target.value = ''; 
  };
  reader.readAsDataURL(file);
}
// 足迹相关
const currentLocation = ref('')
const currentFootprintPhotos = ref([])
const footprintData = {
  '北京': ['https://images.unsplash.com/photo-1508804185872-d7badad00f7d?w=300&q=80'],
  '四川': ['https://images.unsplash.com/photo-1564349683136-772ceb6d3800?w=300&q=80']
}
const favorites = ref(['西藏拉萨', '新疆伊犁', '云南大理'])

const showFootprintPhotos = (place) => {
  currentLocation.value = place
  currentFootprintPhotos.value = footprintData[place] || []
}
</script>

<style scoped>
/* 整体布局 */
.settings-container {
  display: flex;
  width: 90%;
  max-width: 1000px;
  margin: 40px auto;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.08);
  min-height: 600px;
  overflow: hidden;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* 左侧菜单 */
.sidebar {
  width: 240px;
  background: #f8fafc;
  border-right: 1px solid #e2e8f0;
  padding: 20px 0;
}

.menu-item {
  padding: 16px 24px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 16px;
  color: #475569;
  transition: all 0.3s ease;
}

.menu-item:hover {
  background: #f1f5f9;
}

.menu-item.active {
  background: #e0f2fe;
  color: #0284c7;
  font-weight: bold;
  border-right: 4px solid #0284c7;
}

.icon {
  width: 20px;
  height: 20px;
}

/* 右侧内容 */
.content {
  flex: 1;
  padding: 40px;
  background: #ffffff;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid #f1f5f9;
  padding-bottom: 12px;
  margin-bottom: 24px;
}

.panel-header h2 {
  margin: 0;
  color: #1e293b;
}

.desc {
  color: #64748b;
  margin-bottom: 20px;
}

/* 个人信息区 */
.profile-display {
  display: flex;
  align-items: center;
  gap: 30px;
  margin-bottom: 40px;
  padding: 20px;
  background: #f8fafc;
  border-radius: 12px;
}

/* 外层圆圈，固定大小和圆形 */
.avatar-circle {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  overflow: hidden; /* 保证图片超出部分被裁掉 */
  border: 4px solid #fff;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  flex-shrink: 0;
}

/* 内部图片，填满圆圈 */
.avatar {
  width: 100%;
  height: 100%;
  object-fit: cover; /* 保证图片不变形，填满整个圆圈 */
}

.info-section {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.nickname {
  margin: 0;
  font-size: 28px;
  color: #1e293b;
}

.motto {
  margin: 0;
  font-size: 15px;
  color: #64748b;
  font-style: italic;
}

.section-title {
  color: #1e293b;
  margin-bottom: 16px;
  font-size: 18px;
}

/* 按钮与相册 */
.btn {
  padding: 8px 16px;
  background: #0284c7;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: 0.2s;
}
.btn:hover { background: #0369a1; }

.edit-btn {
  background: transparent; /* 背景变透明 */
  color: #94a3b8;          /* 字体保持蓝色 */
  border: none;            /* 去掉边框 */
  padding: 0;              /* 去掉内边距 */
  font-size: 16px;
}
.edit-btn:hover { 
  text-decoration: underline; /* 鼠标移上去时加个下划线，提示可以点击 */
  color:#64748b;
  background: transparent;
}

.album {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  margin-top: 16px;
}

.photo-wrap {
  width: 120px;
  height: 120px;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.photo {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.upload-btn {
  width: 120px;
  height: 120px;
  background: #f8fafc;
  color: #64748b;
  border: 2px dashed #cbd5e1;
}
.upload-btn:hover {
  background: #f1f5f9;
  color: #0284c7;
  border-color: #0284c7;
}

/* 模拟地图区域 (图钉) */
.map-container {
  position: relative;
  width: 100%;
  height: 350px;
  background: #f0f9ff url('https://images.unsplash.com/photo-1524661135-423995f22d0b?w=800&q=80') center/cover;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: inset 0 0 20px rgba(0,0,0,0.1);
  border: 1px solid #bae6fd;
}

.map-bg-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: rgba(255,255,255,0.7);
  font-size: 24px;
  font-weight: bold;
  text-shadow: 0 2px 4px rgba(0,0,0,0.3);
  pointer-events: none;
}

.pin {
  position: absolute;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  animation: bounce 2s infinite ease-in-out;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}

.pin svg {
  filter: drop-shadow(0 4px 4px rgba(0,0,0,0.3));
}

.pin-label {
  background: rgba(0,0,0,0.7);
  color: white;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 12px;
  margin-top: 4px;
  white-space: nowrap;
}

/* 关注网格 */
.favorite-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 16px;
}

.fav-card {
  padding: 20px;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  text-align: center;
  color: #334155;
  font-weight: 500;
  transition: 0.2s;
}

.fav-card:hover {
  background: #fff1f2;
  border-color: #fecdd3;
  color: #e11d48;
  transform: translateY(-2px);
}

/* 弹窗样式 */
.modal-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.modal-content {
  background: #fff;
  padding: 30px;
  border-radius: 12px;
  width: 400px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.modal-content h3 {
  margin-top: 0;
  margin-bottom: 20px;
  color: #1e293b;
  text-align: center;
}

.modal-content .form-group {
  margin-bottom: 15px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.modal-content label {
  font-size: 14px;
  color: #475569;
}

.input-box {
  padding: 10px 14px;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  outline: none;
  transition: 0.2s;
  width: 100%;
  box-sizing: border-box;
}

.input-box:focus {
  border-color: #0284c7;
  box-shadow: 0 0 0 3px rgba(2,132,199,0.1);
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
}

.cancel-btn {
  background: #f1f5f9;
  color: #475569;
}
.cancel-btn:hover { background: #e2e8f0; }

.save-btn {
  background: #0284c7;
  color: white;
}
/* 隐藏原生丑陋的文件上传框 */
.hidden-file-input {
  display: none;
}

.avatar-upload-group {
  align-items: flex-start;
}

.upload-area {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-top: 8px;
}

/* 弹窗里的小头像预览 */
.avatar-preview {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid #e2e8f0;
}

/* 上传按钮样式 */
.upload-btn-small {
  padding: 8px 16px;
  font-size: 14px;
  background: #f8fafc;
  color: #0284c7;
  border: 1px dashed #bae6fd;
  border-radius: 6px;
  cursor: pointer;
  transition: 0.2s;
}

.upload-btn-small:hover {
  background: #e0f2fe;
  border-color: #0284c7;
}
</style>