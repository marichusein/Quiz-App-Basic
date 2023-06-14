<template>
  <div class="container">
    <h1 class="text-center mt-4">Quiz App</h1>
    <div v-if="loading" class="text-center mt-5">
      <div class="spinner-border text-primary" role="status">
        <span class="sr-only">Loading...</span>
      </div>
    </div>
    <div v-else>
      <div v-if="!finished" class="card mt-5">
        <div class="card-body">
          <h2 class="card-title">{{ currentQuestion.question }}</h2>
          <div class="timer">{{ timeLeft }}s</div>
          <ul class="list-group">
            <li
              class="list-group-item list-group-item-action"
              v-for="option in currentQuestion.options"
              :key="option"
              :class="{
                'list-group-item-success': showFeedback && option === currentQuestion.answer,
                'list-group-item-danger': showFeedback && option === selectedAnswer,
                'list-group-item-selected': option === selectedAnswer
              }"
              @click="selectAnswer(option)"
            >
              {{ option }}
            </li>
          </ul>
          <div v-if="showFeedback" class="mt-4">
            <p v-if="isCorrect" class="text-success">Correct!</p>
            <p v-else class="text-danger">Incorrect! The correct answer is {{ currentQuestion.answer }}.</p>
            <button class="btn btn-primary" @click="nextQuestion">Next</button>
          </div>
          <div v-else class="mt-4">
            <button class="btn btn-primary" @click="checkAnswer">Submit</button>
          </div>
        </div>
      </div>
      <div v-else class="card-finished mt-5">
        <div class="card-body">
          <h2 class="card-title">Quiz finished! Your score: {{ score }}/{{ totalPoints }}</h2>
          <h3 class="card-title">Percentage: {{ calculatePercentage() }}%</h3>
          <button class="btn btn-primary" @click="restartQuiz">Restart</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      loading: true,
      questions: [],
      currentQuestionIndex: 0,
      currentQuestion: {},
      selectedAnswer: '',
      showFeedback: false,
      isCorrect: false,
      score: 0,
      totalPoints: 0,
      finished: false,
      timer: null,
      timeLeft: 0,
      timeLimit: 10 // seconds for each question
    };
  },
  mounted() {
    this.fetchQuestions();
  },
  methods: {
    fetchQuestions() {
      axios
        .get('https://opentdb.com/api.php?amount=10&category=18&type=multiple') 
        .then(response => {
          // Handle the API response and extract the questions
          const questions = response.data.results.map(question => {
            const options = [
              question.correct_answer,
              ...question.incorrect_answers
            ].sort(() => Math.random() - 0.5);

            let points = 0;
            if (question.difficulty === 'easy') {
              points = 2;
            } else if (question.difficulty === 'medium') {
              points = 2.5;
            } else if (question.difficulty === 'hard') {
              points = 3;
            }

            return {
              question: question.question,
              options: options,
              answer: question.correct_answer,
              points: points
            };
          });

          this.questions = questions;
          this.currentQuestion = this.questions[0];
          this.totalPoints = this.calculateTotalPoints();
          this.loading = false;
          this.startQuizTimer();
        })
        .catch(error => {
          console.error('Error fetching questions:', error);
        });
    },

    selectAnswer(answer) {
      this.selectedAnswer = answer;
    },

    checkAnswer() {
      this.showFeedback = true;
      this.isCorrect = this.selectedAnswer === this.currentQuestion.answer;
      if (this.isCorrect) {
        this.score += this.currentQuestion.points;
      }
      this.stopQuizTimer();
    },

    nextQuestion() {
      this.showFeedback = false;
      this.selectedAnswer = '';
      this.currentQuestionIndex++;
      if (this.currentQuestionIndex < this.questions.length) {
        this.currentQuestion = this.questions[this.currentQuestionIndex];
        this.startQuizTimer();
      } else {
        this.finished = true;
      }
    },

    restartQuiz() {
      this.loading = true;
      this.questions = [];
      this.currentQuestionIndex = 0;
      this.currentQuestion = {};
      this.selectedAnswer = '';
      this.showFeedback = false;
      this.isCorrect = false;
      this.score = 0;
      this.totalPoints = 0;
      this.finished = false;
      this.fetchQuestions();
    },

    startQuizTimer() {
      this.timeLeft = this.timeLimit;
      this.timer = setInterval(() => {
        if (this.timeLeft > 0) {
          this.timeLeft--;
        } else {
          this.checkAnswer();
        }
      }, 1000);
    },

    stopQuizTimer() {
      clearInterval(this.timer);
    },

    calculateTotalPoints() {
      return this.questions.reduce((total, question) => total + question.points, 0);
    },

    calculatePercentage() {
      return Math.round((this.score / this.totalPoints) * 100);
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 400px;
  margin: 0 auto;
}

.card {
  background-color: #fff;
  border: none;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  padding: 20px;
}

.card-body {
  text-align: center;
}

h1 {
  font-size: 28px;
  color: #333;
}

h2 {
  font-size: 22px;
  color: #333;
  margin-top: 20px;
}

h3 {
  font-size: 18px;
  color: #333;
  margin-top: 10px;
}

.btn {
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.btn:hover {
  background-color: #0069d9;
}

.list-group-item {
  background-color: #f8f9fa;
  color: #333;
  border: none;
  padding: 10px;
  margin-bottom: 10px;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.3s;
}

.list-group-item:hover {
  background-color: #e9ecef;
  transform: scale(1.05);
}

.list-group-item-selected {
  background-color: #007bff;
  color: #fff;
}

.list-group-item-success {
  background-color: #d4edda;
}

.list-group-item-danger {
  background-color: #f8d7da;
}

.text-success {
  color: #28a745;
}

.text-danger {
  color: #dc3545;
}

.timer {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
  color: #333;
}

.card-finished {
  background-color: #f8f9fa;
  border: none;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  padding: 20px;
  text-align: center;
}

.card-finished h2 {
  font-size: 22px;
  color: #333;
  margin-top: 20px;
}

.card-finished h3 {
  font-size: 18px;
  color: #333;
  margin-top: 10px;
}

.card-finished .btn {
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-top: 20px;
}

.card-finished .btn:hover {
  background-color: #0069d9;
}
</style>
