<template>
  <div class="team-forum">
    <!-- ============ 顶部英雄区 ============ -->
    <header class="forum-hero">
      <div class="hero-copy">
        <p class="hero-eyebrow">TRAVEL COMPANIONS · 组队广场</p>
        <h1 class="hero-title">找到顺路的旅伴</h1>
        <p class="hero-sub">一个人看风景，两个人记旅程。发一条组队帖，把独行变成同行。</p>
      </div>
      <el-button class="hero-cta" type="primary" size="large" :icon="Plus" @click="openDialog">
        发起组队
      </el-button>
    </header>

    <!-- ============ 统计条 ============ -->
    <section class="stats-strip">
      <div class="stat">
        <span class="stat-num">{{ posts.length }}</span>
        <span class="stat-label">条组队帖</span>
      </div>
      <div class="stat-divider"></div>
      <div class="stat">
        <span class="stat-num">{{ totalSlots }}</span>
        <span class="stat-label">个待募席位</span>
      </div>
      <div class="stat-divider"></div>
      <div class="search-box">
        <el-input
          v-model="keyword"
          placeholder="搜目的地 / 帖子详情…"
          :prefix-icon="Search"
          clearable
          size="default"
        />
      </div>
    </section>

    <!-- ============ 帖子列表 ============ -->
    <section class="post-grid" v-if="filteredPosts.length">
      <article
        v-for="(post, index) in filteredPosts"
        :key="post.id"
        class="post-card"
        :style="{ animationDelay: `${Math.min(index, 7) * 0.06}s` }"
      >
        <div class="card-cover" :style="coverStyle(index)">
          <span class="cover-destination">{{ post.destination }}</span>
          <span class="cover-date">
            <el-icon><Calendar /></el-icon>
            {{ post.departureTime }} 出发
          </span>
        </div>

        <div class="card-body">
          <div class="meta-row">
            <span class="meta-chip slots">
              <el-icon><User /></el-icon>
              招募 {{ post.slots }} 人
            </span>
            <span class="meta-chip contact">
              <el-icon><ChatDotRound /></el-icon>
              {{ post.contact }}
            </span>
          </div>

          <p class="post-detail">{{ post.detail }}</p>

          <div class="card-foot">
            <time class="publish-time">{{ timeAgo(post.createdAt) }}发布</time>
            <!-- 真实项目中在这里加 v-if="post.authorId === currentUserId" 即可限制只能删自己的 -->
            <el-button
              type="danger"
              plain
              size="small"
              :icon="Delete"
              @click="removePost(post)"
            >
              删除
            </el-button>
          </div>
        </div>
      </article>
    </section>

    <!-- ============ 空状态 ============ -->
    <el-empty
      v-else
      class="forum-empty"
      :description="keyword ? '没有匹配的组队帖，换个关键词试试' : '还没有组队帖，来发第一条吧'"
    >
      <el-button type="primary" :icon="Plus" @click="openDialog">发起组队</el-button>
    </el-empty>

    <!-- ============ 发帖弹窗 ============ -->
    <el-dialog
      v-model="showDialog"
      title="发起组队"
      width="560px"
      class="forum-dialog"
      destroy-on-close
      @closed="resetForm"
    >
      <el-form
        ref="formRef"
        :model="form"
        :rules="rules"
        label-position="top"
        class="team-form"
      >
        <el-form-item label="目的地" prop="destination">
          <el-input
            v-model="form.destination"
            placeholder="如：云南 · 大理洱海"
            maxlength="30"
            show-word-limit
          />
        </el-form-item>

        <div class="form-row">
          <el-form-item label="出发时间" prop="departureTime" class="form-row-item">
            <el-date-picker
              v-model="form.departureTime"
              type="date"
              placeholder="选择出发日期"
              value-format="YYYY-MM-DD"
              :disabled-date="disabledDate"
              style="width: 100%"
            />
          </el-form-item>

          <el-form-item label="招募人数" prop="slots" class="form-row-item">
            <el-input-number
              v-model="form.slots"
              :min="1"
              :max="99"
              style="width: 100%"
            />
          </el-form-item>
        </div>

        <el-form-item label="联系方式" prop="contact">
          <el-input
            v-model="form.contact"
            placeholder="微信号 / 手机号 / QQ 群号"
            maxlength="40"
          />
        </el-form-item>

        <el-form-item label="帖子详情" prop="detail">
          <el-input
            v-model="form.detail"
            type="textarea"
            :rows="4"
            placeholder="行程打算、费用分摊、期望旅伴…写得越细越容易找到同路人"
            maxlength="300"
            show-word-limit
          />
        </el-form-item>
      </el-form>

      <template #footer>
        <el-button @click="showDialog = false">取消</el-button>
        <el-button type="primary" :loading="submitting" @click="submitForm">
          发布组队
        </el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import {
  Plus,
  Search,
  User,
  Delete,
  Calendar,
  ChatDotRound
} from '@element-plus/icons-vue'

/* ---------------- 常量 ---------------- */
const STORAGE_KEY = 'travel-team-forum-posts-v1'

/* ---------------- 列表数据 ---------------- */
const posts = ref([])
const keyword = ref('')

const totalSlots = computed(() =>
  posts.value.reduce((sum, p) => sum + (Number(p.slots) || 0), 0)
)

const filteredPosts = computed(() => {
  const kw = keyword.value.trim().toLowerCase()
  if (!kw) return posts.value
  return posts.value.filter(
    (p) =>
      p.destination.toLowerCase().includes(kw) ||
      p.detail.toLowerCase().includes(kw)
  )
})

/* ---------------- 默认假数据 ---------------- */
const defaultPosts = () => {
  const day = 24 * 60 * 60 * 1000
  return [
    {
      id: 'seed-1',
      destination: '云南 · 大理洱海',
      departureTime: '2026-04-12',
      slots: 3,
      contact: '微信：dali_ride_2026',
      detail:
        '计划环洱海骑行三天，电动车+自行车混搭，预算 1500 左右含住宿。找 2-3 个不赶路、爱拍照的搭子，早晚一起看日出日落。',
      createdAt: Date.now() - 2 * 60 * 60 * 1000
    },
    {
      id: 'seed-2',
      destination: '川西 · 稻城亚丁',
      departureTime: '2026-05-01',
      slots: 4,
      contact: '电话：138-0000-0000',
      detail:
        '五一走长线，成都集合自驾出发，已有一车两人，再拼两辆车。高原徒步强度中等，希望同伴有基本体能、能接受轮流开车，费用 AA。',
      createdAt: Date.now() - 26 * 60 * 60 * 1000
    },
    {
      id: 'seed-3',
      destination: '新疆 · 伊犁杏花谷',
      departureTime: '2026-04-05',
      slots: 2,
      contact: 'QQ 群：123456789',
      detail:
        '花期就那一两周，想包车走杏花沟 + 赛里木湖，四人成行刚好摊车费。摄影爱好者优先，行程宽松，早出晚归拍光影也行。',
      createdAt: Date.now() - 3 * 24 * 60 * 60 * 1000
    }
  ]
}

/* ---------------- localStorage 持久化 ---------------- */
function loadPosts() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    if (!raw) {
      posts.value = defaultPosts()
      savePosts()
      return
    }
    const parsed = JSON.parse(raw)
    posts.value = Array.isArray(parsed)
      ? parsed.map((p) => ({ ...p, slots: Number(p.slots) || 1 }))
      : []
    if (!posts.value.length) {
      posts.value = defaultPosts()
      savePosts()
    }
  } catch (err) {
    console.warn('[TeamForum] 读取本地数据失败，已回退为默认数据', err)
    posts.value = defaultPosts()
  }
}

function savePosts() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(posts.value))
  } catch (err) {
    console.warn('[TeamForum] 写入 localStorage 失败', err)
  }
}

/* ---------------- 表单 ---------------- */
const showDialog = ref(false)
const submitting = ref(false)
const formRef = ref(null)

const createEmptyForm = () => ({
  destination: '',
  departureTime: '',
  slots: 2,
  contact: '',
  detail: ''
})
const form = ref(createEmptyForm())

const rules = {
  destination: [
    { required: true, message: '请填写目的地', trigger: 'blur' }
  ],
  departureTime: [
    { required: true, message: '请选择出发时间', trigger: 'change' }
  ],
  slots: [
    {
      validator: (_rule, value, callback) => {
        if (!value || value < 1) callback(new Error('招募人数至少 1 人'))
        else callback()
      },
      trigger: 'change'
    }
  ],
  contact: [
    { required: true, message: '请留下联系方式', trigger: 'blur' }
  ],
  detail: [
    { required: true, message: '请描述一下行程详情', trigger: 'blur' },
    { min: 10, message: '详情至少写 10 个字，方便旅伴了解你', trigger: 'blur' }
  ]
}

function disabledDate(date) {
  const startOfToday = new Date()
  startOfToday.setHours(0, 0, 0, 0)
  return date.getTime() < startOfToday.getTime()
}

function openDialog() {
  form.value = createEmptyForm()
  showDialog.value = true
}

function resetForm() {
  formRef.value?.clearValidate()
  form.value = createEmptyForm()
}

async function submitForm() {
  if (!formRef.value) return
  try {
    await formRef.value.validate()
  } catch {
    return
  }

  submitting.value = true
  try {
    posts.value.unshift({
      id: `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`,
      destination: form.value.destination.trim(),
      departureTime: form.value.departureTime,
      slots: form.value.slots,
      contact: form.value.contact.trim(),
      detail: form.value.detail.trim(),
      createdAt: Date.now()
    })
    savePosts()
    showDialog.value = false
    ElMessage.success('组队帖发布成功，祝你早日找到旅伴！')
  } finally {
    submitting.value = false
  }
}

/* ---------------- 删除 ---------------- */
async function removePost(post) {
  try {
    await ElMessageBox.confirm(
      `确定删除「${post.destination}」这条组队帖吗？删除后不可恢复。`,
      '删除确认',
      {
        type: 'warning',
        confirmButtonText: '删除',
        cancelButtonText: '再想想',
        confirmButtonClass: 'el-button--danger'
      }
    )
  } catch {
    return
  }

  posts.value = posts.value.filter((p) => p.id !== post.id)
  savePosts()
  ElMessage.success('已删除')
}

/* ---------------- 工具 ---------------- */
const coverPalettes = [
  'linear-gradient(135deg, #0E6E64 0%, #178F7E 55%, #E4572E 130%)',
  'linear-gradient(135deg, #1D3557 0%, #457B9D 60%, #E9C46A 130%)',
  'linear-gradient(135deg, #6A4C93 0%, #B5179E 55%, #F15BB5 130%)',
  'linear-gradient(135deg, #264653 0%, #2A9D8F 55%, #E76F51 130%)',
  'linear-gradient(135deg, #7F4F24 0%, #936639 55%, #A68A64 130%)'
]

function coverStyle(index) {
  return { background: coverPalettes[index % coverPalettes.length] }
}

function timeAgo(ts) {
  const diff = Date.now() - (ts || Date.now())
  const minutes = Math.floor(diff / 60000)
  if (minutes < 1) return '刚刚'
  if (minutes < 60) return `${minutes} 分钟前`
  const hours = Math.floor(minutes / 60)
  if (hours < 24) return `${hours} 小时前`
  const days = Math.floor(hours / 24)
  if (days < 30) return `${days} 天前`
  return new Date(ts).toLocaleDateString('zh-CN')
}

/* ---------------- 初始化 ---------------- */
onMounted(loadPosts)
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@600;700&display=swap');

.team-forum {
  /* Element Plus 主题变量（作用域内覆盖） */
  --el-color-primary: #0e6e64;
  --el-color-primary-light-3: #4d978f;
  --el-color-primary-light-5: #7ab3ac;
  --el-color-primary-light-7: #a6c9c4;
  --el-color-primary-light-8: #bccfcc;
  --el-color-primary-light-9: #e5efed;
  --el-color-primary-dark-2: #0a544c;

  --ink: #14312f;
  --muted: #86806f;
  --paper: #f7f2e8;
  --line: #e5ddcd;
  --accent: #e4572e;

  min-height: 60vh;
  padding: 40px clamp(16px, 4vw, 56px) 72px;
  background:
    radial-gradient(ellipse at 8% 0%, rgba(14, 110, 100, 0.1) 0%, transparent 55%),
    radial-gradient(ellipse at 95% 10%, rgba(228, 87, 46, 0.08) 0%, transparent 45%),
    var(--paper);
  font-family: 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
  color: var(--ink);
  position: relative;
}

/* 细噪点纹理，让底色不那么"平" */
.team-forum::before {
  content: '';
  position: absolute;
  inset: 0;
  pointer-events: none;
  opacity: 0.035;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='140' height='140'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='3'/%3E%3C/filter%3E%3Crect width='140' height='140' filter='url(%23n)'/%3E%3C/svg%3E");
}

/* ---------------- 英雄区 ---------------- */
.forum-hero {
  position: relative;
  display: flex;
  flex-wrap: wrap;
  align-items: flex-end;
  justify-content: space-between;
  gap: 24px;
  padding-bottom: 32px;
  border-bottom: 1px solid var(--line);
}

.hero-eyebrow {
  margin: 0 0 10px;
  font-size: 12px;
  letter-spacing: 0.28em;
  color: var(--accent);
  font-weight: 600;
}

.hero-title {
  margin: 0;
  font-family: 'Noto Serif SC', 'Songti SC', serif;
  font-size: clamp(32px, 4.6vw, 50px);
  font-weight: 700;
  line-height: 1.15;
  letter-spacing: 0.02em;
}

.hero-sub {
  margin: 12px 0 0;
  max-width: 460px;
  font-size: 14px;
  line-height: 1.8;
  color: var(--muted);
}

.hero-cta {
  padding: 16px 32px;
  font-size: 15px;
  letter-spacing: 0.14em;
  border-radius: 2px;
  transition: letter-spacing 0.3s ease, transform 0.2s ease;
}

.hero-cta:hover {
  letter-spacing: 0.24em;
  transform: translateY(-2px);
}

/* ---------------- 统计条 ---------------- */
.stats-strip {
  position: relative;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 28px;
  padding: 22px 0 8px;
}

.stat {
  display: flex;
  align-items: baseline;
  gap: 8px;
}

.stat-num {
  font-family: 'Noto Serif SC', serif;
  font-size: 30px;
  font-weight: 700;
  color: var(--ink);
}

.stat-label {
  font-size: 13px;
  color: var(--muted);
}

.stat-divider {
  width: 1px;
  height: 24px;
  background: var(--line);
}

.search-box {
  margin-left: auto;
  width: min(280px, 100%);
}

/* ---------------- 卡片网格 ---------------- */
.post-grid {
  position: relative;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 26px;
  margin-top: 30px;
}

.post-card {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: #fffdf8;
  border: 1px solid var(--line);
  border-radius: 4px;
  opacity: 0;
  animation: cardIn 0.55s cubic-bezier(0.22, 1, 0.36, 1) forwards;
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}

.post-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 18px 40px -18px rgba(20, 49, 47, 0.35);
}

@keyframes cardIn {
  from {
    opacity: 0;
    transform: translateY(22px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card-cover {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  gap: 8px;
  min-height: 132px;
  padding: 20px;
  color: #fff;
}

.cover-destination {
  font-family: 'Noto Serif SC', 'Songti SC', serif;
  font-size: 26px;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-shadow: 0 2px 12px rgba(0, 0, 0, 0.25);
}

.cover-date {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  font-size: 12px;
  letter-spacing: 0.08em;
  opacity: 0.92;
}

.card-body {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: 14px;
  padding: 20px;
}

.meta-row {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.meta-chip {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 4px 10px;
  font-size: 12px;
  border-radius: 999px;
}

.meta-chip.slots {
  color: var(--accent);
  background: rgba(228, 87, 46, 0.09);
  border: 1px solid rgba(228, 87, 46, 0.25);
}

.meta-chip.contact {
  color: var(--ink);
  background: rgba(14, 110, 100, 0.08);
  border: 1px solid rgba(14, 110, 100, 0.22);
}

.post-detail {
  flex: 1;
  margin: 0;
  font-size: 13.5px;
  line-height: 1.85;
  color: #4c5a52;
  display: -webkit-box;
  -webkit-line-clamp: 4;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card-foot {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 14px;
  border-top: 1px dashed var(--line);
}

.publish-time {
  font-size: 12px;
  color: var(--muted);
  letter-spacing: 0.04em;
}

.forum-empty {
  position: relative;
  padding: 60px 0;
}

/* ---------------- 弹窗表单 ---------------- */
.forum-dialog {
  border-radius: 6px;
}

.team-form {
  padding: 4px 8px 0;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.form-row-item {
  margin-bottom: 18px;
}

@media (max-width: 560px) {
  .form-row {
    grid-template-columns: 1fr;
    gap: 0;
  }

  .search-box {
    margin-left: 0;
    width: 100%;
  }

  .stat-divider {
    display: none;
  }
}

/* 弹窗内部组件适配 */
:deep(.forum-dialog .el-dialog__header) {
  padding-bottom: 12px;
  border-bottom: 1px solid var(--line);
}

:deep(.forum-dialog .el-dialog__title) {
  font-family: 'Noto Serif SC', serif;
  font-weight: 700;
  letter-spacing: 0.06em;
}

:deep(.team-form .el-form-item__label) {
  font-size: 13px;
  font-weight: 600;
  letter-spacing: 0.06em;
  color: var(--ink);
}

:deep(.el-input__wrapper),
:deep(.el-textarea__inner) {
  border-radius: 3px;
}
</style>