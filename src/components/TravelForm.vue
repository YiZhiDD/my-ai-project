<template>
  <div class="form-container">
    <h3>✈️ 规划你的专属旅程</h3>
    <div class="input-group">
      <label>目的地</label>
      <input v-model="destination" type="text" placeholder="想去哪里？例如：三亚、北京" />
    </div>
    <div class="input-group">
      <label>游玩天数</label>
      <input v-model="days" type="number" min="1" max="30" placeholder="计划游玩几天？" />
    </div>
    <button class="generate-btn" @click="handleGenerate">✨ 生成 AI 行程</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const destination = ref('')
const days = ref(3)

// 定义事件，把用户输入的数据传给父组件 (App.vue)
const emit = defineEmits(['generate'])

const handleGenerate = () => {
  if (!destination.value) {
    alert('请先输入目的地哦！')
    return
  }
  // 触发事件，把数据抛出去
  emit('generate', {
    destination: destination.value,
    days: days.value
  })
}
</script>

<style scoped>
.form-container {
  background: #ffffff;
  padding: 30px;
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08); /* 高级感阴影 */
  max-width: 450px;
  margin: 20px auto;
  font-family: 'PingFang SC', sans-serif;
}

h3 {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 25px;
  font-size: 1.4rem;
}

.input-group {
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
}

label {
  font-size: 0.9rem;
  color: #606266;
  margin-bottom: 8px;
  font-weight: bold;
}

input {
  padding: 12px 15px;
  border: 1px solid #dcdfe6;
  border-radius: 8px;
  font-size: 1rem;
  outline: none;
  transition: all 0.3s;
}

input:focus {
  border-color: #409eff;
  box-shadow: 0 0 8px rgba(64, 158, 255, 0.2);
}

.generate-btn {
  width: 100%;
  padding: 14px;
  background: linear-gradient(135deg, #409eff, #79bbff);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  font-weight: bold;
  cursor: pointer;
  margin-top: 10px;
  transition: transform 0.2s, box-shadow 0.2s;
}

.generate-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 15px rgba(64, 158, 255, 0.4);
}
</style>