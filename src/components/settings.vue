<template>
  <div class="settings-container">
    <!-- 左侧菜单栏 -->
    <div class="sidebar">
      <div :class="['menu-item', { active: activeTab === 'profile' }]" @click="activeTab = 'profile'">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg>
        个人信息
      </div>
      <div :class="['menu-item', { active: activeTab === 'favorites' }]" @click="activeTab = 'favorites'">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path></svg>
        我的关注
      </div>
      <div :class="['menu-item', { active: activeTab === 'settings' }]" @click="activeTab = 'settings'">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="3"></circle>
          <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path>
        </svg>
        设置
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
          <div class="avatar-circle">
            <img :src="userInfo.avatar" class="avatar" alt="头像" />
          </div>
          <div class="info-section">
            <h3 class="nickname">{{ userInfo.nickname }}</h3>
            <p class="motto">{{ userInfo.motto || '这个人很懒，还没有写座右铭哦~' }}</p >
          </div>
        </div>
        
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

      <!-- 2. 我的关注模块 -->
      <div v-if="activeTab === 'favorites'" class="panel" style="display: flex; flex-direction: column; align-items: flex-start; text-align: left; width: 100%; padding-top: 40px;">
        <div class="panel-header" style="width: 100%; text-align: left; margin-bottom: 20px;">
          <h2 style="margin: 0;">我的关注</h2>
        </div>
        <p class="desc" style="text-align: left; width: 100%; color: #64748b; margin-bottom: 20px;">您收藏的想去的地方：</p >
        <div style="display: flex; flex-wrap: wrap; gap: 20px; width: 100%; justify-content: flex-start;">
          <div v-for="(item, index) in favorites" :key="index" 
               style="padding: 15px 25px; background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 8px; display: flex; align-items: center; gap: 10px; color: #334155; font-weight: 500;">
            <span style="font-size: 18px;">❤️</span> 
            <span>{{ item }}</span>
          </div>
        </div>
      </div>

      <!-- 3. 设置模块 -->
      <div v-if="activeTab === 'settings'" class="panel" style="display: flex; flex-direction: column; width: 100%; padding: 60px 20px; box-sizing: border-box;">
        <div class="panel-header" style="width: 100%; text-align: left; margin-bottom: 40px;">
          <h2 style="margin: 0;">设置</h2>
        </div>
        <div style="display: flex; flex-direction: column; gap: 45px; width: 100%;">
          <!-- 节奏 -->
          <div style="display: flex; justify-content: space-between; align-items: center; width: 100%;">
            <div style="display: flex; flex-direction: column; align-items: flex-start; gap: 8px;">
              <div style="font-size: 16px; font-weight: bold; color: #1e293b;">节奏</div>
              <div style="font-size: 14px; color: #94a3b8;">轻松漫游 / 标准紧凑 / 特种兵暴走</div>
            </div>
            <label class="switch">
              <input type="checkbox" v-model="settingPace" />
              <span class="slider round"></span>
            </label>
          </div>
          <!-- 优先经济型方案 -->
          <div style="display: flex; justify-content: space-between; align-items: center; width: 100%;">
            <div style="display: flex; flex-direction: column; align-items: flex-start; gap: 8px;">
              <div style="font-size: 16px; font-weight: bold; color: #1e293b;">优先经济型方案</div>
              <div style="font-size: 14px; color: #94a3b8;">优先推荐性价比交通与住宿</div>
            </div>
            <label class="switch">
              <input type="checkbox" v-model="settingEconomy" />
              <span class="slider round"></span>
            </label>
          </div>
          <!-- 启用中国地图可视化 -->
          <div style="display: flex; justify-content: space-between; align-items: center; width: 100%;">
            <div style="display: flex; flex-direction: column; align-items: flex-start; gap: 8px;">
              <div style="font-size: 16px; font-weight: bold; color: #1e293b;">启用中国地图可视化</div>
              <div style="font-size: 14px; color: #94a3b8;">功能开发中，敬请期待</div>
            </div>
            <label class="switch">
              <input type="checkbox" disabled />
              <span class="slider round"></span>
            </label>
          </div>
          <!-- 行程变更提醒 -->
          <div style="display: flex; justify-content: space-between; align-items: center; width: 100%;">
            <div style="display: flex; flex-direction: column; align-items: flex-start; gap: 8px;">
              <div style="font-size: 16px; font-weight: bold; color: #1e293b;">行程变更提醒</div>
              <div style="font-size: 14px; color: #94a3b8;">功能开发中，敬请期待</div>
            </div>
            <label class="switch">
              <input type="checkbox" disabled />
              <span class="slider round"></span>
            </label>
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
            <img v-if="editForm.avatar" :src="editForm.avatar" class="avatar-preview" />
            <input type="file" id="avatar-upload" accept="image/*" @change="handleAvatarUpload" class="hidden-file-input" />
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

const activeTab = ref('profile')
const showEditModal = ref(false)

// 设置页面的开关状态
const settingPace = ref(true)
const settingEconomy = ref(false)

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

const handleAvatarUpload = (event) => {
  const file = event.target.files[0]
  if (!file) return
  if (file.size > 2 * 1024 * 1024) {
    alert("图片太大了，请选择小于 2MB 的图片！")
    return
  }
  const reader = new FileReader()
  reader.onload = (e) => {
    editForm.value.avatar = e.target.result
  }
  reader.readAsDataURL(file)
}

const saveUserInfo = () => {
  userInfo.value = { ...editForm.value }
  localStorage.setItem('userInfo', JSON.stringify(userInfo.value))
  showEditModal.value = false
}

const uploadPhoto = (event) => {
  const file = event.target.files[0]
  if (!file) return
  if (file.size > 2 * 1024 * 1024) {
    alert("照片太大了，请选择小于 2MB 的照片！")
    event.target.value = '' 
    return
  }
  const reader = new FileReader()
  reader.onload = (e) => {
    userInfo.value.album.push(e.target.result)
    try {
      localStorage.setItem('userInfo', JSON.stringify(userInfo.value))
    } catch (error) {
      alert("您的浏览器存储空间已满，无法保存更多照片啦！")
      userInfo.value.album.pop() 
    }
    event.target.value = '' 
  }
  reader.readAsDataURL(file)
}

const favorites = ref(['西藏拉萨', '新疆伊犁', '云南大理'])
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
  width: 100%;
  box-sizing: border-box;
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

.avatar-circle {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  overflow: hidden;
  border: 4px solid #fff;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  flex-shrink: 0;
}

.avatar {
  width: 100%;
  height: 100%;
  object-fit: cover;
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
  background: transparent;
  color: #94a3b8;
  border: none;
  padding: 0;
  font-size: 16px;
}
.edit-btn:hover { 
  text-decoration: underline;
  color: #64748b;
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
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  box-sizing: border-box;
}
.upload-btn:hover {
  background: #f1f5f9;
  color: #0284c7;
  border-color: #0284c7;
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

.avatar-preview {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid #e2e8f0;
}

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

/* --- 开关按钮核心样式 --- */
.switch {
  position: relative;
  display: inline-block;
  width: 52px;
  height: 28px;
  flex-shrink: 0;
}

.switch input { 
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0; left: 0; right: 0; bottom: 0;
  background-color: #cbd5e1;
  transition: .4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 20px;
  width: 20px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  transition: .4s;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

input:checked + .slider {
  background-color: #3b82f6;
}

input:checked + .slider:before {
  transform: translateX(24px);
}

.slider.round {
  border-radius: 34px;
}

.slider.round:before {
  border-radius: 50%;
}
</style>