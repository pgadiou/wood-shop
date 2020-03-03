# Override count route

```javascript
router.get('/users/count', Liana.ensureAuthenticated, (req, res, next) => {
  res.send({ count: 100000000 });
});
```

