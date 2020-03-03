# Create a lead in Hubspot

```javascript
const express = require('express');
const router = express.Router();
const Liana = require('forest-express-sequelize');
const models = require('../models');
const P = require('bluebird');
const superagent = require('superagent');

  // adding action that creates a new company instance in Hubspot (to be implemented in forest/companies.js
  // actions: [{
  //   name: 'Create lead in Hubspot',
  //   type: 'single',
  // }],

router.post('/actions/create-lead-in-Hubspot', Liana.ensureAuthenticated,
  (req, res) => {
    const companyId = req.body.data.attributes.ids[0];

    // asynchronous function that returns the company's sequelize object
    async function setCompany() {
      let company =  await models.companies.findOne({where: {id: companyId}})
      return company
    }

    async function postLead() {
      // set the company object
      const company = await setCompany()
      // send the proper values formatted to fit the hubspot API expectations at the API endpoint
      return await superagent
      .post(`https://api.hubapi.com/companies/v2/companies?hapikey=${process.env.HUBSPOT_API}`)
      .send({"properties": [
        {
          "name":"name",
          "value":company.dataValues.name,
        },
        {
          "name":"description",
          "value": company.dataValues.description,
        },
        {
          "name":"city",
          "value": company.dataValues.city,
        },
        {
          "name":"industry",
          "value": company.dataValues.industry,
        },
        ]})
      .then(response => {
        res.send({ success: 'Lead is created!' })
      })
    }

    postLead()

});

module.exports = router;
```

