<template>
  <div class="app-layout">
    <!-- 地图页 -->
    <div v-show="!currentProvince" class="map-view">
      <div class="map-header">大美中国 旅游推荐</div>
      <div ref="mapContainer" class="chart-container"></div>
    </div>

    <!-- 省份详情页 -->
   <!-- 省份详情页 -->
<div v-if="currentProvince" class="province-view" :style="{ '--theme-color': currentProvince.themeColor }">
  

  <header class="detail-header">
    <div class="logo">大美中国 <span>{{ currentProvince.name }}旅游推荐</span></div>
    <nav class="detail-nav">
      <a @click="currentTab = 'spots'" :class="{ active: currentTab === 'spots' }">景点介绍</a >
      <a @click="currentTab = 'foods'" :class="{ active: currentTab === 'foods' }">美食推荐</a >
      <a @click="currentTab = 'guide'" :class="{ active: currentTab === 'guide' }">旅游指南</a >
      <a @click="currentTab = 'message'" :class="{ active: currentTab === 'message' }">在线留言</a >
      <a @click="currentTab = 'login'" :class="{ active: currentTab === 'login' }">登录注册</a >
    </nav>
    <div class="back-btn" @click="currentProvince = null">← 返回地图</div>
  </header>
 

 
      
      <!-- 轮播图 -->
      <div class="carousel-banner">
        <div 
          class="carousel-item" 
          v-for="(img, index) in currentProvince.banners" 
          :key="index"
          :class="{ 'active-slide': currentIndex === index }"
          :style="{ backgroundImage: `url(${img})`, opacity: currentIndex === index ? 1 : 0 }"
        ></div>
        <div class="banner-mask">
          <h1>{{ currentProvince.name }}</h1>
          <p>{{ currentProvince.intro }}</p >
        </div>
      </div>

      <!-- 内容区 -->
      <transition name="fade-slide" mode="out-in">
        <div class="tab-content" :key="currentTab">
          
          <!-- 景点介绍 -->
          <div v-show="currentTab === 'spots'" class="spots-pane">
            <h2 class="section-title"><span class="bar"></span>十大必去景点</h2>
            <div class="spots-list">
              <div class="spot-card" v-for="(item, index) in currentProvince.spots" :key="index" :class="{ 'reverse': index % 2 !== 0 }">
                <div class="spot-img" :style="{ backgroundImage: `url(${item.imgUrl})` }"></div>
                <div class="spot-info">
                  <h3>{{ item.title }}</h3>
                  <p>{{ item.desc }}</p >
                  <div class="tips">
                    <strong>游玩贴士：</strong>
                    <span v-for="(tip, tIdx) in item.tips" :key="tIdx" class="tip-item">{{ tip }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- 美食推荐 -->
          <div v-show="currentTab === 'foods'" class="foods-pane">
            <h2 class="section-title"><span class="bar"></span>特色美食推荐</h2>
            <div class="foods-list">
              <div class="food-card" v-for="(food, index) in currentProvince.foods" :key="index">
                <div class="food-img" :style="{ backgroundImage: `url(${food.imgUrl})` }"></div>
                <div class="food-info">
                  <h3>{{ food.name }}</h3>
                  <p>{{ food.desc }}</p >
                </div>
              </div>
            </div>
          </div>

          <!-- 旅游指南 -->
          <div v-show="currentTab === 'guide'" class="guide-pane">
            <h2 class="section-title"><span class="bar"></span>旅游指南</h2>
            <ul class="guide-list">
              <li v-for="(guide, index) in currentProvince.guides" :key="index">
                <span class="dot"></span> {{ guide }}
              </li>
            </ul>
          </div>

          <!-- 在线留言板 -->
          <div v-show="currentTab === 'message'" class="message-pane">
            <h2 class="section-title"><span class="bar"></span>在线留言板</h2>
            <div class="message-board">
              <!-- 示例区 -->
              <div class="sample-section">
                <div class="sample-title">💡 留言参考示例</div>
                <div class="sample-msg-item">
                  <div class="sample-avatar">游</div>
                  <div class="sample-content">
                    <div class="sample-header"><span class="sample-user">游客A</span><span class="sample-time">示例参考</span></div>
                    <div class="sample-text">刚去了{{ currentProvince.name }}，风景绝美，大家一定要去看看！</div>
                  </div>
                </div>
              </div>
              <div class="divider"></div>
              
              <!-- 真实列表区 -->
              <div class="msg-list">
                <div class="msg-item" v-for="msg in pagedMessages" :key="msg.id">
                  <div class="avatar" :style="{ background: msg.user === '我' ? 'var(--theme-color)' : '#e0e0e0' }">{{ msg.user.charAt(0) }}</div>
                  <div class="msg-content">
                    <div class="msg-header">
                      <div class="user-info"><span class="msg-user">{{ msg.user }}</span><span class="msg-time">{{ msg.time }}</span></div>
                      <span v-if="msg.isMine" class="delete-btn" @click="confirmDelete(msg)">🗑️ 删除</span>
                    </div>
                    <div class="msg-text">{{ msg.text }}</div>
                  </div>
                </div>
                <div v-if="messages.length === 0" class="empty-state">
                  <div class="empty-icon">📝</div>
                  <div class="empty-text">暂无真实留言，快来发表第一条旅行感受吧~</div>
                </div>
              </div>

              <!-- 分页 -->
              <div class="pagination" v-if="messages.length > pageSize">
                <button :disabled="currentPage === 1" @click="currentPage--">上一页</button>
                <span>{{ currentPage }} / {{ totalPages }}</span>
                <button :disabled="currentPage === totalPages" @click="currentPage++">下一页</button>
              </div>

              <!-- 输入框 -->
              <div class="msg-input-box">
                <textarea v-model="newMsg" placeholder="文明发言，分享你的旅行感受或提问..."></textarea>
                <div class="input-actions">
                  <span class="char-count">{{ newMsg.length }}/200</span>
                  <button @click="submitMsg">发布留言</button>
                </div>
              </div>
            </div>
          </div>

          <!-- 登录注册 -->
          <div v-show="currentTab === 'login'" class="login-pane">
            <div class="login-box">
              <h2>{{ isLogin ? '登录账号' : '注册账号' }}</h2>
              <input type="text" placeholder="请输入手机号/邮箱" />
              <input type="password" placeholder="请输入密码" />
              <button class="submit-btn">{{ isLogin ? '登 录' : '注 册' }}</button>
              <p class="toggle-text" @click="isLogin = !isLogin">{{ isLogin ? '没有账号？点击注册' : '已有账号？返回登录' }}</p >
            </div>
          </div>

        </div>
      </transition>
    </div>

    <!-- 删除确认弹窗 -->
    <div class="modal-overlay" v-if="showDeleteModal">
      <div class="modal-box">
        <h3>⚠️ 确认删除</h3>
        <p>确定要删除这条留言吗？此操作不可恢复。</p >
        <div class="modal-actions">
          <button @click="showDeleteModal = false">取消</button>
          <button class="danger" @click="executeDelete">删除</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, onMounted, onBeforeUnmount, watch, computed } from 'vue'
import * as echarts from 'echarts'
import chinaJson from '../assets/map/china.json' // 注意这里的路径是 ../
import { provincesData } from './data/travelData.js'

const mapContainer = ref(null)
let myChart = null
const currentProvince = ref(null)
const currentTab = ref('spots')
const currentIndex = ref(0)
let timer = null

// 留言板逻辑
const currentUser = ref('我')
const newMsg = ref('')
const messages = ref([])
const currentPage = ref(1)
const pageSize = 5
const showDeleteModal = ref(false)
const msgToDelete = ref(null)

const badWords = ['傻', '笨蛋', '垃圾']

const totalPages = computed(() => Math.ceil(messages.value.length / pageSize) || 1)
const pagedMessages = computed(() => {
  const start = (currentPage.value - 1) * pageSize
  return messages.value.slice(start, start + pageSize)
})

const formatTime = (timestamp) => {
  const d = new Date(timestamp)
  const pad = (n) => n.toString().padStart(2, '0')
  return `${d.getFullYear()}-${pad(d.getMonth()+1)}-${pad(d.getDate())} ${pad(d.getHours())}:${pad(d.getMinutes())}`
}

watch(currentProvince, (newVal) => {
  if (newVal) {
    currentPage.value = 1
    const saved = localStorage.getItem(`msgs_${newVal.name}`)
    if (saved) {
      messages.value = JSON.parse(saved)
    } else {
      messages.value = []
    }
  }
}, { immediate: true })

const saveToLocal = () => {
  if (currentProvince.value) {
    localStorage.setItem(`msgs_${currentProvince.value.name}`, JSON.stringify(messages.value))
  }
}

const submitMsg = () => {
  const text = newMsg.value.trim()
  if (!text) return alert('请输入留言内容，不能全是空格哦~')
  if (text.length > 200) return alert('留言内容不能超过200字！')
  if (badWords.some(word => text.includes(word))) return alert('留言包含敏感词，请文明发言！')

  messages.value.unshift({
    id: Date.now(),
    user: currentUser.value,
    text: text,
    time: formatTime(Date.now()),
    isMine: true
  })
  saveToLocal()
  newMsg.value = ''
  currentPage.value = 1
}

const confirmDelete = (msg) => {
  msgToDelete.value = msg
  showDeleteModal.value = true
}

const executeDelete = () => {
  messages.value = messages.value.filter(m => m.id !== msgToDelete.value.id)
  saveToLocal()
  showDeleteModal.value = false
  msgToDelete.value = null
  if (pagedMessages.value.length === 0 && currentPage.value > 1) {
    currentPage.value--
  }
}

// 地图初始化
onMounted(() => {
  timer = setInterval(() => {
    if (currentProvince.value) currentIndex.value = (currentIndex.value + 1) % 4
  }, 4000)

  setTimeout(() => {
    if (!mapContainer.value) return

    myChart = echarts.init(mapContainer.value)
    echarts.registerMap('china', chinaJson)
    
    const scatterData = Object.keys(provincesData).map(name => ({ 
      name, value: provincesData[name].center 
    }))

    myChart.setOption({
      geo: { 
        map: 'china', roam: true, zoom: 1.2,
        selectedMode: 'single',
        itemStyle: { areaColor: '#f7f9f9', borderColor: '#dcdcdc' },
        emphasis: { 
          label: { show: true, color: '#333' },
          itemStyle: { areaColor: '#e0f3f8', borderColor: '#87ceeb', borderWidth: 1.5 }
        },
        select: { 
          label: { show: true, color: '#000' },
          itemStyle: { areaColor: '#fdf6d9', borderColor: '#d97706', borderWidth: 2.5, shadowBlur: 15, shadowColor: 'rgba(217, 119, 6, 0.5)' }
        }
      },
      series: [
        { type: 'map', map: 'china', geoIndex: 0 },
        { type: 'scatter', coordinateSystem: 'geo', data: scatterData, symbol: 'pin', symbolSize: 15, itemStyle: { color: '#ff4d4f', borderColor: '#ffffff' }, emphasis: { scale: 1.2 } }
      ]
    })

    myChart.on('click', (params) => {
      let dataKey = Object.keys(provincesData).find(key => key.includes(params.name))
      if (dataKey) {
        currentIndex.value = 0
        currentTab.value = 'spots'
        currentProvince.value = { name: dataKey, ...provincesData[dataKey] }
      }
    })
    myChart.resize()
  }, 300)
})

onBeforeUnmount(() => {
  if (myChart) myChart.dispose()
  clearInterval(timer)
})
</script>
<style scoped>
/* ========== 第一部分：基础布局 ========== */
.app-layout { width: 100%; height: 100%; background: #fff; overflow: hidden; font-family: 'Microsoft YaHei', sans-serif; position: relative; }
.map-view { width: 100%; height: 100%; display: flex; flex-direction: column; }
.map-header { height: 60px; background: #fff; border-bottom: 1px solid #eee; display: flex; align-items: center; padding: 0 40px; font-size: 20px; font-weight: bold; color: #2e7d32; }
.chart-container { flex: 1; width: 100%; cursor: pointer; }

.province-view { width: 100%; height: 100%; overflow-y: auto; background: #f9f9f9; position: absolute; top: 0; left: 0; z-index: 9999; }
.detail-header { 
  display: flex; align-items: center; justify-content: space-between; 
  padding: 0 40px; height: 70px; background: #fff; 
  border-bottom: 2px solid var(--theme-color); 
  position: sticky; top: 0; z-index: 100; 
}
.detail-header .logo { font-size: 20px; font-weight: bold; color: var(--theme-color); white-space: nowrap; }
.detail-header .logo span { font-size: 14px; color: #666; font-weight: normal; }
.detail-nav { display: flex; gap: 20px; }
.detail-nav a { cursor: pointer; padding: 10px 5px; font-size: 16px; color: #333; border-bottom: 3px solid transparent; transition: all 0.3s; white-space: nowrap; }
.detail-nav a:hover { color: var(--theme-color); transform: translateY(-2px); }
.detail-nav a.active { color: var(--theme-color); border-bottom-color: var(--theme-color); font-weight: bold; background: rgba(0,0,0,0.02); }
.back-btn { background: var(--theme-color); color: #fff; padding: 8px 16px; border-radius: 4px; cursor: pointer; transition: transform 0.2s; white-space: nowrap; }
.back-btn:hover { transform: scale(1.05); }
/* 轮播图 */
.carousel-banner { width: 100%; height: 400px; position: relative; overflow: hidden; }
.carousel-item { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background-size: cover; background-position: center; transition: opacity 1s ease-in-out, transform 4s linear; }
.carousel-item.active-slide { transform: scale(1.1); }
.banner-mask { position: absolute; bottom: 0; left: 0; width: 100%; height: 100%; background: linear-gradient(to right, rgba(0,0,0,0.7) 0%, rgba(0,0,0,0) 60%); display: flex; flex-direction: column; justify-content: center; padding-left: 50px; pointer-events: none; }
.banner-mask h1 { font-size: 50px; color: #fff; margin: 0 0 10px 0; text-shadow: 2px 2px 4px rgba(0,0,0,0.5); }
.banner-mask p { font-size: 18px; color: #eee; margin: 0; text-shadow: 1px 1px 2px rgba(0,0,0,0.5); }

/* ========== 第二部分：过渡动画与标题 ========== */
.tab-content { max-width: 1000px; margin: 0 auto; padding: 40px 20px; }
.fade-slide-enter-active, .fade-slide-leave-active { transition: opacity 0.3s ease, transform 0.3s ease; }
.fade-slide-enter-from { opacity: 0; transform: translateY(15px); }
.fade-slide-leave-to { opacity: 0; transform: translateY(-15px); }
.section-title { display: flex; align-items: center; margin-bottom: 30px; font-size: 24px; color: #333; }
.section-title .bar { width: 6px; height: 24px; background: var(--theme-color); margin-right: 12px; border-radius: 3px; }

/* ========== 第三部分：景点与美食卡片 ========== */
.spot-card { display: flex; background: #fff; margin-bottom: 30px; border-radius: 12px; overflow: hidden; box-shadow: 0 4px 15px rgba(0,0,0,0.05); height: 260px; border: 1px solid #f0f0f0; transition: all 0.4s cubic-bezier(0.165, 0.84, 0.44, 1); }
.spot-card:hover { transform: translateY(-8px); box-shadow: 0 15px 30px rgba(0,0,0,0.12); }
.spot-card.reverse { flex-direction: row-reverse; }
.spot-img { width: 40%; background-size: cover; background-position: center; transition: transform 0.6s ease; }
.spot-card:hover .spot-img { transform: scale(1.06); }
.spot-info { width: 60%; padding: 30px; display: flex; flex-direction: column; }
.spot-info h3 { font-size: 22px; margin: 0 0 15px 0; color: #222; border-left: 4px solid var(--theme-color); padding-left: 10px; }
.spot-info p { color: #555; line-height: 1.8; font-size: 15px; margin-bottom: 15px; flex: 1; text-align: justify; }
.tips { font-size: 13px; color: #444; background: #f8f9fa; padding: 15px; border-radius: 8px; margin-top: auto; border-left: 3px solid var(--theme-color); }
.tips strong { color: var(--theme-color); display: block; margin-bottom: 5px; }
.tip-item { display: block; margin-bottom: 3px; }

.foods-list { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; }
.food-card { background: #fff; border-radius: 12px; overflow: hidden; box-shadow: 0 4px 15px rgba(0,0,0,0.05); border: 1px solid #f0f0f0; transition: all 0.4s cubic-bezier(0.165, 0.84, 0.44, 1); }
.food-card:hover { transform: translateY(-8px); box-shadow: 0 15px 30px rgba(0,0,0,0.12); }
.food-img { width: 100%; height: 200px; background-size: cover; background-position: center; transition: transform 0.6s ease; }
.food-card:hover .food-img { transform: scale(1.06); }
.food-info { padding: 20px; }
.food-info h3 { margin: 0 0 10px 0; color: #333; font-size: 18px; transition: color 0.3s; }
.food-card:hover .food-info h3 { color: var(--theme-color); }
.food-info p { color: #666; font-size: 14px; line-height: 1.6; margin: 0; text-align: justify; }

/* ========== 第四部分：旅游指南 ========== */
.guide-list { background: #fff; padding: 30px 50px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); border: 1px solid #eee; list-style: none; }
.guide-list li { font-size: 16px; color: #444; line-height: 2.5; border-bottom: 1px dashed #eee; display: flex; align-items: center; transition: background 0.3s; padding: 0 10px; border-radius: 6px; }
.guide-list li:hover { background: #f9f9f9; }
.guide-list li:last-child { border-bottom: none; }
.dot { width: 6px; height: 6px; background: var(--theme-color); border-radius: 50%; margin-right: 12px; }
/* ========== 第五部分：留言板 ========== */
.message-board { background: #fff; padding: 30px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); }
.sample-section { background-color: #f7f9f9; border-radius: 8px; padding: 15px 20px; margin-bottom: 25px; }
.sample-title { font-size: 13px; color: #888; margin-bottom: 15px; font-weight: bold; }
.sample-msg-item { display: flex; gap: 12px; margin-bottom: 15px; }
.sample-msg-item:last-child { margin-bottom: 0; }
.sample-avatar { width: 35px; height: 35px; background: #e0e0e0; color: #888; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 14px; flex-shrink: 0; }
.sample-content { flex: 1; }
.sample-header { display: flex; align-items: center; gap: 10px; margin-bottom: 5px; }
.sample-user { font-weight: bold; color: #666; font-size: 14px; }
.sample-time { font-size: 12px; color: #aaa; }
.sample-text { color: #777; font-size: 14px; line-height: 1.5; }
.divider { height: 1px; background: #eee; margin: 25px 0; }

.msg-list { display: flex; flex-direction: column; gap: 15px; margin-bottom: 20px; }
.msg-item { display: flex; gap: 15px; padding: 20px; border-radius: 10px; background: #fdfdfd; border: 1px solid #f0f0f0; transition: box-shadow 0.3s; }
.msg-item:hover { box-shadow: 0 4px 12px rgba(0,0,0,0.05); }
.avatar { width: 45px; height: 45px; color: #fff; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 18px; font-weight: bold; flex-shrink: 0; }
.msg-content { flex: 1; }
.msg-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
.user-info { display: flex; align-items: center; gap: 10px; }
.msg-user { font-weight: bold; color: #333; font-size: 15px; }
.msg-time { font-size: 12px; color: #999; }
.delete-btn { font-size: 13px; color: #ff4d4f; cursor: pointer; padding: 4px 8px; border-radius: 4px; transition: background 0.3s; }
.delete-btn:hover { background: #fff1f0; }
.msg-text { color: #444; font-size: 15px; line-height: 1.6; }
.empty-state { display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 40px 0; color: #999; }
.empty-icon { font-size: 36px; margin-bottom: 12px; opacity: 0.4; }
.empty-text { font-size: 14px; letter-spacing: 1px; }

.pagination { display: flex; justify-content: center; align-items: center; gap: 15px; margin-bottom: 30px; font-size: 14px; color: #666; }
.pagination button { padding: 6px 15px; border: 1px solid #ddd; background: #fff; border-radius: 6px; cursor: pointer; transition: all 0.3s; }
.pagination button:disabled { opacity: 0.5; cursor: not-allowed; }
.pagination button:not(:disabled):hover { border-color: var(--theme-color); color: var(--theme-color); }

.msg-input-box { display: flex; flex-direction: column; border: 1px solid #e0e0e0; border-radius: 8px; background: #fafafa; padding: 10px; transition: border 0.3s; }
.msg-input-box:focus-within { border-color: var(--theme-color); background: #fff; }
.msg-input-box textarea { width: 100%; height: 60px; padding: 10px; border: none; background: transparent; resize: none; outline: none; font-size: 14px; font-family: inherit; }
.input-actions { display: flex; justify-content: space-between; align-items: center; margin-top: 10px; }
.char-count { font-size: 12px; color: #999; }
.input-actions button { padding: 8px 20px; background: var(--theme-color); color: #fff; border: none; border-radius: 6px; font-size: 14px; font-weight: bold; cursor: pointer; transition: opacity 0.3s, transform 0.2s; }
.input-actions button:hover { opacity: 0.9; transform: scale(1.02); }

/* ========== 第六部分：登录与弹窗 ========== */
.login-pane { display: flex; justify-content: center; padding: 50px 0; }
.login-box { width: 400px; background: #fff; padding: 40px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); text-align: center; }
.login-box h2 { color: #333; margin-bottom: 30px; }
.login-box input { width: 100%; padding: 15px; margin-bottom: 20px; border: 1px solid #ddd; border-radius: 8px; outline: none; box-sizing: border-box; font-size: 15px; transition: border 0.3s; }
.login-box input:focus { border-color: var(--theme-color); }
.submit-btn { width: 100%; padding: 15px; background: var(--theme-color); color: #fff; border: none; border-radius: 8px; font-size: 16px; font-weight: bold; cursor: pointer; transition: background 0.3s; }
.submit-btn:hover { opacity: 0.9; }
.toggle-text { margin-top: 20px; color: var(--theme-color); font-size: 14px; cursor: pointer; font-weight: bold; }

.modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center; z-index: 9999; backdrop-filter: blur(2px); }
.modal-box { background: #fff; width: 320px; padding: 25px; border-radius: 12px; text-align: center; box-shadow: 0 10px 30px rgba(0,0,0,0.2); animation: popIn 0.3s ease; }
@keyframes popIn { from { transform: scale(0.9); opacity: 0; } to { transform: scale(1); opacity: 1; } }
.modal-box h3 { margin: 0 0 15px 0; color: #333; font-size: 18px; }
.modal-box p { margin: 0 0 25px 0; color: #666; font-size: 14px; }
.modal-actions { display: flex; gap: 15px; }
.modal-actions button { flex: 1; padding: 10px; border-radius: 6px; font-size: 14px; cursor: pointer; border: none; transition: opacity 0.3s; }
.modal-actions button:first-child { background: #f0f0f0; color: #333; }
.modal-actions button.danger { background: #ff4d4f; color: #fff; }
.modal-actions button:hover { opacity: 0.8; }
</style>