# Smart collection with hasMany relationship to a "classic" collection

{% code title="forest/movies\_bis.js" %}
```javascript
const Liana = require('forest-express-sequelize');
const models = require('../models');

Liana.collection('movies_bis', {
  isSearchable: true,
  fields: [{
      field: 'title',
      type: 'String'
    }, {
      field: 'roles',
      type: ['String'],
      reference: 'roles.id'
    }],
});
```
{% endcode %}

{% code title="routes/movies\_bis.js" %}
```javascript
const Liana = require('forest-express-sequelize');
const express = require('express');
const router = express.Router();
const models = require('../models');
const P = require('bluebird');
const JSONAPISerializer = require('jsonapi-serializer').Serializer

const moviesSerializer = new JSONAPISerializer('movies_bis', {
  attributes: ['title', 'roles'],
  roles: {
    ref: 'id',
    relationshipLinks: {
      related: function (record, current, parent) {
        return '/forest/movies_bis/' + parent.id +
          '/relationships/roles/'
      }
    }
  }
});

function getMovie(movieId){
  return models.movies.findAll({ where: {id: movieId}})
}

async function fetchMovieDetails(req, res){
  let movie = await getMovie(req.params.id)
  movie[0].dataValues.roles = []
  movie = moviesSerializer.serialize(movie[0].dataValues)
  return res.send(movie)
}

router.get('/movies_bis', Liana.ensureAuthenticated, (req, res, next) => {
  const limit = parseInt(req.query.page.size) || 20;
  const offset = (parseInt(req.query.page.number) - 1) * limit;
  const queryType = models.sequelize.QueryTypes.SELECT;
  let conditionSearch = '';

  if (req.query.search) {
    conditionSearch = `movies.title LIKE '%${req.query.search.replace(/\'/g, '\'\'')}%'`;
  }

  const queryData = `
    SELECT movies.id,
      movies.title
    FROM movies
    ${conditionSearch ? `WHERE ${conditionSearch}` : ''}
    LIMIT ${limit}
    OFFSET ${offset}
  `;

  const queryCount = `
    SELECT COUNT(*)
    FROM movies
    ${conditionSearch ? `WHERE ${conditionSearch}` : ''}
  `;

  return P
    .all([
      models.sequelize.query(queryData, { type: queryType }),
      models.sequelize.query(queryCount, { type: queryType }),
    ])
    .spread((moviesList, moviesCount) => {
      const movies = moviesSerializer.serialize(moviesList);
      const count = moviesCount[0].count
      res.send({ ...movies, meta:{ count: count }});
    })
    .catch((err) => next(err));
});

router.get('/movies_bis/:id', Liana.ensureAuthenticated, (req, res, next) => {
  if(req.params.id === "count") {next()
    } else {
      fetchMovieDetails(req, res)
    }
 
});

router.get('/movies_bis/:id/relationships/roles', Liana.ensureAuthenticated, (req, res, next) => {
    let limit = parseInt(req.query.page.size) || 10;
    let offset = (parseInt(req.query.page.number) - 1) * limit;

    let queryType = models.sequelize.QueryTypes.SELECT;

    let countQuery = `
      SELECT COUNT(*)
      FROM roles
      WHERE roles.movie_id = ${req.params.id};
    `;

    console.log(countQuery)

    let dataQuery = `
      SELECT *
      FROM roles
      WHERE movie_id = ${req.params.id}
      LIMIT ${limit}
      OFFSET ${offset}
    `;

    return P
      .all([
        models.sequelize.query(countQuery, { type: queryType }),
        models.sequelize.query(dataQuery, { type: queryType })
      ])
      .spread((count, roles) => {
        return new Liana.ResourceSerializer(Liana, models.roles, roles, null, {}, {
          count: count[0].count
        }).perform();
      })
      .then((roles) => {
        res.send(roles);
      })
      .catch((err) => next(err));
  });


module.exports = router;
```
{% endcode %}

Response from the route /yourModel/:id must include a relationships subdocument \(even if empty\) to point towards the link of the relationship route.

`{"data":{"type":"movies_bis","id":"6","attributes":{"title":"The Matrix"},"relationships":{"roles":{"data":[],"links":{"related":"/forest/movies_bis/6/relationships/roles/"}}}}}`

