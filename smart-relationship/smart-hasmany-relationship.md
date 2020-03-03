# Smart hasMany relationship

In the example I have Missions that have many MissionApplications. MissionApplications have many CounterOffers.

We are creating here a hasMany relationship between Missions and CounterOffers.

First we declare the relationship in the `forest/missions.js` file.

```javascript
const { collection } = require('forest-express-sequelize');

// This file allows you to add to your Forest UI:
// - Smart actions: https://docs.forestadmin.com/documentation/reference-guide/actions/create-and-manage-smart-actions
// - Smart fields: https://docs.forestadmin.com/documentation/reference-guide/fields/create-and-manage-smart-fields
// - Smart relationships: https://docs.forestadmin.com/documentation/reference-guide/relationships/create-a-smart-relationship
// - Smart segments: https://docs.forestadmin.com/documentation/reference-guide/segments/smart-segments
collection('Missions', {
  actions: [],
  fields: [
    {
      field: 'counter offers',
      type: ['String'],
      reference: 'CounterOffers.id',
    },
  ],
  segments: [],
});
```

Then we declare the logic to retrieve the records fro the `routes/missions.js` file.

```javascript
const express = require('express');
const { PermissionMiddlewareCreator, RecordsGetter } = require('forest-express-sequelize');
const { Missions, MissionApplications, CounterOffers } = require('../models');

const router = express.Router();
const permissionMiddlewareCreator = new PermissionMiddlewareCreator('Missions');

// This file contains the logic of every route in Forest Admin for the collection Missions:
// - Native routes are already generated but can be extended/overriden - Learn how to extend a route here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/routes/extend-a-route
// - Smart action routes will need to be added as you create new Smart Actions - Learn how to create a Smart Action here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/actions/create-and-manage-smart-actions

// Create a Mission
router.post('/Missions', permissionMiddlewareCreator.create(), (request, response, next) => {
  // Learn what this route does here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/routes/default-routes#create-a-record
  next();
});

// Update a Mission
router.put('/Missions/:recordId', permissionMiddlewareCreator.update(), (request, response, next) => {
  // Learn what this route does here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/routes/default-routes#update-a-record
  next();
});

// Delete a Mission
router.delete('/Missions/:recordId', permissionMiddlewareCreator.delete(), (request, response, next) => {
  // Learn what this route does here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/routes/default-routes#delete-a-record
  next();
});

// Get a list of Missions
router.get('/Missions', permissionMiddlewareCreator.list(), (request, response, next) => {
  // Learn what this route does here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/routes/default-routes#get-a-list-of-records
  next();
});

// Get a number of Missions
router.get('/Missions/count', permissionMiddlewareCreator.list(), (request, response, next) => {
  // Learn what this route does here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/routes/default-routes#get-a-number-of-records
  next();
});

// Get a Mission
router.get('/Missions/:recordId', permissionMiddlewareCreator.details(), (request, response, next) => {
  // Learn what this route does here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/routes/default-routes#get-a-record
  next();
});

// Export a list of Missions
router.get('/Missions.csv', permissionMiddlewareCreator.export(), (request, response, next) => {
  // Learn what this route does here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/routes/default-routes#export-a-list-of-records
  next();
});

router.get('/Missions/:recordId/relationships/counter%20offers', permissionMiddlewareCreator.export(), (request, response, next) => {
  // Learn what this route does here: https://docs.forestadmin.com/documentation/v/v5/reference-guide/routes/default-routes#export-a-list-of-records
  const missionId = request.params.recordId;
  const recordsGetter = new RecordsGetter(CounterOffers);
  CounterOffers.findAll({
    include: [{
      model: MissionApplications,
      where: { MissionId : missionId },
    }],
  })
  .then(counterOffers => recordsGetter.serialize(counterOffers))
  .then(recordsSerialized => response.send(recordsSerialized))
  .catch(next);
});

module.exports = router;
```

Don't forget if you want to use include in a sequelize query to declare the relationships between models in your models folder \(belongsTo and hasMany on both sides\)

