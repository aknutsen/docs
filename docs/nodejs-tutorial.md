# Using Auth0 with node.js

Integrating Auth0 with node is straight forward. At the end of this tutorial you will have a working web site, capable of authenticating users with an external identity provider (no more passwords in your app!).

##Before you start

1. We assume you have node.js installed and you are familiar with it.
2. We also assume you have Google OAuth2 connection enabled. If you haven't done so, this [tutorial](enable-simple-connection) shows how to do it.

##Integrating Auth0 with a Node app

If you are building a node based app, you are very likely using [__passportjs__](http://passportjs.org/), the well known middleware for authentication. We've built a [strategy for passport](https://github.com/qraftlabs/passport-auth0) that makes integration even simpler. There's not really much magic there, just boilerplate code.

> 'Strategies' are in essence __passport__ plugins. 

Here's what you do for integrating Auth0 in your app with __passport__:

#####1. Install the Auth0 strategy

	npm install passport-auth0

> __TIP__: Don't have an app to play with? You can create one very easily with express:
>	npm install express -g
>	express 
> That will create a simple "hello world" express website.

#####2. Initialize passport-auth0

Create a new file __setup-passport.js__ with the following code:

	var passport = require('passport'),
		Auth0Strategy = require('passport-auth0');

	var strategy = new Auth0Strategy({  
	  domain:        	'@@account.namespace@@',
	  clientID:         '@@account.clientId@@',
	  clientSecret:     '@@account.clientSecret@@',
	  callbackURL:      '/callback'
	},
	function(accessToken, refreshToken, profile, done) {
	  //Some tracing info
	  console.log('profile is', profile);
	  return done(null, profile);
	});

	passport.use(strategy);

	// This is not a best practice, but we want to keep things simple for now
	passport.serializeUser(function(user, done) {
		done(null, user); 
	});

	passport.deserializeUser(function(user, done) {
		done(null, user);
	});

	module.exports = strategy; 

> __TIP__: you typically would put this file under a 'lib' folder

The __clientId__, __clientSecret__ and __domain__ are available on the [settings](https://app.auth0.com/#/settings) page. Keep this page open. We will need one last thing later on.

#####3. Initialize passport in your app
In the startup file (e.g. _server.js_ or _app.js_) add:

	var passport = require('passport');
	var strategy = require('setup-passport');

and then in the __app.configure__ function:

	app.configure(function(){
	  ...
	  app.use(passport.initialize());
	  app.use(passport.session());
	  ...
	  app.use(app.router);
	});

The last bit of code you will need are the handlers for the passport callbacks:

	//Callback handler
	app.get('/callback', 
	passport.authenticate('auth0', { failureRedirect: '/authfailure' }), 
	function(req, res) {
	  if (!req.user) {
		throw new Error('user null');
	  }
	  res.redirect("/");
	}
	);

	//Authentication error handler
	app.get('/authfailure', function (req, res) {
		res.send('Authentication failed');
	});

	//Logout
	app.get('/logout', function(req, res){
		req.logout();
		res.redirect('/');
	});

	//Login
	app.get('/login', 
		passport.authenticate('auth0', { connection: 'google-openid' }), 
		function (req, res) {
		res.redirect("/");
	});

> Notice the `connection` parameter passed in the login? This is used by Auth0 to determine which identity provider to use. This process is also known as the _"home realm discovery"_. The example above assumes you enabled the __google-openid__ connection.  

You are almost done! 

#####4. Setup the callback URL in Auth0

Did you keep your settings page open? Make sure the __App Callback URLs__ in Auth0 is configured with your app's callback URL:

![](img/settings-callback.png)

> Notice that Auth0 supports many callbacks. Just use ',' as the delimiter.

## Testing the app:

Open a browser an navigate to the login URL (e.g. http://localhost:3000/login)

Congratulations! 

##Suggested follow up tutorials

You have your basic node.js app running. If you want to get a deeper understanding of how this works, go ahead and read our [node.js under the hood](node-underthehood) tutorial.

Also, it is always a good thing to be familiar with the underlying [protocols](protocols).