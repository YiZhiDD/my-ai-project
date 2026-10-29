<template>
  <div class="app-container">
    <header class="top-header">
      <h2>✈️ 智能旅游规划系统</h2>
      <button @click="currentTab = currentTab === 'plan' ? 'map' : 'plan'">
        切换到 {{ currentTab === 'plan' ? '足迹地图' : '计划制定' }}
      </button>
    </header>

    <main class="main-content">
      <!-- 计划制定页 -->
      <div v-show="currentTab === 'plan'" class="plan-section">
        <h3>AI 行程规划</h3>
        <input v-model="destination" placeholder="输入目的地 (如: 三亚)" />
        <button @click="callAI" :disabled="loading">
          {{ loading ? '生成中...' : '生成计划' }}
        </button>
        <div class="result-box">
          <p v-if="error" style="color: red;">{{ error }}</p >
          <p v-if="reply">{{ reply }}</p >
        </div>
      </div>

      <!-- 地图页 -->
      <div v-show="currentTab === 'map'" class="map-section">
        <p>这里未来是队友 B 的 ECharts 地图...</p >
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const currentTab = ref('plan')
const destination = ref('')
const loading = ref(false)
const reply = ref('')
const error = ref('')

async function callAI() {
  if (!destination.value) {
    alert('请输入目的地！')
    return
  }
  loading.value = true
  reply.value = ''
  error.value = ''

  try {
    const res = await fetch('/api/chat/completions', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        model: 'deepseek-chat',
        messages: [
          { role: 'user', content: `请帮我制定一个去${destination.value}的3天旅游计划，包含景点和美食。` }
        ]
      })
    })
    if (!res.ok) throw new Error(`请求失败，状态码: ${res.status}`)
    const data = await res.json()
    reply.value = data.choices[0].message.content
  } catch (err) {
    error.value = err.message || '发生未知错误'
  } finally {
    loading.value = false
  }
}
</script>

<style>
.app-container { font-family: sans-serif; max-width: 800px; margin: 0 auto; }
.top-header { display: flex; justify-content: space-between; align-items: center; padding: 10px; background: #f0f0f0; border-radius: 8px; margin-bottom: 20px;}
.main-content { padding: 20px; border: 1px solid #ddd; border-radius: 8px; }
input { padding: 8px; width: 60%; margin-right: 10px; }
button { padding: 8px 15px; cursor: pointer; }
.result-box { margin-top: 20px; background: #fafafa; padding: 15px; min-height: 100px; white-space: pre-wrap; line-height: 1.6;}
</style>