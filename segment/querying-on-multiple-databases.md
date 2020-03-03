# Querying on multiple databases

```sql
SELECT *
FROM subscriptions s
INNER JOIN (
    SELECT * 
    FROM dblink('dbname=db2 port= 5449 host=host.docker.internal user=forest password=secret', $$ 
	    SELECT 
	        u.id
	    FROM users u
	    WHERE u.email LIKE '%wework%'
    $$) AS tb0 (u_id INTEGER) 
) AS tb0 ON (u_id = s.user_id)
```

