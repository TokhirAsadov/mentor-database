# Create Table Example
```psql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT DEFAULT 18
);
```
***
