# API integration

```javascript
const Liana = require('forest-express-sequelize');
const express = require('express');
const router = express.Router();
const models = require('../models');
const P = require('bluebird');
const JSONAPISerializer = require('jsonapi-serializer').Serializer;
const superagent = require('superagent');

router.get('/quiz_questions', Liana.ensureAuthenticated, (req, res, next) => {
  return P
  .all(
    [
    quiz_questions = superagent
    .get('https://opentdb.com/api.php?amount=10&category=11&difficulty=hard')
    .then(res => {
        //parse a JSON from the response obtained
         json = JSON.parse(res.res.text);
         questions_array = json.results;
         //iterate on the questions array to add the id
         for (var i=0; i<10; i++) {
          var question = questions_array[i]
          question["id"] = i+1;
          question["question"]= question["question"].replace(/&quot;/g, "'")
         };
         //define the needed serializer
         const quizQuestionSerializer = new JSONAPISerializer('quiz_questions', {
            attributes: ['question', 'correct_answer'],
            keyForAttribute: 'underscore_case'
          });
         const quizQuestions = quizQuestionSerializer.serialize(questions_array);
         return quizQuestions
    })
    ]
  )
  .spread((quizQuestions) => {
    console.log(quizQuestions)
    res.send({ ...quizQuestions, meta:{ count: 10 }})
  });

});


module.exports = router;
```

