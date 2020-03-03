# Customizing response with several records

When you perform an action on several records \(bulk action\), you can customize the response as follow:

```javascript
// route/companies.js

const express = require('express');
const router = express.Router();
const Liana = require('forest-express-sequelize');
const models = require('../models');

router.post('/actions/mark-as-live', Liana.ensureAuthenticated,
  (req, res) => {
    const companyIds = req.body.data.attributes.ids;
    const companiesCount = companyIds.length;

    return models.companies
      .update({ status: 'live' }, { where: { id: companyIds }})
      .then(() => {
        const message = companiesCount > 1 ? 'Companies are now live!' : 'Company is now live!';
        res.send({ success: message });
      });
});

module.exports = router;
```

