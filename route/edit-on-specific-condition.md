# Edit on specific condition

```javascript
router.put('/companies/:companyId', Liana.ensureAuthenticated, (req, res, next) => {
  let companyId = req.body.data.id

  models.companies.findById(companyId)
    .then ( (company) => {
      if (company.status === 'live') {
        next();
      } else {
        res.status(403).send('Sorry, you can only edit Live company');
      }
    });
});

module.exports = router;
```

