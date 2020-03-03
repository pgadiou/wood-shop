# Custom scopes

In the following example we need to restrict the access to the records of an Operation collection for which the merchant\_id matches the merchant\_id that is associated to the logged in user.

Implementing scopes implies overriding the get route for the collection targetted.

As such, you first need to set your app to enable the insertion of middlewares for the default routes.

Include the following to your `app.js` file

```javascript
const express = require('express');
const requireAll = require('require-all');
const fs = require('fs');
const cors = require('cors');

const app = express();

app.use(cors({
  origin: /forestadmin\.com$|localhost:.*/,
  allowedHeaders: ['Authorization', 'X-Requested-With', 'Content-Type'],
  maxAge: 86400, // NOTICE: 1 day
  credentials: true,
}));


fs.readdirSync('./decorators/routes').forEach((file) => {
  if (file[0] !== '.') {
    app.use('/forest', require(`./decorators/routes/${file}`));
  }
});

requireAll({
  dirname: __dirname + '/middlewares',
  recursive: true,
  resolve: Module => new Module(app),
});

module.exports = app;
```

Create a folder named decorators at the root of your app. Create a sub-folder named `routes`.

Then create a file with the name of the targeted collection. Now you need to add the logic of the middleware to be included.

```javascript
const express = require('express');
const router = express.Router();
const Liana = require('forest-express-mongoose');

// Create a hash with a key:value pair connecting the user email to the corresponding merchant_id 
const userList = {
    "philippeg@forestadmin.com": "FR1013469",
    // ...
}

router.get('/operations', Liana.ensureAuthenticated, (req, res, next) => {
    // If the user is part of the merchants team, then implement a filter on the data to be retrieved
    if (req.user.team === "Merchants") {
        console.log(req.query)
        // recreate a hash reprising the structure of Forest Admin filters
        // The new filter should filter on merchant_id and retrieve the records for which it equals the value of the merchant_id associated to the user email
        let newFilter = `{"field":"merchant_id","operator":"equal","value":"${userList[req.user.email]}"}`
        // Implement an if function to check for existing filter in the payload to ensure that the new filter can coexist with filters added by the user in the UI
        if (req.query.filters){
            req.query.filters= `{"aggregator":"and", "conditions":[${newFilter},${req.query.filters}]}`;
        } else {
            req.query.filters= newFilter;
        }
    }
    next();
});



module.exports = router;
```

