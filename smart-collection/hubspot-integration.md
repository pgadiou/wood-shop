# Hubspot integration



When creating an account, link it to a CRM account﻿  
**First step =&gt; create smart collection with salesforce accounts**﻿  
In my example I used Hubspot instead of Salesforce but the logic remains the same.﻿  
First step is to declare the collection and the fields that should be expected to be found for this collection.﻿  
JavaScript

```javascript
const Liana = require('forest-express-sequelize');const models = require('../models');﻿
Liana.collection('hubspot_companies', {  isSearchable: true,  fields: [{      field: 'id',      type:'Number',    }, {      field: 'name',      type: 'String',    }],});
```

﻿  
﻿  
Next step is to define the logic to retrieve the data of the smart collection in a `routes/your-model.js` file.﻿  
You first need to set variables according to the context to ensure the query follows the UX \(nb of records per page, index of the page you're on, search performed or not\)﻿  
You then need to define a serializer adapted to the format of the data that will be passed and the expected fields of the collection.﻿  
Finally you need to implement the API call, serialize the data obtained, filter depending on the search performed and return the payload.NB: I used the `superagent` module for the API call﻿  
JavaScript

```javascript
const Liana = require('forest-express-sequelize');const express = require('express');const router = express.Router();const models = require('../models');const P = require('bluebird');const JSONAPISerializer = require('jsonapi-serializer').Serializer;const superagent = require('superagent');﻿
router.get('/hubspot_companies', Liana.ensureAuthenticated, (req, res, next) => {﻿
  // set pagination parameters when exist (default limit is 250 as it is the max allowed by Hubspot)  let limit = 250  let offset = 0  req.query.page ? limit = parseInt(req.query.page.size) : limit  req.query.page ? offset = (parseInt(req.query.page.number) - 1) * limit : offset﻿
  // set search terms when exist  let search = null  req.query.search ? search = req.query.search : search﻿
  // define the serializer used to format the payload  const hubspotCompaniesSerializer = new JSONAPISerializer('hubspotCompanies', {    attributes: ['name'],    keyForAttribute: 'underscore_case',    id: 'companyId',    transform: function (record) {      record.name = record['properties']['name']['value'];      return record;    }  });﻿
  // implement function to call hubspot API and return companies  async function getCompanies() {    return hubspot_companies = await superagent    .get(`https://api.hubapi.com/companies/v2/companies/paged?hapikey=${process.env.HUBSPOT_API}&properties=name&limit=${limit}&offset=${limit}`)    .then(response => {      // parsing the answer from the API      companiesJSON = JSON.parse(response.res.text).companies      // serializing the companies to comply with the format expected by the Forest server      serializedCompanies = hubspotCompaniesSerializer.serialize(companiesJSON)      // return all data or data with a name containing the searched terms from the companies fetched      if (search) {        serializedCompanies.data = serializedCompanies.data.filter(function(item) {          return item.attributes.name.toUpperCase().includes(search.toUpperCase());        });        return serializedCompanies      } else {        return serializedCompanies      }    })  }﻿
﻿
  async function sendCompaniesPayload() {    let hubspotCompanies = await getCompanies()    // defining the count of companies fetched    let count = hubspotCompanies.data.length    return res.send({...hubspotCompanies, meta:{ count: count }})  }﻿
  sendCompaniesPayload()﻿
});﻿
﻿
module.exports = router;﻿
```

﻿  
**Second step =&gt; include a smart field that will reference the smart collection and allow to have an autocomplete input for the user when editing the field**﻿  
Add this to a `forest/your-model.js` file \(here the model is company\)﻿  
JavaScript

```javascript
const Liana = require('forest-express-sequelize');const models = require('../models');﻿
Liana.collection('companies', {  fields: [{    // adding a field that will allow to search on the smart collection hubspot_companies and return a relationship    field: 'crm id setter',    type: 'Number',    reference:'hubspot_companies.id',    },{    // adding a field that will allow to be directed on click to the company's profile in hubspot    field: 'crm link',    type: 'String',    get: (company) => {      return company.dataValues.crm_id ? 'https://app.hubspot.com/contacts/6332498/company/' + company.dataValues.crm_id : null      },  }],});
```

﻿  
**Third step =&gt; override the post route for the companies to catch the value passed by the user to the smart field and update the db column with the salesforce\_id**﻿  
You first need to adapt the `app.js` file to allow for the override. For this you need the module `body-parser`.﻿  
JavaScript

```javascript
const express = require('express');const requireAll = require('require-all');const fs = require('fs');const app = express();const cors = require('cors');const bodyParser = require('body-parser');﻿
app.use(bodyParser());﻿
app.use(cors({  origin: /forestadmin\.com$|localhost:.*/,  allowedHeaders: ['Authorization', 'X-Requested-With', 'Content-Type'],  maxAge: 86400, // NOTICE: 1 day  credentials: true,}));﻿
﻿
fs.readdirSync('./decorators/routes').forEach((file) => {  if (file[0] !== '.') {    app.use('/forest', require(`./decorators/routes/${file}`));  }});﻿
// fs.readdirSync('./routes').forEach((file) => {//   if (file[0] !== '.') {//     app.use('/forest', require('./routes/' + file));//   }// });﻿
﻿
// app.use(require('forest-express-sequelize').init({//   modelsDir: __dirname + '/models',//   envSecret: process.env.FOREST_ENV_SECRET,//   authSecret: process.env.FOREST_AUTH_SECRET,//   sequelize: require('./models').sequelize// }));﻿
requireAll({  dirname: __dirname + '/middlewares',  recursive: true,  resolve: Module => new Module(app),});﻿
module.exports = app;﻿
```

﻿  
Then add the following logic in a `decorator/routes/your-model.js` fileJavaScript

```javascript
const express = require('express');const router = express.Router();const Liana = require('forest-express-sequelize');﻿
// adding middleware to the create route to set the crm_id from the relationship information provided by the smart field crm id setter// the payload sent corresponds to the one that would have been sent if the crm_id had been entered directly in the create form﻿
router.post('/companies', Liana.ensureAuthenticated, (req, res, next) => {    if (req.body.data.relationships) {      req.body.data.attributes.crm_id = parseInt(req.body.data.relationships['crm id setter'].data.id)      next();    } else { next()}});﻿
router.put('/companies/:id', Liana.ensureAuthenticated, (req, res, next) => {  console.log('test')    if (req.body.data.relationships) {      req.body.data.attributes.crm_id = parseInt(req.body.data.relationships['crm id setter'].data.id)      next();    } else { next()}});﻿
module.exports = router;
```

