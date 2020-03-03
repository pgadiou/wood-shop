# Loading using models' hook

{% code title="models/companies.js" %}
```javascript
module.exports = (sequelize, DataTypes) => {
  const Model = sequelize.define('companies', {
    'name': {
      type: DataTypes.STRING,
    },
    'industry': {
      type: DataTypes.STRING,
    },
    'headquarter': {
      type: DataTypes.STRING,
    },
    'status': {
      type: DataTypes.STRING,
    },
    'description': {
      type: DataTypes.STRING,
    },
    'created_at': {
      type: DataTypes.DATE,
    },
    'updated_at': {
      type: DataTypes.DATE,
    },
    'certificate_of_incorporation_id': {
      type: DataTypes.UUID,
    },
    'proof_of_address_id': {
      type: DataTypes.UUID,
    },
    'bank_statement_id': {
      type: DataTypes.UUID,
    },
    'passport_id': {
      type: DataTypes.UUID,
    },
  }, {
    tableName: 'companies',
    underscored: true,

    schema: process.env.DATABASE_SCHEMA,
    hooks: {
      afterFind: async (companies, options) => {
        const isFindOne = !companies.length;
        const limit = isFindOne ? 1 : companies.length;
        // test
        if (isFindOne) { companies = [companies] }
        const companiesIds = companies.map((company) => company.id)
        console.log(companiesIds)
        // end test
        function getTotoValue(order, product) {
          return order.ref + product.label
        }

        const orders = await sequelize.models.orders.findAll({ limit })
        const products = await sequelize.query(`
          SELECT id, label
          FROM products
          LIMIT ${limit}
        `)

        for (let i = 0; i < limit; i++) {
          companies[i].toto = getTotoValue(orders[i], products[0][i]);
        }

        return isFindOne ? companies[0] : companies;
      }

  Model.associate = (models) => {
  };
  return Model;
};
```
{% endcode %}

