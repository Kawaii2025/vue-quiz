<template>
  <div class="container">
    <h1>Javaコア技術 60問インタラクティブ試験（2025年版）</h1>
    <p class="subtitle">点击「答案を表示」查看答案和解説</p>

    <div class="filter-bar">
      <button class="filter-btn" :class="{ active: filter === 'all' }" @click="setFilter('all')">所有题目</button>
      <button class="filter-btn" :class="{ active: filter === 'wrong' }" @click="setFilter('wrong')">做错的题</button>
      <button class="filter-btn" :class="{ active: filter === 'correct' }" @click="setFilter('correct')">做对的题</button>
    </div>

    <div v-if="questions.length === 0">题目加载中...</div>

    <div v-for="(q, index) in filteredQuestions" :key="index" class="question">
      <div class="q-title">
        <span v-if="q.isMulti" class="multi">【多选】</span>
        {{ index + 1 }}. {{ q.title }}
      </div>

      <div class="code" v-if="q.code">{{ q.code }}</div>

      <div class="options">
        <label v-for="opt in q.options" :key="opt.key">
          <input
            :type="q.isMulti ? 'checkbox' : 'radio'"
            :name="'q' + index"
            :value="opt.key"
            v-model="userAnswers[index]"
          />
          {{ opt.key }}. {{ opt.text }}
        </label>
      </div>

      <button class="show-btn" @click="checkAnswer(index)">答案を表示</button>

      <div class="answer" v-if="answerVisible[index]">
        <div v-if="isCorrect(index)" class="correct">
          正确！<br />
          <strong>答案：{{ correctAnswerString(index) }}</strong>
        </div>

        <div v-else class="wrong">
          错误<br />
          <strong>答案：{{ correctAnswerString(index) }}</strong>
        </div>

        <div class="explanation">{{ q.explanation }}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      questions: [],
      filter: "all",
      userAnswers: {},
      answerVisible: {},
      status: {}
    };
  },
  computed: {
    filteredQuestions() {
      return this.questions.filter((q, i) => {
        const s = this.status[i];
        if (this.filter === "all") return true;
        if (this.filter === "correct") return s === "correct";
        if (this.filter === "wrong") return s === "wrong";
        return true;
      });
    }
  },
  methods: {
    async loadQuestions() {
      const res = await fetch("/questions.json");
      this.questions = await res.json();
    },
    setFilter(f) {
      this.filter = f;
    },
    correctAnswerString(i) {
      const c = this.questions[i].correct;
      return Array.isArray(c) ? c.join(", ") : c;
    },
    isCorrect(i) {
      return this.status[i] === "correct";
    },
    checkAnswer(i) {
      const q = this.questions[i];
      const user = this.userAnswers[i];
      let correct = false;

      if (q.isMulti) {
        const sel = Array.isArray(user) ? [...user].sort() : [];
        const ans = [...q.correct].sort();
        correct = JSON.stringify(sel) === JSON.stringify(ans);
      } else {
        correct = user === q.correct;
      }

      this.status[i] = correct ? "correct" : "wrong";
      this.$set(this.answerVisible, i, true);
    }
  },
  mounted() {
    this.loadQuestions();
  }
};
</script>

<style scoped>
.container { max-width: 900px; margin: auto; padding: 20px; }
.question { background: white; padding: 20px; margin-top: 20px; border-radius: 10px; }
.q-title { font-weight: bold; font-size: 18px; margin-bottom: 10px; color: #1a73e8; }
.code { background: #eee; padding: 10px; border-radius: 6px; white-space: pre-wrap; }
.correct { background: #e8f5e9; padding: 10px; border-left: 5px solid #4caf50; }
.wrong { background: #ffebee; padding: 10px; border-left: 5px solid #f44336; }
.multi { color: #d81b60; font-weight: bold; }
.filter-bar { text-align: center; margin: 20px 0; }
.filter-btn { padding: 10px 20px; border-radius: 6px; border: 2px solid #1a73e8; margin: 0 5px; cursor: pointer; }
.filter-btn.active { background: #1a73e8; color: white; }
.show-btn { background: #1a73e8; color: white; padding: 10px 20px; border-radius: 6px; cursor: pointer; }
</style>
