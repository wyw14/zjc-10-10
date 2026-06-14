<template>
  <div class="container">
    <div class="header">
      <h1>💡 每日问答</h1>
      <p>每天一个问题，激发深度思考，记录成长轨迹</p>
    </div>

    <div class="tabs">
      <button
        class="tab-btn"
        :class="{ active: currentTab === 'today' }"
        @click="currentTab = 'today'"
      >
        今日问题
      </button>
      <button
        class="tab-btn"
        :class="{ active: currentTab === 'history' }"
        @click="currentTab = 'history'; loadHistory()"
      >
        历史记录
      </button>
    </div>

    <div v-if="loading || !todayData" class="loading">加载中...</div>

    <TodayView
      v-else-if="currentTab === 'today'"
      :today-data="todayData"
      :submitting="submitting"
      @submit="submitAnswer"
    />

    <HistoryView
      v-else-if="currentTab === 'history'"
      :history="history"
      :loading="historyLoading"
      @prev-month="prevMonth"
      @next-month="nextMonth"
    />

    <div v-if="toast.show" class="toast" :class="{ error: toast.isError }">
      {{ toast.message }}
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, watch } from 'vue'
import TodayView from './components/TodayView.vue'
import HistoryView from './components/HistoryView.vue'

const currentTab = ref('today')
const loading = ref(false)
const submitting = ref(false)
const todayData = ref(null)
const history = ref(null)
const historyLoading = ref(false)

const toast = reactive({
  show: false,
  message: '',
  isError: false
})

function showToast(message, isError = false) {
  toast.message = message
  toast.isError = isError
  toast.show = true
  setTimeout(() => {
    toast.show = false
  }, 3000)
}

async function loadToday() {
  loading.value = true
  try {
    const res = await fetch('/api/today')
    const json = await res.json()
    if (json.success) {
      todayData.value = json.data
    } else {
      showToast('加载失败', true)
    }
  } catch (e) {
    showToast('网络错误', true)
  } finally {
    loading.value = false
  }
}

async function submitAnswer(answer) {
  submitting.value = true
  try {
    const res = await fetch('/api/answer', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ answer })
    })
    const json = await res.json()
    if (json.success) {
      showToast('回答已保存 ✓')
      await loadToday()
    } else {
      showToast(json.message || '保存失败', true)
    }
  } catch (e) {
    showToast('网络错误', true)
  } finally {
    submitting.value = false
  }
}

const viewYear = ref(new Date().getFullYear())
const viewMonth = ref(new Date().getMonth() + 1)

async function loadHistory() {
  historyLoading.value = true
  try {
    const res = await fetch(`/api/history?year=${viewYear.value}&month=${viewMonth.value}`)
    const json = await res.json()
    if (json.success) {
      history.value = json.data
    }
  } catch (e) {
    showToast('加载历史失败', true)
  } finally {
    historyLoading.value = false
  }
}

function prevMonth() {
  if (viewMonth.value === 1) {
    viewMonth.value = 12
    viewYear.value--
  } else {
    viewMonth.value--
  }
  loadHistory()
}

function nextMonth() {
  if (viewMonth.value === 12) {
    viewMonth.value = 1
    viewYear.value++
  } else {
    viewMonth.value++
  }
  loadHistory()
}

onMounted(() => {
  loadToday()
})
</script>
