# authentication-with-passportJS



Passport.js is a flexible authentication middleware (allowing users to log in) that can be fully customised and works great with connect/express.

It is flexible in the sense that it allows for different authentication strategies (think via Twitter, via our own user database - installed via separate modules that can be combined) and it allows us to specify any route or output during authentication.

The Local Strategy allows us to authenticate users by looking up their data in the app's database. It has some great examples how to use it.

In this post, we walk through the authentication flow, the next post discusses some partical use cases using passportjs.

DIFFERENT PARTS IN USING PASSPORT.JS
There are three main parts in using passport.js:

Requiring the module and using its passport.initialize() and passport.session() middleware with express.
Configuring passport with at least one Strategy and setting up passport's serializeUser and deserializeUser methods.
Specifying a route which uses the passport.authenticate middleware to actually authenticate a user.
The example clearly demonstrates the different items. We wont go over it again.

AUTHENTICATION REQUEST FLOW
With the tree parts configured as in the example, the following happens when a user tries the authenticate via the /login route:

When the user submits the login form, a POST request to /login is made resulting in the execution of the passport.authenticate middleware we've set up.
As the authenticate middleware for that route is configured to handle the local strategy, passport will invoke our implementation of the local strategy.
Passport takes the req.body.username and req.body.password and passes it to our verification function in the local strategy.
Now we do our thing: loading the user from the database and checking if the password given matches the one in the database.
In case of an Error interacting with our database, we need to invoke done(err). When we cannot find the user or the passwords do not watch, we invoke done(null, false). If everything went fine and we want the user to login we invoke done(null, user).
Calling done will make the flow jump back into passport.authenticate. It's passed the error, user and additional info object (if defined).
If the user was passed, the middleware will call req.login (a passport function attached to the request).
This will call our passport.serializeUser method we've defined earlier. This method can access the user object we passed back to the middleware. It's its job to determine what data from the user object should be stored in the session. The result of the serializeUser method is attached to the session as req.session.passport.user = { // our serialised user object // }.
The result is also attached to the request as req.user.
Once done, our requestHandler is invoked. In the example the user is redirected to the homepage.
SUBSEQUENT AUTHENTICATED REQUESTS FLOW
On subsequent request, the following occurs:

Express loads the session data and attaches it to the req. As passport stores the serialised user in the session, the serialised user object can be found at req.session.passport.user.
The general passport middleware we setup (passport.initialize) is invoked on the request, it finds the passport.user attached to the session. If is doesn't (user is not yet authenticated) it creates it like req.passport.user = {}.
Next, passport.session is invoked. This middleware is a Passport Strategy invoked on every request. If it finds a serialised user object in the session, it will consider this request authenticated.
The passport.session middleware calls passport.deserializeUser we've setup. Attaching the loaded user object to the request as req.user.
SUMMARY PASSPORT METHODS AND MIDDLEWARE
passport.initialize middleware is invoked on every request. It ensures the session contains a passport.user object, which may be empty.
passport.session middleware is a Passport Strategy which will load the user object onto req.user if a serialised user object was found in the server.
passport.deserializeUser is invoked on every request by passport.session. It enables us to load additional user information on every request. This user object is attached to the request as req.user making it accessible in our request handling.
Our Local Strategy is only invoked on the route which uses the passport.authenticate middleware.
Only during this authentication passport.serializeUser is invoked allowing us the specify what user information should be stored in the session.
OVERVIEW PASSPORT METHODS ATTACHED TO THE REQUEST
To finish an overview of passport methods accessible within request handlers:

req.login()
req.logout()
req.isAuthenticated()
req.isUnAuthenticated()


http://toon.io/understanding-passportjs-authentication-flow/
