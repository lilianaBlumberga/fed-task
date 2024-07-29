<template>
    <div class="container text-white">
        <!-- Start Section -->
        <div class="row justify-content-center align-items-center min-vh-100" v-if="!questions.length">
            <div class="col-lg-6 col-md-8 col-sm-12">
                <div class="text-center">
                    <img src="https://cdn-icons-png.flaticon.com/512/2958/2958990.png" alt="">
                    <h1 class="mb-4 text-uppercase fw-bold" style="font-size: 80px;">Trivia Questions</h1>
                    <form @submit.prevent="fetchTrivia" class="mb-4">
                    <div class="form-group row">
                        <label for="amount" class="col-sm-4 col-form-label fw-bold">Number of Questions (5-10):</label>
                        <div class="col-sm-8">
                        <input type="number" id="amount" v-model="amount" min="5" max="10" class="form-control w-75" required />
                        </div>
                    </div>
                    <div class="form-group row">
                        <label for="difficulty" class="col-sm-4 col-form-label fw-bold">Difficulty:</label>
                        <div class="col-sm-8">
                        <select id="difficulty" v-model="difficulty" class="form-control w-75">
                            <option value="">Any</option>
                            <option value="easy">Easy</option>
                            <option value="medium">Medium</option>
                            <option value="hard">Hard</option>
                        </select>
                        </div>
                    </div>
                    <div class="form-group row">
                        <label for="type" class="col-sm-4 col-form-label fw-bold">Type of Questions:</label>
                        <div class="col-sm-8">
                        <select id="type" v-model="type" class="form-control w-75">
                            <option value="">Any</option>
                            <option value="multiple">Multiple Choice</option>
                            <option value="boolean">True/False</option>
                        </select>
                        </div>
                    </div>
                    <button type="submit" class="btn btn-primary border-0 px-5 py-3 mt-4 fw-bold text-uppercase">Get Trivia Questions</button>
                    </form>
                </div>
            </div>
        </div>

        <!-- Questions Section -->
        <div class="row justify-content-center align-items-center min-vh-100" v-if="questions.length">
            <div class="col-md-10">
                <div class="pt-4">
                <!-- Display the current question -->
                <div v-if="currentQuestionIndex < questions.length" class="card mb-3">
                    <div class="card-body">
                    <p class="fw-bold text-center" v-html="decodeHtml(questions[currentQuestionIndex].question)"></p>
                    <div class="d-flex flex-wrap justify-content-center">
                        <div v-for="(answer, idx) in questions[currentQuestionIndex].shuffledAnswers" :key="idx" class="form-check me-3 mb-2 px-4">
                        <input type="radio" :name="'question-' + currentQuestionIndex" :value="answer" v-model="userAnswers[currentQuestionIndex]" required class="form-check-input" @change="goToNextQuestion">
                        <label class="form-check-label">
                            {{ answer }}
                        </label>
                        </div>
                    </div>
                    </div>
                </div>
                </div>
            </div>
        </div>

        <!-- Modal for results -->
        <div class="modal fade" id="resultsModal" tabindex="-1" aria-labelledby="resultsModalLabel" aria-hidden="true" v-if="result !== null">
                    <div class="modal-dialog modal-dialog-centered text-dark">
                        <div class="modal-content">
                            <div class="modal-header">
                                <h5 class="modal-title fw-bold" id="resultsModalLabel">Results</h5>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                            </div>
                            <div class="modal-body">
                                <p>You got <b>{{ result }}</b> out of <b>{{ questions.length }}</b> correct!</p>
                            </div>
                            <div class="modal-footer">
                                <button type="button" class="btn btn-primary border-0 px-5 py-3 mt-4 fw-bold text-uppercase" @click="resetQuiz">Back to Start</button>
                            </div>
                        </div>
                    </div>
                </div>
    </div>
</template>
  
<script>
import axios from 'axios';
import { Modal } from 'bootstrap';

export default {
  data() {
    return {
      amount: 5,
      difficulty: '',
      type: '',
      questions: [],
      userAnswers: [],
      currentQuestionIndex: 0,
      result: null
    };
  },
  methods: {
    async fetchTrivia() {
      if (this.amount < 5 || this.amount > 10) {
        alert('Number of questions must be between 5 and 10.');
        return;
      }

      try {
        const response = await axios.get('https://opentdb.com/api.php', {
          params: {
            amount: this.amount,
            difficulty: this.difficulty,
            type: this.type
          }
        });
        if (response.data.response_code === 0) {
          this.questions = response.data.results.map(question => {
            return {
              ...question,
              shuffledAnswers: this.shuffleAnswers([question.correct_answer, ...question.incorrect_answers])
            };
          });
          this.userAnswers = new Array(this.questions.length).fill(null);
          this.currentQuestionIndex = 0; // Reset to the first question
          this.result = null;
        } else {
          console.error('Error fetching trivia questions');
        }
      } catch (error) {
        console.error('Error fetching trivia questions', error);
      }
    },
    shuffleAnswers(answers) {
      return answers.sort(() => Math.random() - 0.5);
    },
    decodeHtml(html) {
      const txt = document.createElement('textarea');
      txt.innerHTML = html;
      return txt.value;
    },
    goToNextQuestion() {
      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++;
      } else {
        this.submitAnswers();
      }
    },
    submitAnswers() {
      let correctCount = 0;
      console.log(this.result);
      this.questions.forEach((question, index) => {
        if (this.userAnswers[index] === question.correct_answer) {
          correctCount++;
        }
      });
      this.result = correctCount;
      console.log(this.result);
      this.$nextTick(() => {
        // Show modal after result is set
        const modalEl = document.getElementById('resultsModal');
        if (modalEl) {
          const modal = new Modal(modalEl);
          modal.show();
        }
      });
    },
    resetQuiz() {
      this.amount = 5;
      this.difficulty = '';
      this.type = '';
      this.questions = [];
      this.userAnswers = [];
      this.currentQuestionIndex = 0;
      this.result = null;
      // Close modal
      const modalEl = document.getElementById('resultsModal');
      if (modalEl) {
        const modal = Modal.getInstance(modalEl);
        if (modal) {
          modal.hide();
        }
      }
    }
  }
};
</script>
  
<style scoped>
.container {
    font-family: Arial, sans-serif;
    background: rgb(2,0,36);
    background: linear-gradient(90deg, rgba(2,0,36,1) 0%, rgba(163,120,226,1) 100%, rgba(0,212,255,1) 100%);
    box-shadow: rgba(0, 0, 0, 0.35) 0px 5px 15px;
    border-radius: 10px;
}

form {
    margin-bottom: 20px;
}

form div {
    margin-bottom: 10px;
}

.question {
    margin-bottom: 20px;
}

.btn {
    background: rgb(113,55,207);
    background: linear-gradient(90deg, rgba(113,55,207,1) 61%, rgba(55,83,207,1) 100%);

}
img {
    width: 90px;
}

.card{
    color:black;
}

  </style>
  