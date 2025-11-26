<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 py-8 px-4" v-if="questions.length > 0">
    <div class="max-w-4xl mx-auto">
      <h1 class="text-4xl font-bold text-center text-blue-600 mb-2">Vue3 コア技術 {{ questions.length }}問インタラクティブ試験（2025年版）</h1>
      <p class="text-center text-gray-600 mb-8">単選：クリック即出答案 | 複数選択：「答案を表示」ボタンをクリック</p>

      <div class="flex justify-center gap-3 mb-8 flex-wrap">
        <button 
          :class="filter === 'all' ? 'bg-blue-600 text-white shadow-lg' : 'bg-white text-blue-600 border-2 border-blue-600 hover:bg-blue-50'"
          class="px-6 py-2 rounded-lg font-medium transition-all duration-300"
          @click="filter = 'all'"
        >
          すべて表示
        </button>
        <button 
          :class="filter === 'wrong' ? 'bg-blue-600 text-white shadow-lg' : 'bg-white text-blue-600 border-2 border-blue-600 hover:bg-blue-50'"
          class="px-6 py-2 rounded-lg font-medium transition-all duration-300"
          @click="filter = 'wrong'"
        >
          間違えた問題のみ
        </button>
        <button 
          :class="filter === 'correct' ? 'bg-blue-600 text-white shadow-lg' : 'bg-white text-blue-600 border-2 border-blue-600 hover:bg-blue-50'"
          class="px-6 py-2 rounded-lg font-medium transition-all duration-300"
          @click="filter = 'correct'"
        >
          正解した問題のみ
        </button>
      </div>

      <div v-for="(q, index) in filteredQuestions" :key="index" class="bg-white rounded-xl shadow-md p-6 mb-6 hover:shadow-lg transition-shadow duration-300">
        <div class="text-lg font-bold text-blue-600 mb-4">
          <span v-if="q.isMulti" class="inline-block bg-pink-100 text-pink-700 px-3 py-1 rounded-full text-sm font-semibold mr-2">【複数選択】</span>
          <span v-else class="inline-block bg-blue-100 text-blue-700 px-3 py-1 rounded-full text-sm font-semibold mr-2">【単選】</span>
          {{ getOriginalIndex(index) + 1 }}. {{ q.title }}
        </div>

        <div v-if="q.code" class="bg-gray-900 text-gray-100 p-4 rounded-lg font-mono text-sm mb-4 overflow-x-auto whitespace-pre-wrap break-words">
          {{ q.code }}
        </div>

        <div class="space-y-3 mb-6">
          <label v-for="opt in q.options" :key="opt.key" class="flex items-center p-3 rounded-lg hover:bg-gray-50 transition-colors duration-200" :class="{ 'cursor-default hover:bg-white': answerVisible[index] }">
            <input
              :type="q.isMulti ? 'checkbox' : 'radio'"
              :name="'q' + index"
              :value="opt.key"
              class="w-5 h-5 text-blue-600 rounded cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed"
              :checked="isChecked(index, opt.key, q.isMulti)"
              :disabled="answerVisible[index]"
              @change="handleInputChange(index, opt.key, q.isMulti, $event, q)"
            />
            <span class="ml-3 text-gray-700">{{ opt.key }}. {{ opt.text }}</span>
          </label>
        </div>

        <button 
          v-if="q.isMulti && !answerVisible[index]"
          class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 px-6 rounded-lg transition-colors duration-200 mb-4"
          @click="handleCheckAnswer(index, q.isMulti)"
        >
          答案を表示
        </button>

        <div v-if="answerVisible[index]" class="animate-fadeIn">
          <div v-if="status[index] === 'correct'" class="bg-green-50 border-l-4 border-green-500 p-4 rounded mb-3">
            <p class="text-green-700 font-semibold">✓ {{ q.isMulti ? '全正解！' : '正解！' }}</p>
            <p class="text-green-700 font-bold mt-2">正解：{{ getCorrectAnswerString(q) }}</p>
          </div>

          <div v-else class="bg-red-50 border-l-4 border-red-500 p-4 rounded mb-3">
            <p class="text-red-700 font-semibold">✗ 不正解</p>
            <p class="text-red-700 font-bold mt-2">正解：{{ getCorrectAnswerString(q) }}</p>
            <div v-if="userSelectedStr[index]" class="text-red-700 text-sm mt-2">
              あなたの選択：{{ userSelectedStr[index] }}
            </div>
          </div>

          <div class="bg-gray-100 p-4 rounded text-gray-800 text-sm leading-relaxed">
            <strong>解説：</strong> {{ q.explanation }}
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 py-8 px-4 flex items-center justify-center" v-else>
    <div class="text-center">
      <p class="text-2xl text-gray-600">読み込み中...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const questions = ref([])
const filter = ref('all')
const userAnswers = ref({})
const answerVisible = ref({})
const userSelectedStr = ref({})
const status = ref({})

const loadQuestions = async () => {
  try {
    const res = await fetch('/questions.json')
    questions.value = await res.json()
    questions.value.forEach((q, idx) => {
      userAnswers.value[idx] = q.isMulti ? [] : ''
    })
  } catch (err) {
    console.error('Failed to load questions:', err)
  }
}

const filteredQuestions = computed(() => {
  return questions.value.filter((q, i) => {
    const s = status.value[i]
    if (filter.value === 'all') return true
    if (filter.value === 'correct') return s === 'correct'
    if (filter.value === 'wrong') return s === 'wrong'
    return true
  })
})

const getOriginalIndex = (filteredIndex) => {
  let count = 0
  for (let i = 0; i < questions.value.length; i++) {
    if (filter.value === 'all' || 
        (filter.value === 'correct' && status.value[i] === 'correct') ||
        (filter.value === 'wrong' && status.value[i] === 'wrong')) {
      if (count === filteredIndex) return i
      count++
    }
  }
  return filteredIndex
}

const getCorrectAnswerString = (q) => {
  const c = q.correct
  return Array.isArray(c) ? c.join(', ') : c
}

const isChecked = (questionIndex, optionKey, isMulti) => {
  const answer = userAnswers.value[questionIndex]
  if (isMulti) {
    return Array.isArray(answer) && answer.includes(optionKey)
  } else {
    return answer === optionKey
  }
}

const handleInputChange = (index, optionKey, isMulti, event, q) => {
  if (isMulti) {
    const currentAnswers = userAnswers.value[index] || []
    if (event.target.checked) {
      if (!currentAnswers.includes(optionKey)) {
        currentAnswers.push(optionKey)
      }
    } else {
      const idx = currentAnswers.indexOf(optionKey)
      if (idx > -1) {
        currentAnswers.splice(idx, 1)
      }
    }
    userAnswers.value[index] = [...currentAnswers]
  } else {
    // 単選：クリック即出答案
    userAnswers.value[index] = optionKey
    handleCheckAnswer(index, isMulti)
  }
}

const handleCheckAnswer = (index, isMulti) => {
  const q = questions.value[index]
  const user = userAnswers.value[index]
  let isCorrect = false

  if (isMulti) {
    const correctArray = Array.isArray(q.correct) ? q.correct : [q.correct]
    const userArray = Array.isArray(user) ? user : []
    
    const correctSorted = [...correctArray].sort()
    const userSorted = [...userArray].sort()
    
    isCorrect = JSON.stringify(correctSorted) === JSON.stringify(userSorted)
    userSelectedStr.value[index] = userArray.length > 0 ? userArray.join(', ') : 'なし'
  } else {
    if (!user) {
      userSelectedStr.value[index] = 'なし'
      isCorrect = false
    } else {
      isCorrect = user === q.correct
      userSelectedStr.value[index] = user
    }
  }

  status.value[index] = isCorrect ? 'correct' : 'wrong'
  answerVisible.value[index] = true
}

onMounted(() => {
  loadQuestions()
})
</script>

<style scoped>
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fadeIn {
  animation: fadeIn 0.5s ease-out;
}
</style>
