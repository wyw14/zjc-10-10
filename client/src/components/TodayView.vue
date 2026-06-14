<template>
  <div class="card">
    <span class="date-label">📅 {{ formatDate(data.date) }}</span>

    <div v-if="data.answered" class="status-badge status-answered">
      ✓ 今日已回答
    </div>
    <div v-else class="status-badge status-unanswered">
      ⏰ 等待你的回答
    </div>

    <div class="question-text">
      {{ data.question.question }}
    </div>

    <label class="answer-label">✍️ 我的回答：</label>

    <textarea
      v-model="localAnswer"
      class="answer-textarea"
      placeholder="在这里写下你的思考和答案..."
      :disabled="submitting"
    ></textarea>

    <div class="mood-section">
      <label class="mood-label">😊 今天的心情：</label>
      <div class="mood-options">
        <button
          v-for="mood in moodOptions"
          :key="mood.id"
          class="mood-btn"
          :class="{ active: selectedMood === mood.id }"
          @click="selectedMood = mood.id"
          :disabled="submitting"
          :title="mood.label"
        >
          <span class="mood-emoji">{{ mood.emoji }}</span>
          <span class="mood-text">{{ mood.label }}</span>
        </button>
      </div>
    </div>

    <button
      class="submit-btn"
      :disabled="submitting || !localAnswer.trim()"
      @click="handleSubmit"
    >
      {{ submitting ? '保存中...' : '💾 保存回答' }}
    </button>

    <div v-if="data.answered && data.answer" class="displayed-answer">
      <h4>📝 已保存的回答：</h4>
      <p>{{ data.answer }}</p>
      <div v-if="data.mood" class="saved-mood">
        <span class="saved-mood-label">今日心情：</span>
        <span class="saved-mood-emoji">{{ getMoodEmoji(data.mood) }}</span>
        <span class="saved-mood-text">{{ getMoodLabel(data.mood) }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue'

const props = defineProps({
  todayData: Object,
  submitting: Boolean
})

const emit = defineEmits(['submit'])

const data = ref(props.todayData)
const localAnswer = ref('')
const selectedMood = ref(null)

const moodOptions = computed(() => {
  return data.value?.moodOptions || []
})

watch(() => props.todayData, (newVal) => {
  if (newVal) {
    data.value = newVal
    localAnswer.value = newVal.answer || ''
    selectedMood.value = newVal.mood || null
  }
}, { immediate: true })

function formatDate(dateStr) {
  if (!dateStr) return ''
  const d = new Date(dateStr)
  const weekdays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六']
  return `${d.getFullYear()}年${d.getMonth() + 1}月${d.getDate()}日 ${weekdays[d.getDay()]}`
}

function getMoodEmoji(moodId) {
  const mood = moodOptions.value.find(m => m.id === moodId)
  return mood ? mood.emoji : ''
}

function getMoodLabel(moodId) {
  const mood = moodOptions.value.find(m => m.id === moodId)
  return mood ? mood.label : ''
}

function handleSubmit() {
  emit('submit', {
    answer: localAnswer.value,
    mood: selectedMood.value
  })
}
</script>
