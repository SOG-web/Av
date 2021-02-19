
##  Authentication with Node using Passport.js
###  Build a Login System in Node.js

This tutorial will be focusing on the login and logout system portion of the website. This is called user authentication. If you’re using JavaScript, a library called [Passport.js](http://www.passportjs.org/) can be used for user authentication. Passport.js is essentially middleware used for authentication in Node.Js

# Get Started

Before typing any code, first install the following modules:

-   `express`  for handling your routes
-   `express-session`  for building a user session
-   `express-ejs-layouts`  layout support for  `ejs`  in Express
-   `connect-flash`  will be used to display flash messages. We will cover more on this later.
-   `passport`  to handle user authentication
-   `passport-local`, which is a strategy(authentication mechanism) with a username and password
-   `mongoose` for storing users in the database
-   `bcrypt`  for encrypting your passwords before you store them in your database. It’s a necessity to never store passwords in plain text for obvious security reasons.
-   `ejs`  module to send data from Express to  `ejs`  files

After running `npm init ` in the project folder .

This is the command you should run:

    npm i express express-session express-ejs-layouts connect-flash passport passport-local mongoose bcrypt ejs

When that’s done, follow the further steps listed below for the initial stages.

#### The index.js route

Create a folder in your project directory with the name of  `route`. Here, create a JavaScript file called  `index.js`. In the end, the location of  `index.js`  
will be at  `routes/index.js`.

In  `routes/index.js,`  write the following code:
```js

const router = express.Router();

//login page

router.get('/', (req,res)=>{

res.render('welcome');

})

module.exports = router;
```
-   `line 4`: When the user navigates to the  `root` directory (performs a  `GET` request) render the  `welcome.ejs`  page.
-    `line 12`: Export the router instance so that it can be used in other files.
#### The users.js route

Go to your  `routes`  folder and create a file called  `users.js`  such that the  `users.js`  location is  `routes/users.js`.
```js
const router = express.Router();

//login handle

router.get('/login',(req,res)=>{

res.render('login');

})

router.post('/login',(req,res,next)=>{

})

//logout

router.get('/logout',(req,res)=>{

})

module.exports = router;
```
In  `routes/users.js`,

-   `Lines 6 to 7`: Handle the respective  `GET` requests and render the appropriate pages
-   `Lines 11 and 17`: Handle the respective  `POST` requests. They will be filled in later on in this tutorial.
#### app.js
This will be your main file. In your root directory, create a file called  `app.js`. Here, write the following code
```js
const express = require('express');

const router = express.Router();

const app = express();

const mongoose = require('mongoose');

const expressEjsLayout = require('express-ejs-layouts')

//mongoose

mongoose.connect('mongodb://localhost/test',{useNewUrlParser: true, useUnifiedTopology : true})

.then(() => console.log('connected,,'))

.catch((err)=> console.log(err));

//EJS

app.set('view engine','ejs');

app.use(expressEjsLayout);

//BodyParser

app.use(express.urlencoded({extended : false}));

//Routes

app.use('/',require('./routes/index'));

app.use('/users',require('./routes/users'));

app.listen(3000);
```
-   `line 11`: Tells Express that you will be using  `ejs`  as your template engine

Now that you have defined your routes, let’s create the  `ejs`  files that will be rendered to the screen.

Create a  `views`  folder in your root directory and write the following files. These are standard, static  `ejs`  files. You can find the explanations for the code in the  [EJS documentation website](https://ejs.co/).
#### layout.ejs

In  `views/layout.ejs`:
```html
<html>  
<head>  
<title>Node.Js and Passport App</title>  
</head>  
<body>  
<%- body %>  
</body>  
</html>
```
#### welcome.ejs
```html
<html>  
<h1>Welcome!</h1>  
<p>Login</p>   
<a href="/users/login"> Login</a>  
</html>
```
#### login.ejs
```html
<h1> Login </h1>
<form action="/users/login" method="POST">
<div>
<label for="email">Email</label>
<input
type="email"
name="email"
placeholder="Enter Email"
/>
</div>
<div>
<label for="password">Password</label>
<input
type="password"
name="password"
placeholder="Enter Password"
/>
</div>
<button type="submit" class="btn btn-primary btn-block">Login</button>
</form>
```
Note: You have used the  `name`  attributes for the elements in the form as you will extract the values in the form by identifying each element by name.

To run this code, go to the command line to run the  `app.js`  file.
`node app`
Go to `localhost:3000` and you will find your output to be identical to this:

You have finally completed your beginning stages. It’s now time to use [Mongoose](https://mongoosejs.com/docs/) to save your users in your database.

# Save Users to Database

#### The user schema and model

As usual, you have to define a schema before creating documents and saving them to the database. First, create the  `models`  directory and there create a file called  `user.js`  such that it is  `models/user.js`. First, let’s create a schema in  `user.js`. Next, we’ll create a model from that schema.

In  `models/user.js`:
```js
const mongoose = require('mongoose');
const UserSchema = new mongoose.Schema({

name :{
type : String,
required : true
} ,
email :{
type : String,
required : true
} ,
password :{
type : String,
required : true
} ,
date :{
type : Date,
default : Date.now
}
});

const User= mongoose.model('User',UserSchema);

module.exports = User;
```
This means that your model will have a name, an email, its associated password, and the date of creation.

On  `line 20`  , we’re using that schema in your  `User`  model. All of your users will have the data in this format.

# User Authentication With Passport

#### LocalStrategy

Create a file  `/config/passport.js`  and then start by importing the following libraries:

```js
const LocalStrategy = require('passport-local').Strategy;  
const bcrypt = require('bcrypt');  
const User = require("../models/user");
```
You are now importing  `passport-local`  with its  `Strategy`  instance for a user authentication mechanism with a simple username and password. Since you will be comparing passwords this time, you need to decrypt the 
password that was returned by the database. For this reason, you will also bring in  `bcrypt`. Furthermore, since you are essentially comparing passwords returned from the database, you will import  `User`  model for database-related operations.

Further on, type the following code:
```js
module.exports = function(passport) {

passport.use(

new LocalStrategy({usernameField : 'email'},(email,password,done)=> {

//match user

User.findOne({email : email})

.then((user)=>{

if(!user) {

return done(null,false,{message : 'that email is not registered'});

}

//match pass

bcrypt.compare(password,user.password,(err,isMatch)=>{

if(err) throw err;

if(isMatch) {

return done(null,user);

} else {

return done(null,false,{message : 'pass incorrect'});

}

})

})

.catch((err)=> {console.log(err)})

})

)

passport.serializeUser(function(user, done) {

done(null, user.id);

});

passport.deserializeUser(function(id, done) {

User.findById(id, function(err, user) {

done(err, user);
});
});
};
```
-   `Line 2`: Configure the  `passport`  instance such that it will be using the  `LocalStrategy`  strategy. You have to specify the  `usernameField`  option, as this option will be used to identify the email address that will be compared against the database. You have identified with this option that you will be using the  `email input type`  element in the form. You can specify the  `passwordField`  option as well. However, by default, it is  `password`, so you don’t need to specify it here.
-   `Lines 5–24`: First, find whether the given email address is present in the database. If it is not, it’ll throw an error that the email isn’t recognized. Later on, you then compare whether the user-entered password matches the password in the database. If it does, it will return the relevant user information.
-   `Lines 25–34`: In order to support login sessions, `passport`  will use the  `serialize`  and  `deserialize`  methods on the  `user`  instances to and from the session.

### Handling the POST request to the login directory

You now need to tell Passport where you want to use user authentication. You will use this authentication mechanism on the  `login`  page.

Go to  `/routes/users.js`  and find the following piece of code:
![enter image description here](https://miro.medium.com/max/700/1*2T9smfs5Mo8wix96uwNejA.png)
Within this POST handler, write the following lines of code:

```js
passport.authenticate('local',{  
successRedirect : '/dashboard',  
failureRedirect : '/users/login',  
})(req,res,next);
```
This means:

-   If the user has successfully logged in, they will be redirected to the  `dashboard`  directory (`successRedirect`).
-   If the user does not log in successfully, redirect them to the  `login`  directory (`failureRedirect`).

Now go to `app.js` and start by importing the `passport` library:

`const passport = require('passport');`

Then assign this the  `LocalStrategy`  configuration you created earlier. Do this step just after the imports:

`require("./config/passport")(passport)`

Now, find the following lines of code:
![enter image description here](https://miro.medium.com/max/654/1*6405jff89WjJFRHyIf8cgQ.png)

At the comment  `CODE TO COME HERE`  (`line 7`) write the following code:

```js
app.use(passport.initialize());  
app.use(passport.session());
```

You can now use user authentication functionality in your app. However, you are not done yet. You still need to handle the redirect to the  `dashboard`  directory.

Go to  `/routes/index.js`  and type the following code:

```js
router.get('/dashboard',(req,res)=>{  
res.render('dashboard');  
})
```
Now run the code and go to `localhost:3000/users/login`. 
Notice that you can finally log in. This indicates successful user authentication.

Let’s now move on to display user data.

### Render the Dashboard to Display User Data

You’re almost done!

Go to  `/routes/index.js`  and find the following piece of code:

![enter image description here](https://miro.medium.com/max/700/1*n84G2ERW2GwioRWTR33Zsg.png)

Replace the  `res.render`  method as follows:

```js
res.render('dashboard',{  
user: req.user  
});
```

You’ve now sent the user information data to the web page. This user information data is returned by Passport.

Then go to  `/views/dashboard.ejs`  and modify the Welcome message like so:

```html
<p> Welcome <%= user.name %></p>
```

You are displaying the  `name`  field from the  `user`  data of the logged-in user.

If you log in with the correct credentials, this is the output:

### Handling Logout Requests

This is your last step, and the easiest one.

Go to  `/routes/users.js`  and find the following piece of code:

![enter image description here](https://miro.medium.com/max/700/1*fHZHPPeW8-TMb94ZWn-agw.png)

In this handler, write the following code:

```js
req.logout();  
res.redirect('/users/login');
```

he  `req.logout`  function is created by Passport. As the name suggests, it logs out the user session. Later on, you send a success flash message and then redirect the user to the  `login`  page.

However, if you now go to  `localhost:3000/dashboard`, you’re greeted with a server error. Now let’s handle this.

#### Prevent access to dashboard when logged out

Go to  `config`  and create a file called  `auth.js`.

In  `/config/auth.js` add:

```js
module.exports = {  
ensureAuthenticated : function(req,res,next) {  
if(req.isAuthenticated()) {  
return next();  
}  
req.flash('error_msg' , 'please login to view this resource');  
res.redirect('/users/login');  
}  
}
```
This creates a function called  `ensureAuthenticated`, which will ensure that a user has logged in (`req.isAuthenticated()`). If so, it moves on to the next middleware. If not, it throws an error and redirects the user to the  `login`  page again.

Now, go to  `/routes/index.js`  and import this function:
```js
const {ensureAuthenticated} = require("../config/auth.js")
```
Find the following code:
![enter image description here](https://miro.medium.com/max/700/1*KDL1fmQUBWipiN4-R_TCCA.png)

Now modify the `router.get` function like so:
![enter image description here](https://miro.medium.com/max/2400/1*KF6kxpSl8TCoDh8Q2O5n5w.png)
You’ve now indicated that you want to use this function as middleware in this directory.

The code will output the following. Click on Logout:

## All done
