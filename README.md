# SQL CRUD
## Link of my csv files: 
#### restaurants: https://github.com/dbdesign-assignments-spring2023/sql-crud-Keyao666/blob/main/data/restaurants.csv
#### users: https://github.com/dbdesign-assignments-spring2023/sql-crud-Keyao666/blob/main/data/users.csv
#### posts: https://github.com/dbdesign-assignments-spring2023/sql-crud-Keyao666/blob/main/data/posts.csv

## Part 1: Restaurant finder
Firstly, I designed a database table named restaurants that would allow an application that uses it to find restaurants and a table named reviews that would hold reviews for any restaurant.
```SQL
sqlite3 restaurants.db
```

### Create tables
```SQL
create table restaurants (
  id INTEGER PRIMARY KEY,
  restaurant_name TEXT,
  category TEXT,
  price_tier TEXT,
  neighborhood TEXT,
  opening_hours TEXT,
  closing_hours TEXT,
  average_rating INTEGER,
  good_for_kids BOOLEAN
);

create table reviews (
  id INTEGER PRIMARY KEY,
  restaurant_id TEXT,
  comment TEXT
);
```
### Import csv file
```SQL
.mode csv
.import /Users/oliviasun/sql-crud-Keyao666/data/restaurants.csv restaurants --skip 1
```
### 1. Find all cheap restaurants in Kips Bay
```SQL
SELECT id, restaurant_name FROM restaurants WHERE neighborhood = "Kips Bay" AND price_tier = "Cheap";
```

### 2. Find all Mexican restaurants with 3 stars or more, ordered by the number of stars in descending order.
```SQL
SELECT id,restaurant_name FROM restaurants WHERE category = "Mexican" AND average_rating >= 3 ORDER BY average_rating DESC;
```

### 3. Find all restaurants that are open now.
```SQL
SELECT id,restaurant_name FROM restaurants WHERE strftime('%H:%M', 'now') BETWEEN opening_hours AND closing_hours;
```

### 4. Leave a review for a restaurant (pick any restaurant as an example).
```SQL
INSERT INTO reviews (restaurant_id, comment)
VALUES (1, "It tastes horrible");
```

### 5. Delete all restaurants that are not good for kids.
```SQL
DELETE FROM restaurants
WHERE good_for_kids = 'FALSE';
```

### 6. Find the number of restaurants in each NYC neighborhood.
```SQL
SELECT neighborhood, COUNT(*) AS num_restaurants
FROM restaurants
GROUP BY neighborhood;
```

## Part 2: Social media app
Here, I've created a database named social_media for allowing Users to share two kinds of content: Messages and Stories.
```SQL
sqlite3 social_media.db
```
### Create tables
```SQL
create table users (
  id INTEGER PRIMARY KEY,
  email TEXT,
  password TEXT,
  handle TEXT
);

create table posts (
    id INTEGER PRIMARY KEY,
    post_type TEXT,
    content TEXT,
    created_at DATETIME,
    send_id INTEGER,
    recipient_id INTEGER,
    is_visible BOOLEAN
);
```

### Import data into posts table
```SQL
.mode csv
.import /Users/oliviasun/Downloads/posts.csv posts --skip 1
.import /Users/oliviasun/Downloads/users.csv users --skip 1
```

### 1. Register a new User.
```SQL
INSERT INTO users (email, password, handle)
VALUES ('olivia-sun123@nyu.edu', 'thisismypassword', 'olivia-sun');
```
### 2. Create a new Message sent by a particular User to a particular User
```SQL
INSERT INTO posts (post_type, content, created_at,send_id,recipient_id,is_visible)
VALUES ('messages', 'How are you', datetime('now'),1,2,'TRUE');
```

### 3. Create a new Story by a particular User (pick any User for example).
```SQL
INSERT INTO posts (post_type, content, created_at, send_id, recipient_id, is_visible)
VALUES ('stories', "I'm feeling good", datetime('now'), 5, "", 'TRUE');
```

### 4. Show the 10 most recent visible Messages and Stories, in order of recency.
```SQL
SELECT id, post_type, content, created_at, is_visible FROM posts WHERE is_visible = 'TRUE' 
ORDER BY ABS(ROUND(JULIANDAY('now') - JULIANDAY(created_at))*24) ASC,id LIMIT 10;
```

### 5. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.
```SQL
SELECT post_type, content, send_id, recipient_id, created_at
FROM posts
WHERE post_type = 'messages'
  AND send_id = 541
  AND recipient_id = 736
  AND is_visible = 'TRUE'
ORDER BY ROUND(JULIANDAY('now') - JULIANDAY(created_at))*24 DESC LIMIT 10;
```

### 6. Make all Stories that are more than 24 hours old invisible.
```SQL
UPDATE posts
SET is_visible = 'FALSE'
WHERE post_type = 'stories' AND ROUND((JULIANDAY('now') - JULIANDAY(posts.created_at))*24) >= 24;
```

### 7. Show all invisible Messages and Stories, in order of recency.
```SQL
SELECT id, post_type, content, created_at,is_visible
FROM posts
WHERE is_visible = 'FALSE'
ORDER BY ROUND(JULIANDAY('now') - JULIANDAY(created_at))*24 DESC,id;
```

### 8. Show the number of posts by each User.
```SQL
SELECT users.handle, COUNT(posts.id) AS num_posts
FROM users
LEFT JOIN posts ON users.id = posts.send_id
GROUP BY users.id
ORDER BY num_posts DESC;
```

### 9. Show the post text and email address of all posts and the User who made them within the last 24 hours.
```SQL
SELECT posts.content, users.email, users.handle
FROM posts
INNER JOIN users ON posts.send_id = users.id
WHERE ROUND((JULIANDAY('now') - JULIANDAY(posts.created_at))*24) <= 24;
```

### 10. Show the email addresses of all Users who have not posted anything yet.
```SQL
SELECT users.email
FROM users
LEFT JOIN posts ON users.id = posts.send_id
GROUP BY users.id
HAVING COUNT(posts.id) = 0;
```