<!--
  途笺 · WAYFARER — AI 旅行计划生成器
  Vue3 + Vite + Element Plus + DeepSeek Chat Completions
  依赖：element-plus（main.js 全局注册即可，无需 icons 包）
-->
<template>
  <div class="app-shell">
    <!-- ============ 顶部：品牌 + 搜索 + 登录 ============ -->
    <header class="topbar">
      <div class="topbar-inner">
        <a class="brand" href="javascript:;" @click.prevent>
          <svg class="mark" viewBox="0 0 32 32" aria-hidden="true">
            <circle cx="16" cy="16" r="14.2" fill="none" stroke="currentColor" stroke-width="1.4" />
            <path d="M16 5.5 L20.2 16 L16 26.5 L11.8 16 Z" fill="currentColor" />
            <circle cx="16" cy="16" r="2.3" fill="var(--paper)" />
          </svg>
          <span class="brand-text">
            <strong>途笺</strong>
            <em>WAYFARER · AI TRIP STUDIO</em>
          </span>
        </a>
       

        <div class="search">
          <el-input
            v-model="keyword"
            class="search-input"
            size="large"
            placeholder="搜索城市、目的地或玩法灵感…"
            clearable
            @keyup.enter="onSearch"
          >
            <template #prefix>
              <svg class="ico" viewBox="0 0 24 24" aria-hidden="true">
                <circle cx="11" cy="11" r="6.4" fill="none" stroke="currentColor" stroke-width="1.7" />
                <path d="M16 16 L20.5 20.5" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" />
              </svg>
            </template>
          </el-input>
        </div>

        <div class="account">
          <template v-if="user">
            <div class="user-chip">
              <span class="avatar">{{ user.name.slice(0, 1) }}</span>
              <span class="uname">{{ user.name }}</span>
              <button class="link-btn" type="button" @click="logout">退出</button>
            </div>
          </template>
          <el-button v-else class="btn-login" size="large" @click="loginVisible = true">登录</el-button>
        </div>
      </div>

      <!-- ============ 导航标签 ============ -->
      <nav class="tabs">
        <button
          v-for="t in tabs"
          :key="t.key"
          type="button"
          class="tab"
          :class="{ 'is-active': activeTab === t.key }"
          @click="activeTab = t.key"
        >
          <i class="idx">{{ t.idx }}</i>{{ t.label }}
        </button>
        <span class="tabs-hint mono">CURRENT · {{ currentTabLabel }}</span>
      </nav>
    </header>


    <!-- ============ 主体 ============ -->
    <main class="stage">
      <!-- ---- 计划制定（默认 / 已实现） ---- -->
      <section v-if="activeTab === 'plan'" class="panel">
        <div class="panel-head">
          <span class="eyebrow mono">STEP 01 — ROUTE &amp; BUDGET</span>
          <h1>把想法交给 AI，<br />把路线握在手里。</h1>
          <p class="lede">
            填写出发地、目的地与天数，AI 将生成包含交通方式、打卡地点、住宿推荐与逐日路线的完整方案。
          </p>
          <div class="chips">
            <span class="chips-label mono">试试示例</span>
            <button
              v-for="s in samples"
              :key="s.origin + s.destination"
              type="button"
              class="chip"
              @click="fillSample(s)"
            >
              {{ s.origin }} → {{ s.destination }} · {{ s.days }}天
            </button>
          </div>
        </div>

        <!-- 表单卡片 -->
        <div class="card form-card">
          <div class="card-tag mono">TRIP BRIEF</div>
          <div class="form-grid">
            <div class="field">
              <label>出发地</label>
              <el-input v-model="form.origin" size="large" placeholder="如：北京" maxlength="20" />
            </div>
            <div class="field">
              <label>目的地</label>
              <el-input v-model="form.destination" size="large" placeholder="如：成都" maxlength="20" />
            </div>
            <div class="field field-days">
              <label>理想花费时间</label>
              <el-input v-model="form.days" size="large" placeholder="5" class="input-days">
                <template #suffix><span class="unit">天</span></template>
              </el-input>
            </div>
          </div>

          <div class="form-actions">
            <el-button
              class="btn-generate"
              size="large"
              :loading="loading"
              :disabled="loading"
              @click="generatePlan"
            >
              <svg v-if="!loading" class="ico" viewBox="0 0 24 24" aria-hidden="true">
                <path d="M12 2.6 L14.2 9.2 L20.8 11.4 L14.2 13.6 L12 20.2 L9.8 13.6 L3.2 11.4 L9.8 9.2 Z" fill="currentColor" />
              </svg>
              <span>{{ loading ? '正在生成…' : '一键生成旅行计划' }}</span>
            </el-button>
            <p class="hint">
              由 DeepSeek 驱动 · 生成内容仅供参考，请以实际票务与天气为准
            </p>
          </div>
        </div>

<!<ChinaMap />
        <!-- 结果卡片 -->
        <section class="card result-card" ref="resultCard">
          <div class="result-head">
            <div class="result-title">
              <span class="pulse" :class="{ on: loading }"></span>
              AI 行程方案
              <span v-if="isDemo" class="badge-demo mono">示例数据</span>
            </div>
            <div class="result-meta">
              <span v-if="routeSummary" class="meta-chip mono">{{ routeSummary }}</span>
              <span v-if="elapsed" class="meta-chip mono">耗时 {{ elapsed }}s</span>
              <button v-if="planContent" type="button" class="ghost-btn" @click="copyPlan">复制</button>
              <button v-if="planContent" type="button" class="ghost-btn" @click="resetPlan">清空</button>
            </div>>
          </div>

          <div class="result-body">
            <!-- 空状态 -->
            <div v-if="!planContent && !loading" class="empty">
              <svg class="empty-art" viewBox="0 0 320 140" aria-hidden="true">
                <path d="M40 96 C 110 20, 210 20, 280 78" fill="none" stroke="var(--line-strong)" stroke-width="1.6" stroke-dasharray="5 7" />
                <g fill="var(--teal)">
                  <path d="M40 96 c-7-11-11-17-11-23 a11 11 0 0 1 22 0 c0 6-4 12-11 23 Z" />
                  <circle cx="40" cy="73" r="4" fill="var(--paper)" />
                </g>
                <g fill="var(--clay)">
                  <path d="M280 78 c-7-11-11-17-11-23 a11 11 0 0 1 22 0 c0 6-4 12-11 23 Z" />
                  <circle cx="280" cy="55" r="4" fill="var(--paper)" />
                </g>
                <circle cx="160" cy="42" r="3" fill="var(--amber)" />
                <circle cx="200" cy="33" r="2" fill="var(--amber)" opacity=".6" />
              </svg>
              <h3>你的行程将在这里展开</h3>
              <p>填写左侧信息后点击生成，AI 会输出一份可直接照着走的日程安排。</p>
            </div>

            <!-- 加载状态 -->
            <div v-else-if="loading" class="loading">
              <el-skeleton :rows="6" animated>
                <template #template>
                  <div class="sk-wrap">
                    <el-skeleton-item variant="h3" style="width: 42%" />
                    <el-skeleton-item variant="text" style="width: 88%" />
                    <el-skeleton-item variant="text" style="width: 76%" />
                    <el-skeleton-item variant="text" style="width: 92%" />
                    <el-skeleton-item variant="text" style="width: 60%" />
                    <el-skeleton-item variant="text" style="width: 80%" />
                  </div>
                </template>
              </el-skeleton>
              <div class="thinking mono">
                <span class="spin"></span>{{ thinkingStep }}
              </div>
            </div>

            <!-- 结果 Markdown -->
            <article v-else class="md" v-html="renderedPlan"></article>
          </div>
        </section>
      </section>
      <!-- 2. 智能化推荐页面 -->
    <section v-if="activeTab === 'rec'" class="panel" style="padding: 100px 20px; text-align: center;">
      <h1>📍 智能化推荐</h1>
      <p style="color: #666;">（地图区域，等待拼装）</p >
    </section>



      <!-- ---- 智能化推荐（占位） ---- -->
      <section v-else-if="activeTab === 'rec'" class="panel">
        <div class="panel-head">
          <span class="eyebrow mono">STEP 02 — RECOMMEND</span>
          <h1>智能化推荐</h1>
          <p class="lede">基于你的偏好与足迹推荐目的地。功能开发中，以下为占位假数据。</p>
        </div>
        <div class="rec-grid">
          <article v-for="(r, i) in fakeRecs" :key="r.name" class="rec-card" :style="{ '--i': i }">
            <div class="rec-cover" :class="r.tone"></div>
            <div class="rec-info">
              <h3>{{ r.name }}</h3>
              <p>{{ r.desc }}</p>
              <div class="rec-tags">
                <span v-for="t in r.tags" :key="t">{{ t }}</span>
              </div>
            </div>
          </article>
        </div>
      </section>

      <!-- ---- 设置（占位） ---- -->
     <section v-if="activeTab === 'settings'" class="panel">
        <div class="panel-head">
          <span class="eyebrow mono">STEP 03 — PREFERENCES</span>
          <h1>设置</h1>
          <p class="lede">调整出行偏好，让生成结果更贴合你。当前为演示开关，暂未持久化。</p>
        </div>
        <div class="card settings-card">
          <div v-for="s in fakeSettings" :key="s.key" class="setting-row">
            <div>
              <h4>{{ s.label }}</h4>
              <p>{{ s.desc }}</p>
            </div>
            <el-switch v-model="s.value" :disabled="s.disabled" />
          </div>
        </div>
      </section>
       <!-- 3. 设置页面 -->
    <section v-if="activeTab === 'settings'" class="panel" style="padding: 100px 20px; text-align: center;">
      <h1>⚙️ 个人设置</h1>
      <p style="color: #666;">（个人信息区域，等待拼装）</p >
    </section>
   
   <!-- 4. 旅行组队论坛 -->
      <section v-if="activeTab === 'team'" class="panel" style="padding: 20px;">
       <TeamForum />
      </section>
    </main>

    <footer class="foot mono">
      <span>旅途笺记 · 演示项目</span>
      <span>MODEL: deepseek-chat · STREAM: ON</span>
    </footer>

    <!-- 登录弹窗（假登录） -->
    <el-dialog v-model="loginVisible" width="380px" class="login-dialog" :show-close="false" align-center>
      <div class="login-box">
        <span class="eyebrow mono">SIGN IN</span>
        <h3>登录途笺</h3>
        <p>演示环境，任意输入即可进入。</p>
        <el-input v-model="loginForm.phone" size="large" placeholder="手机号" maxlength="11" />
        <el-input v-model="loginForm.code" size="large" placeholder="验证码（随意）" maxlength="6" />
        <el-button class="btn-generate" size="large" type="primary" @click="doLogin">进入</el-button>
      </div>
    </el-dialog>
  </div>
</template>


<script setup>
import { ref, reactive, computed, onBeforeUnmount, nextTick } from 'vue'
import { ElMessage } from 'element-plus'
import TeamForum from './components/TeamForum.vue'


/* ---------------- 基础状态 ---------------- */
const tabs = [
  { key: 'rec', idx: '01', label: '智能化推荐' },
  { key: 'plan', idx: '02', label: '计划制定' },
  { key: 'team', idx: '03', label: '旅行组队' },
  { key: 'settings', idx: '04', label: '设置' },
]
const activeTab = ref('plan') // 默认选中「计划制定」
const currentTabLabel = computed(() => tabs.find(t => t.key === activeTab.value)?.label ?? '')

const keyword = ref('')
const user = ref(null)
const loginVisible = ref(false)
const loginForm = reactive({ phone: '', code: '' })

const form = reactive({ origin: '', destination: '', days: '' })
const samples = [
  { origin: '北京', destination: '成都', days: '5' },
  { origin: '上海', destination: '西安', days: '4' },
  { origin: '广州', destination: '三亚', days: '3' },
]

const planContent = ref('')
const loading = ref(false)
const isDemo = ref(false)
const elapsed = ref('')
const resultCard = ref(null)

const routeSummary = computed(() =>
  form.origin && form.destination && form.days
    ? `${form.origin} → ${form.destination} / ${form.days}天`
    : ''
)

/* ---------------- 假数据 ---------------- */
const fakeRecs = [
  { name: '大理 · 洱海西岸', desc: '环海骑行、白族古镇与黄昏时分的生态廊道。', tags: ['慢节奏', '摄影', '3–4天'], tone: 'tone-a' },
  { name: '重庆 · 山城步道', desc: '立体魔幻城市，火锅、江景与老街区的烟火气。', tags: ['美食', '城市漫步', '2–3天'], tone: 'tone-b' },
  { name: '敦煌 · 河西走廊', desc: '莫高窟、鸣沙山与雅丹地貌，一条线路读懂千年。', tags: ['人文', '自驾', '5–6天'], tone: 'tone-c' },
]
const fakeSettings = reactive([
  { key: 'pace', label: '节奏', desc: '轻松漫游 / 标准紧凑 /特种兵暴走', value: true, disabled: false },
  { key: 'budget', label: '优先经济型方案', desc: '优先推荐性价比交通与住宿', value: false, disabled: false },
  { key: 'map', label: '启用中国地图可视化', desc: '功能开发中，敬请期待', value: false, disabled: true },
  { key: 'notify', label: '行程变更提醒', desc: '功能开发中，敬请期待', value: false, disabled: true },
])

/* ---------------- 交互 ---------------- */
function onSearch() {
  if (!keyword.value.trim()) return
  ElMessage.info(`「${keyword.value.trim()}」的搜索功能开发中，先试试「计划制定」吧`)
}
function fillSample(s) {
  Object.assign(form, s)
  ElMessage.success('已填入示例路线')
}
function doLogin() {
  if (!loginForm.phone.trim()) return ElMessage.warning('请输入手机号')
  user.value = { name: '旅行者' + loginForm.phone.slice(-4) }
  loginVisible.value = false
  ElMessage.success('登录成功（演示）')
}
function logout() {
  user.value = null
  ElMessage.info('已退出登录')
}
function resetPlan() {
  planContent.value = ''
  elapsed.value = ''
  isDemo.value = false
}
async function copyPlan() {
  try {
    await navigator.clipboard.writeText(planContent.value)
    ElMessage.success('已复制到剪贴板')
  } catch {
    ElMessage.warning('复制失败，请手动选择文本')
  }
}

/* ---------------- AI 调用（DeepSeek / SSE 流式） ---------------- */
// 若在 .env 配置了 VITE_DEEPSEEK_API_KEY 则前端直连；
// 若在 Vite Proxy 注入 Authorization，则无需该变量，代码会自动省略请求头。
const API_KEY = import.meta.env.VITE_DEEPSEEK_API_KEY || ''
const SYSTEM_PROMPT =
  '你是一位经验丰富的中国旅行规划师。请用简体中文输出结构清晰的行程方案，' +
  '使用 Markdown 格式：包含行程概览、逐日安排（Day 1/Day 2…）、交通方式、打卡地点、住宿推荐、预算参考与注意事项。' +
  '内容要具体、可执行，避免空话。'

const thinkingStep = ref('正在理解你的出行需求…')
const THINKING_STEPS = [
  '正在理解你的出行需求…',
  '正在比对交通方式与耗时…',
  '正在挑选打卡地点…',
  '正在匹配住宿与预算…',
  '正在编排逐日路线…',
]
let thinkTimer = null

function startThinking() {
  let i = 0
  thinkingStep.value = THINKING_STEPS[0]
  thinkTimer = setInterval(() => {
    i = (i + 1) % THINKING_STEPS.length
    thinkingStep.value = THINKING_STEPS[i]
  }, 1600)
}
function stopThinking() {
  clearInterval(thinkTimer)
  thinkTimer = null
}
onBeforeUnmount(stopThinking)

function buildPrompt() {
  const { origin, destination, days } = form
  return (
    `请帮我制定从${origin}到${destination}游玩${days}天的详细旅行计划，` +
    `包括交通方式、打卡地点、住宿推荐和路线规划。` +
    `出发地：${origin}；目的地：${destination}；天数：${days}天；` +
    `出行偏好：${fakeSettings[0].value ? '轻松漫游' : '标准紧凑'}，` +
    `${fakeSettings[1].value ? '优先经济型' : '不限预算'}。请给出逐日安排。`
  )
}

async function generatePlan() {
  const { origin, destination, days } = form
  if (!origin.trim() || !destination.trim()) return ElMessage.warning('请填写出发地与目的地')
  if (!/^\d+$/.test(String(days).trim()) || Number(days) < 1 || Number(days) > 60)
    return ElMessage.warning('请输入 1–60 之间的天数')

  loading.value = true
  isDemo.value = false
  planContent.value = ''
  elapsed.value = ''
  startThinking()

  const t0 = performance.now()
  await nextTick()
  resultCard.value?.scrollIntoView({ behavior: 'smooth', block: 'start' })

  const headers = { 'Content-Type': 'application/json' }
  if (API_KEY) headers.Authorization = `Bearer ${API_KEY}`

  try {
    const res = await fetch('/api/chat/completions', {
      method: 'POST',
      headers,
      body: JSON.stringify({
        model: 'deepseek-chat',
        stream: true,
        temperature: 0.7,
        messages: [
          { role: 'system', content: SYSTEM_PROMPT },
          { role: 'user', content: buildPrompt() },
        ],
      }),
    })

    if (!res.ok) {
      const errText = await res.text().catch(() => '')
      throw new Error(`HTTP ${res.status}${errText ? ' · ' + errText.slice(0, 120) : ''}`)
    }

    // 非流式兜底
    if (!res.body) {
      const data = await res.json()
      planContent.value = data?.choices?.[0]?.message?.content ?? ''
    } else {
      await readStream(res.body)
    }

    if (!planContent.value.trim()) throw new Error('接口返回内容为空')
  } catch (err) {
    console.error('[AI] 生成失败：', err)
    ElMessage.warning('接口暂不可用，已为你载入示例方案（演示模式）')
    await runDemo(origin, destination, days)
  } finally {
    stopThinking()
    loading.value = false
    elapsed.value = ((performance.now() - t0) / 1000).toFixed(1)
  }
}

/* 解析 SSE：data: {...} / data: [DONE] */
async function readStream(body) {
  const reader = body.getReader()
  const decoder = new TextDecoder('utf-8')
  let buffer = ''
  for (;;) {
    const { done, value } = await reader.read()
    if (done) break
    buffer += decoder.decode(value, { stream: true })
    const lines = buffer.split('\n')
    buffer = lines.pop() ?? ''
    for (const raw of lines) {
      const line = raw.trim()
      if (!line || !line.startsWith('data:')) continue
      const payload = line.slice(5).trim()
      if (payload === '[DONE]') continue
      try {
        const json = JSON.parse(payload)
        const delta = json?.choices?.[0]?.delta?.content
        if (delta) planContent.value += delta
      } catch {
        /* 忽略半包 */
      }
    }
  }
}

/* 接口不可用时的演示模式：本地打字机输出 */
async function runDemo(origin, destination, days) {
  isDemo.value = true
  const text = demoPlan(origin, destination, days)
  planContent.value = ''
  for (let i = 0; i < text.length; i += 6) {
    planContent.value += text.slice(i, i + 6)
    if (i % 120 === 0) await new Promise(r => setTimeout(r, 12))
  }
}

function demoPlan(o, d, days) {
  const n = Number(days)
  const daysBlock = Array.from({ length: Math.min(n, 6) }, (_, i) => {
    const day = i + 1
    const theme = ['城市初见 · 落地与老城', '经典地标 · 深度打卡', '周边秘境 · 自然风光', '在地生活 · 市井与夜色', '博物馆与文创园区', '自由活动 · 返程准备'][i % 6]
    return [
      `### Day ${day} · ${theme}`,
      `- **上午**：前往${d}${['核心老城区', '代表性历史街区', '近郊自然景区'][i % 3]}，建议 2–3 小时慢慢逛。`,
      `- **下午**：打卡${d}${['博物馆 / 美术馆', '观景台 / 古城墙', '特色主题街区'][i % 3]}，记得提前预约。`,
      `- **晚上**：回到住处附近觅食，推荐尝试当地${['小吃夜市', '老字号餐厅', '河畔餐吧'][i % 3]}。`,
    ].join('\n')
  }).join('\n\n')

  return [
    `## ${o} → ${d} · ${n}天旅行计划`,
    '',
    `> 出发地：${o} ｜ 目的地：${d} ｜ 天数：${n}天 ｜ 节奏：${fakeSettings[0].value ? '轻松漫游' : '标准紧凑'}`,
    '',
    '### 行程概览',
    `- 交通方式：**高铁 / 飞机**（建议提前 7–15 天购票，${o}出发主要有 ${['早班高铁', '直飞航班', '夜卧列车'][0]}可选）。`,
    '- 住宿建议：优先选择地铁站 800 米内、评分 4.7 以上的酒店或民宿，便于每日往返。',
    '- 预算参考：经济型约 600–900 元/人/天，舒适型约 1200–1800 元/人/天（含住宿、餐饮、门票与市内交通）。',
    '',
    daysBlock,
    '',
    '### 打卡地点清单',
    `- ${d}必去 TOP1：城市地标与观景平台（日落前 1 小时抵达最佳）。`,
    `- ${d}必去 TOP2：历史街区 / 博物馆（周一常闭馆，务必提前查开放时间）。`,
    `- ${d}必去 TOP3：本地市集或夜市（体验在地生活的最佳入口）。`,
    '',
    '### 住宿推荐',
    '- **位置优先**：选在交通枢纽或核心商圈，节省每日通勤时间。',
    '- **类型选择**：独行可选青旅单间，家庭出行优选两居民宿，商务感行程可选连锁中高端。',
    '',
    '### 注意事项',
    '- 出发前查看目的地天气，山区昼夜温差大，备一件薄外套。',
    '- 热门景点旺季需提前 1–3 天在官方渠道预约。',
    '- 保留 20% 的弹性时间，给偶遇的街角与咖啡馆。',
    '',
    '---',
    '*本方案由演示数据生成，正式接入 DeepSeek 后将替换为真实模型输出。*',
  ].join('\n')
}

/* ---------------- 轻量 Markdown 渲染（无需额外依赖） ---------------- */
function escapeHtml(s) {
  return s.replace(/[&<>"']/g, c => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]))
}
const inline = t =>
  t
    .replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>')
    .replace(/`([^`]+)`/g, '<code>$1</code>')
    .replace(/(^|[^*])\*([^*\n]+)\*/g, '$1<em>$2</em>')

function renderMarkdown(src) {
  const lines = escapeHtml(src || '').split(/\r?\n/)
  let html = ''
  let list = null // 'ul' | 'ol'
  const closeList = () => {
    if (list) {
      html += `</${list}>`
      list = null
    }
  }
  for (const raw of lines) {
    const line = raw.trim()
    if (!line) {
      closeList()
      continue
    }
    let m
    if ((m = line.match(/^(#{1,4})\s+(.*)$/))) {
      closeList()
      const lv = Math.min(m[1].length + 1, 5)
      html += `<h${lv}>${inline(m[2])}</h${lv}>`
      continue
    }
    if (/^(---|\*\*\*|___)\s*$/.test(line)) {
      closeList()
      html += '<hr />'
      continue
    }
    if ((m = line.match(/^[-*•]\s+(.*)$/))) {
      if (list !== 'ul') {
        closeList()
        html += '<ul>'
        list = 'ul'
      }
      html += `<li>${inline(m[1])}</li>`
      continue
    }
    if ((m = line.match(/^\d{1,2}[.、)]\s+(.*)$/))) {
      if (list !== 'ol') {
        closeList()
        html += '<ol>'
        list = 'ol'
      }
      html += `<li>${inline(m[1])}</li>`
      continue
    }
    if (line.startsWith('&gt;') || line.startsWith('***')) {
      closeList()
      html += `<blockquote>${inline(line.replace(/^(&gt;|\*\*\*)\s*/, ''))}</blockquote>`
      continue
    }
    closeList()
    html += `<p>${inline(line)}</p>`
  }
  closeList()
  return html
}
const renderedPlan = computed(() => renderMarkdown(planContent.value))
</script>

<!-- 全局：Element Plus 主题 + 字体 + 纸张质感 -->
<style>
@import url('
https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,600;9..144,700&family=DM+Mono:wght@400;500&family=Noto+Serif+SC:wght@500;700&display=swap
');

:root {
  /* Element Plus 品牌化 */
  --el-color-primary: #0e4f47;
  --el-color-primary-light-3: #4d8079;
  --el-color-primary-light-5: #7ba49e;
  --el-color-primary-light-7: #a9c6c2;
  --el-color-primary-light-8: #c2d7d4;
  --el-color-primary-light-9: #e3edeb;
  --el-color-primary-dark-2: #0a3a34;
  --el-border-radius-base: 6px;
  --el-font-family: 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;

  /* 纸张色板 */
  --paper: #f6f1e8;
  --paper-2: #fbf8f2;
  --ink: #1c2a28;
  --ink-soft: #566663;
  --teal: #0e4f47;
  --teal-deep: #0a3a34;
  --amber: #d98e4a;
  --clay: #c4623d;
  --line: #e6ddcc;
  --line-strong: #cfc3ac;

  --font-display: 'Fraunces', 'Noto Serif SC', 'Songti SC', 'STSong', serif;
  --font-cjk-serif: 'Noto Serif SC', 'Songti SC', 'STSong', serif;
  --font-mono: 'DM Mono', ui-monospace, SFMono-Regular, Menlo, monospace;
  --font-body: 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
}

html,
body {
  margin: 0;
  background: var(--paper);
  color: var(--ink);
  font-family: var(--font-body);
  -webkit-font-smoothing: antialiased;
}
body::before {
  content: '';
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 9999;
  opacity: 0.05;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='
http://www.w3.org/2000/svg
' width='140' height='140'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='3'/%3E%3C/filter%3E%3Crect width='140' height='140' filter='url(%23n)'/%3E%3C/svg%3E");
}
::selection {
  background: var(--teal);
  color: var(--paper);
}
.mono {
  font-family: var(--font-mono);
  letter-spacing: 0.06em;
}
</style>

<!-- 组件样式（scoped） -->
<style scoped>
* {
  box-sizing: border-box;
}
.app-shell {
  min-height: 100vh;
  background:
    radial-gradient(60rem 40rem at 88% -8%, rgba(14, 79, 71, 0.09), transparent 60%),
    radial-gradient(50rem 34rem at 4% 12%, rgba(217, 142, 74, 0.1), transparent 55%),
    var(--paper);
}

/* ---------- 顶栏 ---------- */
.topbar {
  position: sticky;
  top: 0;
  z-index: 50;
  backdrop-filter: blur(14px);
  background: rgba(246, 241, 232, 0.82);
  border-bottom: 1px solid var(--line);
}
.topbar-inner {
  max-width: 1180px;
  margin: 0 auto;
  padding: 18px 28px 14px;
  display: flex;
  align-items: center;
  gap: 22px;
}
.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  text-decoration: none;
  color: var(--teal);
  flex-shrink: 0;
}
.mark {
  width: 30px;
  height: 30px;
}
.brand-text {
  display: flex;
  flex-direction: column;
  line-height: 1.05;
}
.brand-text strong {
  font-family: var(--font-cjk-serif);
  font-size: 20px;
  letter-spacing: 0.14em;
  color: var(--ink);
}
.brand-text em {
  font-style: normal;
  font-family: var(--font-mono);
  font-size: 9px;
  letter-spacing: 0.22em;
  color: var(--ink-soft);
  margin-top: 3px;
}
.search {
  flex: 1;
  max-width: 400px;
}
.search-input {
  --el-input-bg-color: var(--paper-2);
  --el-input-border-color: var(--line);
  --el-input-hover-border-color: var(--line-strong);
  --el-input-focus-border-color: var(--teal);
  --el-input-text-color: var(--ink);
  --el-input-placeholder-color: #9aa5a2;
  --el-input-border-radius: 999px;
}
.search-input :deep(.el-input__wrapper) {
  border-radius: 999px;
  padding-left: 16px;
  box-shadow: 0 0 0 1px var(--line) inset;
  background: var(--paper-2);
}
.search-input :deep(.el-input__prefix) {
  color: var(--ink-soft);
  display: flex;
  align-items: center;
}
.ico {
  width: 17px;
  height: 17px;
}
.account {
  margin-left: auto;
}
.btn-login {
  border-radius: 999px;
  padding: 0 26px;
  background: var(--teal);
  color: var(--paper);
  border-color: var(--teal);
  font-weight: 600;
  letter-spacing: 0.12em;
  transition: background 0.25s ease, transform 0.25s ease;
}
.btn-login:hover {
  background: var(--teal-deep);
  border-color: var(--teal-deep);
  color: var(--paper);
  transform: translateY(-1px);
}
.user-chip {
  display: flex;
  align-items: center;
  gap: 10px;
}
.avatar {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  background: var(--teal);
  color: var(--paper);
  display: grid;
  place-items: center;
  font-family: var(--font-cjk-serif);
  font-size: 15px;
}
.uname {
  font-size: 14px;
  color: var(--ink);
}
.link-btn {
  border: 0;
  background: none;
  color: var(--ink-soft);
  font-size: 13px;
  cursor: pointer;
  text-decoration: underline;
  text-underline-offset: 3px;
}
.link-btn:hover {
  color: var(--clay);
}

/* ---------- Tabs ---------- */
.tabs {
  max-width: 1180px;
  margin: 0 auto;
  padding: 0 28px;
  display: flex;
  align-items: flex-end;
  gap: 6px;
  position: relative;
}
.tab {
  position: relative;
  border: 0;
  background: none;
  cursor: pointer;
  padding: 12px 10px 14px;
  margin-right: 26px;
  font-size: 15.5px;
  font-weight: 600;
  letter-spacing: 0.06em;
  color: var(--ink-soft);
  font-family: var(--font-cjk-serif);
  transition: color 0.25s ease;
}
.tab .idx {
  font-family: var(--font-mono);
  font-style: normal;
  font-size: 10px;
  color: var(--line-strong);
  margin-right: 7px;
  vertical-align: 3px;
  transition: color 0.25s ease;
}
.tab::after {
  content: '';
  position: absolute;
  left: 10px;
  right: 10px;
  bottom: 0;
  height: 2px;
  background: var(--teal);
  transform: scaleX(0);
  transform-origin: left center;
  transition: transform 0.32s cubic-bezier(0.65, 0, 0.35, 1);
}
.tab:hover {
  color: var(--ink);
}
.tab.is-active {
  color: var(--teal);
}
.tab.is-active .idx {
  color: var(--amber);
}
.tab.is-active::after {
  transform: scaleX(1);
}
.tabs-hint {
  margin-left: auto;
  font-size: 10px;
  letter-spacing: 0.2em;
  color: var(--line-strong);
  padding-bottom: 16px;
}

/* ---------- 主体 ---------- */
.stage {
  max-width: 1180px;
  margin: 0 auto;
  padding: 52px 28px 90px;
}
.panel-head {
  max-width: 760px;
  margin-bottom: 34px;
}
.eyebrow {
  display: inline-block;
  font-size: 10.5px;
  letter-spacing: 0.28em;
  color: var(--clay);
  border: 1px solid var(--line-strong);
  border-radius: 999px;
  padding: 5px 13px;
  background: var(--paper-2);
}
.panel-head h1 {
  font-family: var(--font-cjk-serif);
  font-weight: 700;
  font-size: clamp(30px, 4.6vw, 46px);
  line-height: 1.22;
  letter-spacing: 0.01em;
  margin: 18px 0 14px;
  color: var(--ink);
}
.lede {
  font-size: 15.5px;
  line-height: 1.85;
  color: var(--ink-soft);
  margin: 0;
}
.chips {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 22px;
}
.chips-label {
  font-size: 10px;
  letter-spacing: 0.2em;
  color: var(--line-strong);
}
.chip {
  border: 1px dashed var(--line-strong);
  background: var(--paper-2);
  color: var(--ink-soft);
  border-radius: 999px;
  padding: 7px 15px;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.22s ease;
}
.chip:hover {
  border-style: solid;
  border-color: var(--teal);
  color: var(--teal);
  transform: translateY(-1px);
}

/* ---------- 卡片通用 ---------- */
.card {
  position: relative;
  background: var(--paper-2);
  border: 1px solid var(--line);
  border-radius: 16px;
  box-shadow: 0 1px 0 rgba(255, 255, 255, 0.7) inset, 0 18px 40px -30px rgba(28, 42, 40, 0.5);
}
.card-tag {
  position: absolute;
  top: -9px;
  left: 26px;
  font-size: 9.5px;
  letter-spacing: 0.24em;
  color: var(--paper);
  background: var(--teal);
  padding: 3px 11px;
  border-radius: 4px;
}

/* ---------- 表单 ---------- */
.form-card {
  padding: 34px 30px 26px;
  background:
    linear-gradient(180deg, rgba(14, 79, 71, 0.03), transparent 40%),
    var(--paper-2);
}
.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 0.56fr;
  gap: 20px;
}
.field label {
  display: block;
  font-family: var(--font-cjk-serif);
  font-size: 13.5px;
  font-weight: 700;
  letter-spacing: 0.08em;
  color: var(--ink);
  margin-bottom: 9px;
}
.field :deep(.el-input__wrapper) {
  border-radius: 10px;
  background: #fff;
  box-shadow: 0 0 0 1px var(--line) inset;
  padding-left: 15px;
  transition: box-shadow 0.2s ease;
}
.field :deep(.el-input__wrapper:hover) {
  box-shadow: 0 0 0 1px var(--line-strong) inset;
}
.field :deep(.el-input__wrapper.is-focus) {
  box-shadow: 0 0 0 1.5px var(--teal) inset;
}
.unit {
  color: var(--ink-soft);
  font-size: 13px;
  padding-right: 4px;
}
.form-actions {
  display: flex;
  align-items: center;
  gap: 18px;
  margin-top: 26px;
  padding-top: 22px;
  border-top: 1px dashed var(--line-strong);
  flex-wrap: wrap;
}
.btn-generate {
  --el-button-bg-color: var(--teal);
  --el-button-border-color: var(--teal);
  --el-button-text-color: var(--paper);
  --el-button-hover-bg-color: var(--teal-deep);
  --el-button-hover-border-color: var(--teal-deep);
  --el-button-hover-text-color: var(--paper);
  --el-button-disabled-bg-color: #7ea39d;
  --el-button-disabled-border-color: #7ea39d;
  --el-button-disabled-text-color: #f0f4f3;
  border-radius: 999px;
  padding: 0 30px;
  height: 48px;
  font-size: 15.5px;
  font-weight: 600;
  letter-spacing: 0.1em;
  display: inline-flex;
  align-items: center;
  gap: 9px;
  box-shadow: 0 14px 26px -16px rgba(14, 79, 71, 0.8);
  transition: transform 0.22s ease, box-shadow 0.22s ease, letter-spacing 0.3s ease;
}
.btn-generate:hover:not(.is-disabled) {
  transform: translateY(-2px);
  letter-spacing: 0.16em;
  box-shadow: 0 20px 32px -18px rgba(14, 79, 71, 0.9);
}
.hint {
  margin: 0;
  font-size: 12.5px;
  color: var(--ink-soft);
}

/* ---------- 结果卡片 ---------- */
.result-card {
  margin-top: 30px;
  overflow: hidden;
}
.result-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 14px;
  flex-wrap: wrap;
  padding: 18px 28px;
  border-bottom: 1px solid var(--line);
  background: linear-gradient(90deg, rgba(14, 79, 71, 0.05), transparent 65%);
}
.result-title {
  display: flex;
  align-items: center;
  gap: 10px;
  font-family: var(--font-cjk-serif);
  font-weight: 700;
  font-size: 16.5px;
  letter-spacing: 0.06em;
  color: var(--ink);
}
.pulse {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--line-strong);
}
.pulse.on {
  background: var(--amber);
  animation: blink 1.1s ease-in-out infinite;
}
@keyframes blink {
  0%,
  100% {
    opacity: 1;
    box-shadow: 0 0 0 0 rgba(217, 142, 74, 0.55);
  }
  50% {
    opacity: 0.55;
    box-shadow: 0 0 0 7px rgba(217, 142, 74, 0);
  }
}
.badge-demo {
  font-size: 9.5px;
  letter-spacing: 0.16em;
  color: var(--clay);
  border: 1px solid rgba(196, 98, 61, 0.4);
  border-radius: 4px;
  padding: 2px 8px;
}
.result-meta {
  display: flex;
  align-items: center;
  gap: 9px;
  flex-wrap: wrap;
}
.meta-chip {
  font-size: 10.5px;
  letter-spacing: 0.12em;
  color: var(--ink-soft);
  border: 1px solid var(--line);
  background: #fff;
  border-radius: 999px;
  padding: 4px 11px;
}
.ghost-btn {
  border: 1px solid var(--line-strong);
  background: transparent;
  color: var(--ink-soft);
  border-radius: 999px;
  font-size: 12.5px;
  padding: 5px 14px;
  cursor: pointer;
  transition: all 0.2s ease;
}
.ghost-btn:hover {
  border-color: var(--teal);
  color: var(--teal);
}
.result-body {
  padding: 30px 32px 36px;
  min-height: 260px;
}

/* 空状态 */
.empty {
  text-align: center;
  padding: 26px 0 12px;
}
.empty-art {
  width: min(320px, 80%);
  height: auto;
  opacity: 0.9;
}
.empty h3 {
  font-family: var(--font-cjk-serif);
  font-size: 19px;
  margin: 16px 0 8px;
  color: var(--ink);
}
.empty p {
  margin: 0;
  font-size: 14px;
  color: var(--ink-soft);
}

/* 加载 */
.sk-wrap {
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 6px 0 10px;
}
.thinking {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 18px;
  font-size: 12px;
  letter-spacing: 0.1em;
  color: var(--clay);
}
.spin {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: 1.6px solid rgba(196, 98, 61, 0.28);
  border-top-color: var(--clay);
  animation: rot 0.8s linear infinite;
}
@keyframes rot {
  to {
    transform: rotate(360deg);
  }
}

/* Markdown 排版 */
.md {
  font-size: 15px;
  line-height: 1.95;
  color: var(--ink);
}
.md :deep(h2),
.md h2 {
  font-family: var(--font-cjk-serif);
  font-size: 23px;
  color: var(--teal);
  margin: 6px 0 16px;
  padding-bottom: 12px;
  border-bottom: 2px solid var(--line);
  letter-spacing: 0.04em;
}
.md h3,
.md :deep(h3) {
  position: relative;
  font-family: var(--font-cjk-serif);
  font-size: 17.5px;
  color: var(--ink);
  margin: 30px 0 12px;
  padding-left: 14px;
}
.md h3::before,
.md :deep(h3)::before {
  content: '';
  position: absolute;
  left: 0;
  top: 5px;
  bottom: 5px;
  width: 3px;
  border-radius: 2px;
  background: var(--amber);
}
.md h4,
.md :deep(h4) {
  font-family: var(--font-cjk-serif);
  font-size: 15.5px;
  color: var(--teal);
  margin: 22px 0 8px;
}
.md p,
.md :deep(p) {
  margin: 10px 0;
}
.md ul,
.md ol,
.md :deep(ul),
.md :deep(ol) {
  margin: 10px 0;
  padding-left: 22px;
}
.md li,
.md :deep(li) {
  margin: 6px 0;
}
.md ul li::marker,
.md :deep(ul li)::marker {
  color: var(--clay);
}
.md ol li::marker,
.md :deep(ol li)::marker {
  color: var(--teal);
  font-family: var(--font-mono);
}
.md strong,
.md :deep(strong) {
  color: var(--teal-deep);
  font-weight: 700;
}
.md blockquote,
.md :deep(blockquote) {
  margin: 16px 0;
  padding: 12px 18px;
  background: rgba(14, 79, 71, 0.05);
  border-left: 3px solid var(--teal);
  border-radius: 0 8px 8px 0;
  color: var(--ink-soft);
  font-size: 14px;
}
.md code,
.md :deep(code) {
  font-family: var(--font-mono);
  font-size: 13px;
  background: rgba(196, 98, 61, 0.1);
  color: var(--clay);
  padding: 2px 6px;
  border-radius: 4px;
}
.md hr,
.md :deep(hr) {
  border: 0;
  border-top: 1px dashed var(--line-strong);
  margin: 28px 0;
}

/* ---------- 推荐 / 设置占位 ---------- */
.rec-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 22px;
}
.rec-card {
  border: 1px solid var(--line);
  border-radius: 14px;
  overflow: hidden;
  background: var(--paper-2);
  transition: transform 0.28s ease, box-shadow 0.28s ease;
}
.rec-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 24px 40px -30px rgba(28, 42, 40, 0.65);
}
.rec-cover {
  height: 130px;
}
.tone-a {
  background: linear-gradient(135deg, #0e4f47, #4d8079 55%, #d98e4a);
}
.tone-b {
  background: linear-gradient(135deg, #c4623d, #d98e4a 55%, #0e4f47);
}
.tone-c {
  background: linear-gradient(135deg, #0a3a34, #c4623d 60%, #e6c07b);
}
.rec-info {
  padding: 18px 20px 22px;
}
.rec-info h3 {
  font-family: var(--font-cjk-serif);
  margin: 0 0 8px;
  font-size: 17px;
}
.rec-info p {
  margin: 0 0 12px;
  font-size: 13.5px;
  line-height: 1.75;
  color: var(--ink-soft);
}
.rec-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 7px;
}
.rec-tags span {
  font-size: 11px;
  color: var(--teal);
  border: 1px solid rgba(14, 79, 71, 0.25);
  border-radius: 999px;
  padding: 3px 10px;
}
.settings-card {
  padding: 8px 28px;
  max-width: 760px;
}
.setting-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  padding: 20px 0;
  border-bottom: 1px dashed var(--line);
}
.setting-row:last-child {
  border-bottom: 0;
}
.setting-row h4 {
  margin: 0 0 5px;
  font-family: var(--font-cjk-serif);
  font-size: 15.5px;
}
.setting-row p {
  margin: 0;
  font-size: 13px;
  color: var(--ink-soft);
}

/* ---------- 页脚 / 弹窗 ---------- */
.foot {
  max-width: 1180px;
  margin: 0 auto;
  padding: 20px 28px 46px;
  display: flex;
  justify-content: space-between;
  gap: 14px;
  flex-wrap: wrap;
  font-size: 10px;
  letter-spacing: 0.2em;
  color: var(--line-strong);
  border-top: 1px solid var(--line);
}
.login-box {
  display: flex;
  flex-direction: column;
  gap: 13px;
  padding: 8px 6px 14px;
}
.login-box h3 {
  font-family: var(--font-cjk-serif);
  margin: 4px 0 0;
  font-size: 21px;
}
.login-box p {
  margin: 0 0 6px;
  font-size: 13px;
  color: var(--ink-soft);
}
.login-box :deep(.el-input__wrapper) {
  border-radius: 10px;
}
.login-box .btn-generate {
  width: 100%;
  justify-content: center;
  margin-top: 4px;
}

/* ---------- 响应式 ---------- */
@media (max-width: 860px) {
  .topbar-inner {
    flex-wrap: wrap;
    gap: 14px;
  }
  .search {
    order: 3;
    max-width: none;
    flex-basis: 100%;
  }
  .form-grid {
    grid-template-columns: 1fr;
  }
  .stage {
    padding: 36px 18px 70px;
  }
  .result-body {
    padding: 22px 20px 28px;
  }
  .tabs-hint {
    display: none;
  }
}
</style>