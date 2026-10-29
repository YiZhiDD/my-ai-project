<template>
  <el-dialog v-model="dialogVisible" title="登录账号" width="400px" @close="handleClose">
    <el-tabs v-model="activeTab" class="demo-tabs">
      
      <!-- 手机号登录 Tab -->
      <el-tab-pane label="手机号登录" name="phone">
        <el-form class="login-form">
          <el-form-item>
            <el-input v-model="phone" placeholder="请输入手机号" maxlength="11" />
          </el-form-item>
          <el-form-item>
            <div style="display: flex; gap: 10px;">
              <el-input v-model="code" placeholder="请输入验证码" />
              <el-button :disabled="countdown > 0" @click="sendCode" type="primary" plain>
                {{ countdown > 0 ? `${countdown}s后重发` : '获取验证码' }}
              </el-button>
            </div>
          </el-form-item>
          <el-button type="primary" class="submit-btn" @click="handleLogin">登 录</el-button>
        </el-form>
      </el-tab-pane>

      <!-- 微信扫码 Tab -->
      <el-tab-pane label="微信扫码" name="wechat">
        <div class="scan-box">
          <div class="qr-placeholder">二维码占位</div>
          <p>请使用微信扫一扫登录</p >
        </div>
      </el-tab-pane>

      <!-- QQ登录 Tab -->
      <el-tab-pane label="QQ登录" name="qq">
        <div class="scan-box">
          <el-button type="info" class="submit-btn">前往 QQ 授权</el-button>
        </div>
      </el-tab-pane>

    </el-tabs>
  </el-dialog>
</template>

<script setup>
import { ref } from 'vue'

const dialogVisible = ref(false)
const activeTab = ref('phone')
const phone = ref('')
const code = ref('')
const countdown = ref(0)
let timer = null

const emit = defineEmits(['login-success'])

// 让父组件能控制弹窗开启
const openModal = () => {
  dialogVisible.value = true
}

// 倒计时逻辑
const sendCode = () => {
  if (!phone.value) return alert('请先输入手机号')
  countdown.value = 60
  timer = setInterval(() => {
    countdown.value--
    if (countdown.value <= 0) clearInterval(timer)
  }, 1000)
}

// 登录逻辑
const handleLogin = () => {
  if (!phone.value || !code.value) return alert('请填写完整信息')
  
  // 模拟存入本地
  localStorage.setItem('userToken', 'mock-token-12345')
  localStorage.setItem('userPhone', phone.value)
  
  alert('登录成功！')
  dialogVisible.value = false
  emit('login-success') // 告诉父组件登录成功了
}

const handleClose = () => {
  if (timer) clearInterval(timer)
}

// 暴露方法给父组件调用
defineExpose({ openModal })
</script>

<style scoped>
.login-form { margin-top: 20px; }
.submit-btn { width: 100%; border-radius: 8px; }
.scan-box { text-align: center; padding: 30px 0; color: #666; }
.qr-placeholder { width: 150px; height: 150px; background: #eee; margin: 0 auto 15px; display: flex; align-items: center; justify-content: center; border-radius: 8px; }
</style>