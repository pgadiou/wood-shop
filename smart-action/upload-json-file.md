# Upload JSON file

```javascript
const P = require('bluebird');
const express = require('express');
const router = express.Router();
const Liana = require('forest-express-sequelize');
const models = require('../models');

// Smart action implemented in forest/sandbox_items.js
// Liana.collection('sandbox_items', {
//   actions: [{
//       name: 'Upload JSON',
//       type: 'bulk',
//       fields: [{
//           field: 'json',
//           widget: 'file picker'
//       }]
//   }],
// });

router.post('/actions/upload-json', Liana.ensureAuthenticated,
  (req, res) => {
    // return the correct sequelize model
    const sandboxItemId = req.body.data.attributes.ids[0];

    // define function to select and update the correct item
    async function addJsonToItem(itemId, attribute, json) {
      // select the item
      let item =  await models.sandbox_items.findOne({where: {'untitledId': itemId}})
      // update the item - important to specify the attribute to be updated
      return await item.update({[attribute]: json})
      .then((item)=>{
        res.send({ success: 'item updated!' })
      })
    }
  
    // Upload the json to the corresponding field

    // get the raw base64 file => if your field is a string and you want to insert the JSON as a base64 to use the file viewer, this is the value you want to return
    let rawFile = req.body.data.attributes.values.json
    // trim the base64string to delete the prefix
    let rawFileCleaned = rawFile.replace('data:application/json;base64','');
    // get string from base64 string => if your field is a string and you want to insert the JSON in it, this is the value you want to return
    let stringFile = Buffer.from(rawFileCleaned, 'base64').toString('utf8');
    // check that you can properly parse json from the string obtained
    let jsonFile;
    try {
      jsonFile = JSON.parse(stringFile)
    } catch (error){ 
      return res.status(400).send({error: 'not a correctly formatted json file'}) 
    }

    addJsonToItem(sandboxItemId, 'jsonText', stringFile)
    
});

module.exports = router;
```

