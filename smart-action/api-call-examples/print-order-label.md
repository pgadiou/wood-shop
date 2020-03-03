# Print order label

{% code title="forest/orders.js" %}
```javascript
// $ npm install --save node-fetch

const Liana = require('forest-express-sequelize');

Liana.collection('orders', {
  actions: [{
    name: 'Print order label'
  }],
});
```
{% endcode %}

{% code title="routes/orders.js" %}
```javascript
const express = require('express');
const router = express.Router();
const Liana = require('forest-express-sequelize');
const models = require('../models');
const fetch = require('node-fetch');

router.post('/actions/print-order-label', Liana.ensureAuthenticated,
  (request, response) => {
  const orderIds = request.body.data.attributes.ids;
  const promises = [];

  for (orderId of orderIds) {
    const promise = models.orders.findById(orderId)
      .then(order => {
        if (!order) {
          return response.status(404).send({ error: 'Order not found.' })
        }
        return fetch('https://your-printer/print', {
          method: 'post',
          body:    JSON.stringify({ label: order.label }),
          headers: { 'Content-Type': 'application/json' },
        })
        .then(response => response.json())
        .then((response) => {
          return { orderId: orderId, message: response };
        })
        .catch((error) => ({ orderId: orderId, message: error.message }));
      });
    promises.push(promise);
  }

  return Promise.all(promises)
    .then((responses) => {
      let html = '';
      responses.forEach(response => {
        html += `<div>Order id ${response.orderId}, response: ${response.message}</div>`;
      });

      response.json({ html: html });
    })
    .catch(() => response.status(500).send({ error: 'Oops, something went wrong!' }));
});

module.exports = router;
```
{% endcode %}

