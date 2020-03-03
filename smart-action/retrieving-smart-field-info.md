---
description: Get smart field in a smart actin
---

# Retrieving smart field info

```javascript
// forest/users.js
const Liana = require('forest-express-sequelize');

Liana.collection('users', {
  fields: [{
    field: 'fullemail',
    type: 'String',
    get: (user) => {
      return user.email + ' + ' + 'hello';
    }
  }],
  actions: [{
    name: 'test',
    type: 'single'
  }]
});

// routes/users.js
const express = require('express');
const router = express.Router();
const Liana = require('forest-express-sequelize');
const models = require('../models');

router.post('/actions/test', Liana.ensureAuthenticated,
  (req, res, next) => {
  const userId = req.body.data.attributes.ids[0];
  return models.users
    .findByPk(userId)
    .then(user => new Liana.ResourceSerializer(Liana, models.users, user, null, {}, {}).perform())
    .then((userSerialized) => {
      // NOTICE: Liana.ResourceSerializer will compute all Smart Field values of the record.
      console.log(userSerialized);
      return res.send({ success: `Top Top ${userSerialized.data.attributes.fullemail}` });
    })
    .catch(next);
});

module.exports = router;
```

