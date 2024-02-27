# SQL CRUD

## Restaurant finder


[Link to database](./data/rest_data.csv)

### SQL commands

- Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example).

```
SELECT * FROM rest_table
WHERE neighborhood = 'manhattan' AND price_tier = 'cheap';
```

- Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.

```
SELECT * FROM rest_table WHERE category = 'Italian' AND average_rating >= 3 ORDER BY average_rating DESC;
```

- Find all restaurants that are open now

```
SELECT * FROM rest_table WHERE strftime('%H:%M', 'now') BETWEEN opening_hours AND '23:59'
   OR strftime('%H:%M', 'now') BETWEEN '00:00' AND opening_hours;
```

- Leave a review for a restaurant

```
INSERT INTO reviews (restaurant_id, rating, comment)
VALUES (
    (SELECT restaurant_id FROM rest_table WHERE name = 'mattis'),
    4,
    'Great experience!'
);
```

- Delete all restaurants that are not good for kids

```
DELETE FROM rest_table
WHERE good_for_kids = false;
```

- Find the number of restaurants in each NYC neighborhood.

```
SELECT neighborhood, COUNT(*) as restaurant_count
FROM rest_table
GROUP BY neighborhood;
```

## Social media app


[Link to user database](./data/users.csv)
<br> 
[Link to post database](./data/posts.csv)


**Code to import data from CSV file into SQLite database**

- User.csv
```
.mode csv
.headers on
.import users.csv users
```

- post.csv

```
.mode csv
.headers on
.import posts.csv posts
```

### Table structures

**User table structure**

```
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email TEXT,
    username TEXT, 
    password TEXT
    );
```

Primary Key: user_id <br>
Foreign Key: user_id

**Posts table structure**

```
CREATE TABLE posts ( 
    id INTEGER PRIMARY KEY, msg_sender INTEGER, 
    receiver INTEGER, 
    message TEXT, 
    message_time TEXT, 
    msg_date TEXT, 
    story_sender INTEGER, 
    story TEXT, 
    story_time TEXT, 
    story_date TEXT
     );
```

Primary Key: id <br>
Foreign Key: id

---

### Write a single SQL query to perform each of the following tasks:

- Register a new User.

```
INSERT INTO users (email, username, password) VALUES ('janesmith@example.com', 'janesmith', 'JaneSmith');
```

- Create a new Message sent by a particular User to a particular User (pick any two Users for example).

```
INSERT INTO posts (msg_sender, receiver, message, message_time, msg_date, story_sender, story, story_time, story_date) VALUES (312, 405, 'Hey, what's up?', '18:39', '02/14/2023', null, null, null, null);
```

- Create a new Story by a particular User (pick any User for example).

```
INSERT INTO posts (msg_sender, receiver, message, message_time, msg_date, story_sender, story, story_time, story_date) VALUES (null, null, null, null, null, 650, 'Hasta la Vista', '14:94', '02/24/2023');
```

- Show the 10 most recent visible Messages and Stories, in order of recency.

```
SELECT type, sender, receiver, content, time, date FROM ( SELECT 'message' AS type, msg_sender AS sender, receiver, message AS content, message_time AS time, msg_date AS date FROM posts WHERE msg_date IS NOT NULL UNION SELECT 'story' AS type, story_sender AS sender, NULL AS receiver, story AS content, story_time AS time, story_date AS date FROM posts WHERE story_date IS NOT NULL ) AS combined ORDER BY datetime(combined.date || ' ' || combined.time) DESC LIMIT 10;
```

- Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.

```
SELECT msg_sender AS sender, receiver, message AS content, message_time AS time, msg_date AS date FROM posts WHERE msg_date IS NOT NULL AND msg_sender = '<1>' AND receiver = '<365>' ORDER BY datetime(msg_date || ' ' || message_time) DESC LIMIT 10;
```

- Make all Stories that are more than 24 hours old invisible.
```
UPDATE posts SET visible = 0 WHERE story_date < DATE('now', '-1 day') AND story_date IS NOT NULL;
```


- Show all invisible Messages and Stories, in order of recency.
```
SELECT * FROM posts WHERE visible = 0 ORDER BY COALESCE(message_time, story_time) DESC;
```

- Show the number of posts by each User.
```
SELECT msg_sender, COUNT (*) AS TotalPosts FROM posts GROUP BY msg_sender UNION SELECT story_sender, COUNT ( * ) AS TotalPosts FROM posts GROUP BY story_sender;

```

- Show the post text and email address of all posts and the User who made them within the last 24 hours.

```
SELECT p.message, p.story, u.email, u.username FROM posts p JOIN users u ON p.msg_sender = u.id OR p.story_sender = u.id WHERE p.msg_date >= datetime('now', '-1 day') OR p.story_date >= datetime('now', '-1 day');
```
- Show the email addresses of all Users who have not posted anything yet.
```
SELECT email FROM users WHERE id NOT IN ( SELECT msg_sender FROM posts WHERE msg_sender IS NOT NULL UNION SELECT story_sender FROM posts WHERE story_sender IS NOT NULL );
```
