+++
title = "Error Handling With Express.js and Typescript Made Easy"
date = "2024-11-13T12:10:22Z"
+++

A github repo is available with the completed code for the blog post <a href="https://github.com/jacobcons/error-handling-blog-post-code" target="_blank">here</a>

### Getting Started

The below dependencies are required (note that a workaround will be provided later in the post if you're using express 4).

`npm i express@5.01`

`npm i -D @types/express`

### Dealing with errors where the server is at fault
For example sending an http request to a url that doesn't exist, executing a malformed sql statement etc...

In these instances it's best to respond with a generic error message as to not give away server internals, with a status
code of 500.

Instead of wrapping lots of code in try/catch blocks and providing an http response in each one, 
we can leverage the fact that express passes control to a central error handling middleware when an error is thrown in a handler,
in order to have a single place to handle these errors
```typescript
app.get('/1', async (req: Request, res: Response) => {
  // this will throw an error since the url doesn't exist
  // the error handler below will be executed, being passed the thrown error
  await fetch('https://iasfijefasd.com')
  res.json({ message: 'success' })
})

const errorHandler: ErrorRequestHandler = (err, req, res, next) => {
  /*
  we simply log the error so we can see what went wrong on the server and return a generic error message to the client
  as to not expose how our server is working internally
  */
  console.log(err)
  res.status(500).json({ message: 'something went wrong!' });
};
app.use(errorHandler)
```

Note that if you are using express 4, errors thrown by async code e.g. `await fetch('https://iasfijefasd.com')`
will not automatically pass control to the error handler. To remedy this simply run `npm i express-async-errors` and include
an import to it at the top of the file that acts as your entry point to your server `import 'express-async-errors'`

### Dealing with errors where the client is at fault
e.g. sending data in an invalid format, not having permissions to access the data etc...

In these instances it's best to respond with a custom error message explaining what went wrong along with the correct status code.
This is so the client understands what they did wrong on their end.

We could simply respond with the message and status code we want directly in the handler, but this could get awkward when
calling functions that are responsible for doing so. Instead we can simply throw a custom error with the message and status code which can also be handled
by our error handling middleware:

```typescript
// contains status and message attributes
class CustomError extends Error {
  public status: number

  constructor(status: number, message: string) {
    super(message);
    this.status = status
  }
}

app.get('/2', async (req: Request, res: Response) => {
  const isValid = false
  if (!isValid) {
    // throwing the error will pass control the the error handler
    // this is a nice way to do it since functions you call inside this handler will also pass control to the error handler when they throw
    throw new CustomError(400, 'Invalid data!')
  }
  res.json({ message: 'success' })
})

const errorHandler: ErrorRequestHandler = (err, req, res, next) => {
  if (err instanceof CustomError) {
    // respond with the status code and message of the custom error
    res.status(err.status).json({ message: err.message })
    return
  }

  console.log(err)
  res.status(500).json({ message: 'something went wrong!' });
};
```

This should be enough to get you going with error handling. Of course depending on your requirements, you might want to
alter things e.g. custom error class taking in custom object to respond with, throwing named errors whose status code and
response are dealt with in the error handler to separate concerns.

Thanks for reading! If you have any questions, feel free to shoot over an email at [j@jacobcons.com](mailto:j@jacobcons.com) :)