# Docker image mongoDB

{% code title="docker-compose.yml" %}
```yaml
mongodb:
    image: mongo:latest
    container_name: "beamerynew"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=forest
      - MONGO_INITDB_ROOT_PASSWORD=secret
    ports:
        - 27040:27017
```
{% endcode %}

