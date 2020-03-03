# Docker image postgreSQL

{% code title="docker-compose.yml" %}
```yaml
postgres :
  image : postgres:latest
  container_name : june
  ports :
    - "5450:5432"
  environment:
    - POSTGRES_DB=june
    - POSTGRES_USER=forest
    - POSTGRES_PASSWORD=secret
```
{% endcode %}

