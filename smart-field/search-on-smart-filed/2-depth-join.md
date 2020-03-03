# 2 depth join

```javascript
{
  field: 'customer_mobile',
  type: 'String',
  get: async complaint => {
    if (complaint.address_id) {
      const customer = await customer_service.fetchCustomerEntryFromAddressId(complaint.address_id);
      return customer.username
    } else {
      if (complaint._crm_customer_id) {
        return customer_service.fetchCRMMobile(complaint._crm_customer_id)
      } else {
        const customer = await customer_service.fetchCRMCustomerEntry(complaint.crm_customer_id);
        return customer_service.fetchCRMMobile(customer);
      }
    }
  },
  search: async function(query, search) {
    let s = models.sequelize;

    query.include.push({
      model: models.address,
      include: [{
        model: models.customer,
    });

    // NOTICE: Mobile number search condition
    var searchCondition = models.sequelize.literal(`"address->customer"."customer_mobile" ILIKE '%${search}%'`)

    let searchConditions = _.find(query.where.$and, '$or');
    searchConditions.$or.push(searchCondition)
  },
}
```

