# Printify Quiz App Trial

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Details
Quiz App created by using Vue 3 & Composition API
onMounted method it pulls all quizzes.
When we start quiz it checks if fields are empty and show relevant errors. 
Then it creates question with question id, title, answers and selectedAnswer as empty string.
Select Answer updates question's Selected Answer and
Next Question checks if answer is empty and prevents to go to next question. If question is not empty then checks if answer is correct from the database and update the score.
and we show the results
