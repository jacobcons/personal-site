+++
title = "Technical decisions whilst making Emote"
date = "2024-05-08T14:45:18+01:00"
description = "A writeup on my thoughts on the different decisions/tradeoffs I made whilst building a backend with Express.js."

tags = []
+++

I recently finished building the backend for <a href="https://github.com/jacobcons/Emote" target="_blank">Emote</a>, which is a social network site where you can 
only post/comment using exclusively emojis. I fancied writing up some of my thoughts on the different decisions/tradeoffs I
made along the way. 

### SQL
I used the query builder knex with most SQL just written raw with the occasional use of knex for dynamic queries when
building it out gets ugly. I prefer writing raw sql since you don't have to learn the quirks and syntax of different ORMs
and can ensure you exactly know the sql being executed. This leads to a simpler mental model, and allows you to write
efficient queries making full use of all of your SQL dialect's features. However, I can see how an ORMs ability to
fetch related data and neatly format it in the json could reduce a lot of boilerplate as I had to make use of PostgreSQL's
JSON functions which can lead to some fairly complex queries. But, most of the time when you need to do anything
slightly complicated the ORM either generates inefficient SQL or it simply cant meet all your requirements and so you
have to do further processing on the data at the application level. Overall, I'm far more happy having to write a bit
more boilerplate if it gives me complete control over the SQL.

### Auth
I used JWTs for auth because less state and one less dependency in the form of redis for managing sessions sounded
appealing. The only downside I can see with them is there is no way to revoke tokens on demand. This could be useful, for example, 
if a malicious actor got a hold of one. There are remedies to this such as storing a blacklist of tokens or
using refresh tokens, but these all reintroduce state and hence defeat the purpose of using JWTs in the first place.

### Performance
I wrote a script to seed the database with realistic data for over 1M rows. This was useful to test the performance of some
of my more complex queries which I then tweaked to make better.

### Validation
The request body, query and route params are all validated against Joi schemas using reusable middlewares which
carry out the validation and pass control to the error handler if the data is in an invalid format.

### Error handling
Objects with a status code and message are passed to my error handler for a variety of different http errors. Also thrown
errors are automatically passed to the error handler with the express-async-errors package, so I don't have to try/catch every
async await statement.
