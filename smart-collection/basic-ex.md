# Basic ex

```javascript
// forest/customers.js
const Liana = require('forest-express-sequelize');
const models = require('../models');

Liana.collection('customers', {
  isSearchable: true,
  fields: [{
      field: 'id',
      type: 'Number',
    }, {
      field: 'firstName',
      type: 'String',
    }, {
      field: 'lastName',
      type: 'String',
    }],
});


// routes/customers.js
const Liana = require('forest-express-sequelize');
const express = require('express');
const router = express.Router();
const models = require('../models');
var JSONAPISerializer = require('jsonapi-serializer').Serializer;

var UserSerializer = new JSONAPISerializer('customers', {
  attributes: ['firstName', 'lastName'],
  keyForAttribute: 'camelCase'
});

var data = [
  { id: 1, firstName: 'Sandro', lastName: 'Munda' },
  { id: 2, firstName: 'John', lastName: 'Doe' }
];

router.get('/customers', Liana.ensureAuthenticated,(req, res, next) => {
  var customers = UserSerializer.serialize(data);
  res.send({ ...customers, meta:{ count: data.length }});
});

router.get('/customers/:customerId', Liana.ensureAuthenticated,(req, res, next) => {
  var customer = data.find((c) => c.id.toString() === req.params.customerId)
  if (!customer) { return res.status(404).send(); }
  var customerSerialized = UserSerializer.serialize(customer);
  res.send(customerSerialized);
});

module.exports = router;
```

