# Prevent specific users to access data

{% code title="index.js" %}
```javascript
const models = require('../../models');
const requireAll = require('require-all');
const Liana = require('forest-express-sequelize');


// middleware checking the email of the user and return 400 if user not allowed
function scopeMiddleware(request, response, next) {
  Liana.ensureAuthenticated(request, response, (error) => {
    if (error || !request.user) {
      return next();
    }

    const notAllowedUsers = ['yourdevname@mail.com'];

    if (notAllowedUsers.includes(request.user.email)) {
      response.status(400).send();
    } else {
      next();
    }
  });
}


module.exports = function (app) {
  // call middleware
  app.use('/forest', scopeMiddleware);

  require('lumber-forestadmin').run(app, {
    modelsDir: __dirname + '/../../models',
    envSecret: process.env.FOREST_ENV_SECRET,
    authSecret: process.env.FOREST_AUTH_SECRET,
    sequelize: models.sequelize,
  });

  requireAll({
    dirname: __dirname + '/../../routes',
    recursive: true,
    resolve: Module => app.use('/forest', Module)
  });
};
```
{% endcode %}

