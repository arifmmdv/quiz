<template>
  <div class="container">
    <div class="row">
      <div class="col-md-6 offset-md-3">

        <div class="card" v-if="!quiz.started">
          <div class="card-header">
            <h1>Technical Task</h1>
          </div>
          <div class="card-body">
            <div class="mb-3">
              <input type="text" v-model="quiz.name" class="form-control mb-1" placeholder="Enter Your Name">
              <div class="alert alert-danger" role="alert" v-if="!errors.name.length == 0">
                {{errors.name}}
              </div>
            </div>
            <div class="mb-3">
              <select name="" id="" v-model="quiz.quizId" class="form-control mb-1" >
                <option value="0">Choose test</option>
                <option :value="topic.id" v-for="topic in topics" :key="topic.id">{{topic.title}}</option>
              </select>
              <div class="alert alert-danger" role="alert" v-if="!errors.topic.length == 0">
                {{errors.topic}}
              </div>
            </div>
          </div>
          <div class="card-footer">
            <button @click="startQuiz()" class="btn btn-primary">Start</button>
          </div>
        </div>

        <div class="card" v-if="quiz.started && !quiz.finished && questions.length > 0">
          <div class="card-header">
            {{questions[quiz.currentQuestion].title}}
          </div>

          <div class="card-body">
            <div class="row">
              <div class="col-md-6 mb-2" v-for="answer in questions[quiz.currentQuestion].answers">
                <button
                    class="btn answer"
                    @click="selectAnswer(answer.id)"
                    :class="[questions[quiz.currentQuestion].selectedAnswer ==  answer.id ? 'btn-success' : 'btn-warning']"
                >
                  {{answer.title}}
                </button>
              </div>
            </div>

            <div class="alert alert-danger" role="alert" v-if="!errors.question.length == 0">
              {{errors.question}}
            </div>

            <div class="progress mt-3">
              <div class="progress-bar bg-success" role="progressbar" :style="{ width: (100 * (quiz.currentQuestion))/questions.length + '%' }"></div>
            </div>
          </div>

          <div class="card-footer">
            <button @click="nextQuestion()" class="btn btn-success">
              <span v-if="quiz.currentQuestion+1 < questions.length">Next</span>
              <span v-else>Finish</span>
            </button>
          </div>
        </div>

        <div class="card" v-if="quiz.finished">
          <div class="card-body">
            <h2>Thanks, {{quiz.name}}</h2>
            <p>You responded correctly {{quiz.score}} out of {{questions.length}}</p>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import axios from "axios";

export default {
  name: 'App',
  setup(){

    const quiz = ref({
      name: '',
      quizId: 0,
      started: false,
      finished: false,
      currentQuestion: 0,
      score: 0
    })
    const errors = ref({
      name: '',
      topic: '',
      question: '',
    })
    const topics = ref({})
    const questions = ref({})
    const selectedAnswers = ref([])

    onMounted(() => {

      var config = {
        method: 'get',
        url: 'https://printful.com/test-quiz.php?action=quizzes',
        headers: { }
      };

      axios(config)
          .then(function (response) {
            topics.value = response.data;
          })
          .catch(function (error) {
            console.log(error);
          });

    })

    const startQuiz = async () => {

      if(quiz.value.name.length == 0){
        errors.value.name = 'Name is required.'
        quiz.value.started = false;
      }else{
        errors.value.name = ''
      }

      if(quiz.value.quizId == 0){
        errors.value.topic = 'Please choose the topic'
        quiz.value.started = false;
      }else{
        errors.value.topic = ''
      }

      if(errors.value.topic == '' && errors.value.name == ''){
        quiz.value.started = true;
      }

      if(quiz.value.started == true){

        var questionsConfig = {
          method: 'get',
          url: 'https://printful.com/test-quiz.php?action=questions&quizId='+quiz.value.quizId,
          headers: { }
        };

        axios(questionsConfig)
            .then(function (response) {
              questions.value = response.data;

              questions.value.forEach(function(question) {
                var answersConfig = {
                  method: 'get',
                  url: 'https://printful.com/test-quiz.php?action=answers&quizId='+quiz.value.quizId+'&questionId='+question.id,
                  headers: { }
                };

                axios(answersConfig)
                    .then(function (response) {
                      question['answers'] = response.data;
                      question['selectedAnswer'] = "";
                    })
                    .catch(function (error) {
                      console.log(error);
                    });
              });
            })
            .catch(function (error) {
              console.log(error);
            });
      }

    }

    const selectAnswer = async (answer_id) => {
      questions.value[quiz.value.currentQuestion]['selectedAnswer'] = answer_id
    }

    const nextQuestion = async () => {

      if(questions.value[quiz.value.currentQuestion]['selectedAnswer'] == ''){
        errors.value.question = 'Please select answer'
      }else{
        errors.value.question = ''
        var quizConfig = {
          method: 'get',
          url: 'https://printful.com/test-quiz.php?action=submit&quizId='+quiz.value.quizId+'&answers[]='+questions.value[quiz.value.currentQuestion]['selectedAnswer'],
          headers: { }
        };

        axios(quizConfig)
            .then(function (response) {
              quiz.value.score = quiz.value.score+response.data.correct
              if(quiz.value.currentQuestion+1 == questions.value.length){
                quiz.value.finished = true;
              }else{
                quiz.value.currentQuestion = quiz.value.currentQuestion+1;
              }

            })
            .catch(function (error) {
              console.log(error);
            });
      }
    }

    return { quiz, topics, questions, selectedAnswers, errors, startQuiz, selectAnswer, nextQuestion }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.alert {
  text-align: left;
  padding: 5px 10px;
}
.answer {
  width: 100%;
}
</style>
