---
layout: post
title: Authenticating with JSON Web Tokens and Passportjs
comments: true
---

This article is aimed at developers who are looking to intagrate JWTs into their stateless API.
I won't be going through the setup of your express app, but will only be focussing on how to integrate JWT authentication and we'll also be looking at how JWTs can work in parallel with other authentication / authorazation strategies such as Facebook and Twitter.

### What you'll need

* Some knowledge on how to work with Express.js and passport.js
* Basic Node.js knowledge

## JSON web tokens (JWT)

[JWTs](https://jwt.io/) are an open, industry standard [RFC 7519](https://tools.ietf.org/html/rfc7519) method for representing claims securely between two parties.

In short, this means that your application will generate a random token, return it to the user, and allow the user to use that token as a means of authenticating with your application.
Traditional APIs might store a session server side or keep track of the user in a different way, but there are multiple reasons why you might want to keep your API stateless.

JWTs consists out of three parts; a header, a payload and a signature. [Read more about the JWT structure](https://jwt.io/introduction/)

## PassportJs + JWT

[Passport](http://passportjs.org/) allows you to easily integrate JWTs into your application simply by defining the correct strategy.
While making a custom implementation isn't that difficult, we'll be using an existing strategy: [passport-jwt](https://github.com/themikenicholson/passport-jwt).

[Passport-jwt](https://github.com/themikenicholson/passport-jwt) does a good job at implementing some additional features such as validating the experation of the token and allowing multiple ways of passing the token from the client side.

There's plenty of documentation available, the easiest way to implement passport-jwt is as follows:

```javascript
const opts = {
	jwtFromRequest: ExtractJwt.fromAuthHeader(),
	secretOrKey: 'Your secret here',
	issuer: 'myApp',
	audience: 'myapp.com',
	ignoreExpiration: false,
	algorithms: ['HS256']
};

passport.use(new JwtStrategy(opts, (payload, done) => {
	const User = mongoose.model('User', UserSchema);

	// Look up the user based on the id in the payload
	return User.findById(payload.id)
		.then(user => {
			if (!user) {
				return done(null, false, 'User not found');
			}

			return done(null, user);
		})
		.catch(err => {
			return done(err);
		});
}));
```

In this case, we'll asume that the user id was included in the payload when signing the token.
In our strategy, we'll simply use `payload.id` to retrieve the user from our database.

If you want to protect a route and only allow authorized users:

```
router.get('/profile', passport.authenticate('jwt', { session: false }), (req, res) => {
	res.send(req.user);
});
```

## Generating a JSON Web Token

We'll need a way to actually generate JWTs, there's multiple ways to do this, but I'll be focussing on traditional web applications, where the user is required to login first with their email and password.
Since we're using Passport, we can simply integrate [passport-local](https://github.com/jaredhanson/passport-local).

We can then create our login route:

```javascript
router.post('/login', (req, res, next) => {
	passport.authenticate('local-login', (err, user, info) => {
		if (err) {
			return res.status(401).send({
				success: false,
				error: err
			});
		}

		if (!user) {
			return res.status(401).send({
				success: false,
				error: info
			});
		}

		return res.send({
			success: true
		});
	})(req, res, next);
});
```

At this point a user can login with his/her username and passport, when successfull, our API will respond, but we want our API to respond with a token, so the client side can use this for further API requets.

To generate the token, we'll be using [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken).

```javascript
const jwt = require('jsonwebtoken');

// Generate JWT with lifetime
const token = jwt.sign({
	id: user.id
}, 'Your secret here', {
	expiresIn: 432000, // In seconds = 5 days
	audience: 'myapp.com',
	issuer: 'myApp'
});
```

Insted of just returning `success: true`, we'll include the token as well:

```javascript
return res.send({
	success: true,
	token: token
});
```

Once we have the token, we can include it as a header in every request.

```bash
curl -X GET -H "Authorization: JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjU4NGJkZGFhY2Y4NWQ1MzBhNTE5OGM0ZCIsImlhdCI6MTQ4MzEwNTI0NSwiZXhwIjoxNDgzNzEwMDQ1LCJhdWQiOiJtZS53b3V0ciIsImlzcyI6IndvdXRyLm1lIn0.Hhox6e5zMXpuaicjnaFNkD6PaslB2_oDexHEdanY7t8" "http://localhost:4000/api/profile"
```

## What's next?

In the next post, we'll be looking at how we can use JWTs in parallel with other Passport strategies such as Facebook and Twitter.
I'll be including the link here once the article goes live.