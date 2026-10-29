<template>
  <div class="container">
    <h2>AI 旅游助手 - 接口测试</h2>
    
    <button class="call-btn" :disabled="loading" @click="callAI">
      {{ loading ? '呼叫中...' : '呼叫 AI' }}
    </button>

    <div class="result-box">
      <p v-if="reply" class="reply">🤖 AI回复：{{ reply }}</p >
      <p v-if="error" class="error">❌ 报错：{{ error }}</p >
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const loading = ref(false)
const reply = ref('')
const error = ref('')

async function callAI() {
  loading.value = true
  reply.value = ''
  error.value = ''

  try {
    

    const res = await fetch('/api/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model: 'deepseek-chat',
        messages: [
          { role: 'user', content: '你好，请用一句话介绍你自己' }
        ]
      })
    })

    if (!res.ok) {
      throw new Error(`请求失败，状态码: ${res.status}`)
    }

    const data = await res.json()
    // 解析 DeepSeek 返回的结果
    reply.value = data.choices[0].message.content

  } catch (err) {
    error.value = err.message || '发生未知错误'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 50px auto;
  text-align: center;
  font-family: sans-serif;
}
.call-btn {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #3b82f6;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}
.call-btn:disabled {
  background-color: #9ca3af;
  cursor: not-allowed;
}
.result-box {
  margin-top: 20px;
  padding: 20px;
  background-color: #f3f4f6;
  border-radius: 8px;
  text-align: left;
  min-height: 100px;
}
.reply {
  color: #1f2937;
  line-height: 1.6;
}
.error {
  color: #ef4444;
  font-weight: bold;
}
</style>
