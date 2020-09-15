---
title: Authentication with Django and Single Page Apps
description: TL;DR Just use sessions.
date: "2020-09-14T02:00:00.000Z"
---

When using Django for a single page application like a React, Vue or Angular app, a typical first initial question is: "how do I handle authentication?"

There are a few options out there for people:

- Use [JSON Web Tokens](https://simpleisbetterthancomplex.com/tutorial/2018/12/19/how-to-use-jwt-authentication-with-django-rest-framework.html)
- Use REST Framework's [`TokenAuthentication`](https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication)
- Use [django-rest-knox](http://james1345.github.io/django-rest-knox/) (an improvement over REST Framework's `TokenAuthentication`)

It's definitely a cause for [confusion](https://www.reddit.com/r/django/comments/irs2of/can_i_use_both_jwt_and_regular_tokens_in_one_drf/) among developers (myself included).

I'd like to offer a third option: **just use Django**. If you're just curious about the code for how to do this, check out the [sample Django and React app](https://github.com/msukmanowsky/django-spa-auth).

Let's first look at the options mentioned above and explain their short comings.

# JSON Web Tokens (JWTs)

[JWTs](https://jwt.io/introduction/) are a newish ([2010](https://en.wikipedia.org/wiki/JSON_Web_Token)) standard for representing claims between two parties.

The "claim" can be anything but usually it's as simple as: "I'm user 1234, please let me access the resource at `/api/something`".

The server can quickly validate that the claim is accurate using the hashed signature contained within a JWT (for more info, read the [JWT introduction](https://jwt.io/introduction/)).

So on every request, we send the JWT via an HTTP header:

```
Authorization: Bearer <jwt>
```

OK cool, we may not understand everything when it comes to JWTs, but we get that a server gives us a token, we include it on every request, and we get access to our protected `/api/something` resource.

But if we need to send this on every request, we need to persist these credentials somewhere. In a native mobile environments, there are [secure options](https://docs.expo.io/versions/latest/sdk/securestore/), but on browsers we only have `localStorage` or `sessionStorage`, both of which are [100% insecure](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#local-storage).

Both `localStorage` and `sessionStorage` are vulnerable to [cross site scripting (XSS)](https://owasp.org/www-community/attacks/xss/) attacks.

Browsers really only have one place that's safe from these attacks: cookies (note non-HTTP cookies stored and accessible via `document.cookie` are not secure as other scripts are able to read these). We could store JWTs in cookies, but there are still more fundamental issues with JWTs.

First and most importantly, JWTs are [vulnerable to brute force attacks](https://owasp.org/www-chapter-vancouver/assets/presentations/2020-01_Attacking_and_Securing_JWT.pdf) once intercepted. Thus, they're stongly recommended as a temporary authentication mechanism to obtain something more secure like a session ID or OAuth access token (stored via cookie).

The other issue with JWTs is that they **cannot be invalidated** which is an issue if you want to handle any of these cases:

- logout
- compromised accounts
- password changes
- permission changes
- user de-provisioning

In short, if you do use JWTs, please ensure they are short lived and exchanged for something more secure.

# REST Framework

[Django REST Framework](https://www.django-rest-framework.org/) offers a built in [`TokenAuthentication`](https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication) mechanism which creates secret tokens for every user and issues them via a built in view (`rest_framework.authtoken.views.obtain_auth_token`).

Upon successful authentication, that view returns a JSON response with token we can send via an HTTP header:

```
{ 'token' : '9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b' }
```

This method is fine, but again in a browser context we're stuck with the same dilemma: **where do we store this token?**

If only Django gave us a secure token stored via cookies that we could use on every request...oh wait, it does!

# Sessions to the Rescue!

Django's contains a [`login()`](https://docs.djangoproject.com/en/3.1/topics/auth/default/#how-to-log-a-user-in) method which conveniently issues a `sessionid` cookie in the HTTP response.

REST Framework provides a [`SessionAuthentication`](https://www.django-rest-framework.org/api-guide/authentication/#sessionauthentication) class which can already use this cookie to autheniticate all requests.

This sound like exactly what we need but how exactly would it work?

## How Would This Work?

If you're impatent by now, feel free to view the full [sample Django + React app](https://github.com/msukmanowsky/django-spa-auth).

First, we'll want to subclass REST Framework's `BasicAuthentication` to [prevent a browser from presenting a login popup if a user provides invalid credentials](https://stackoverflow.com/questions/9859627/how-to-prevent-browser-to-invoke-basic-auth-popup-and-handle-401-error-using-jqu?lq=1).

`gist:msukmanowsky/586f62afb506f14f486c9601bca54445`

Next, we'll wire up a few views that let us `login` and `logout` (I've also included a `me` view which lets us check if a user is authenticated):

`gist:msukmanowsky/11e667954161b1d256667d4a4d977012`

Assuming these are connected via URLs like `/api/auth/login` and `/api/auth/logout`, React can easily make use of these to [login](https://github.com/msukmanowsky/django-spa-auth/blob/master/webapp/src/App.tsx#L17-L33) and [logout](https://github.com/msukmanowsky/django-spa-auth/blob/master/webapp/src/App.tsx#L34-L54) users.

## The Perks of Sessions

When it comes to security, old and boring technologies are great because it often means tons of time has passed to find and address vulnerabilities.

If we set a few more settings in Django, we'll enable additional security:

```
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SECURE = True
```

Session cookies and those settings above buy you a few things:

1. **Secure authentication for "free"**: when your `/api/auth/login` route responds `200 OK`, you just redirect a user and you're on your way. No auth credentials to store within `localStorage` or `sessionStorage`. Your browser takes care of sending the `sessionid` cookie on all requests to your API.

2. **No credential snooping**: The `HTTPONLY` flag tells browsers to not let JavaScript code read this cookie so no concerns about third-party code sniffing session credentials. The `SECURE` flag ensures the cookie is only sent over an SSL connection (HTTPS) so no risk of man-in-the-middle attacks.

3. **Implicit Token Invalidation**: By default, Django's session cookie will expire 2 weeks after inactivity but you can configure that with `SESSION_COOKIE_AGE`. So you get an automatic "logout after 2 weeks of inactivity". You get this automatically without running a script to clean up expired tokens.

4. **Explicit Token Invalidation**: You can logout users with the `/api/auth/logout` endpoint where Django will call `logout()` and clear the session cookie.

Of course, if you want to check if a user is authenticated or not, you could always add something like a `/me` endpoint to your API which you'd check at app startup. If that endpoint returns `200 OK`, you're all set, if it returns `401 Unauthorized`, you know you have to redirect to a login page.

## But what about Mobile Apps?

I'm not an expert here, but to the best of my knowledge, both iOS and Android support secure cookie storage and if a platform doesn't, you can always read the `sessionid` cookie and store it somewhere securely.

# Conclusion

Like a lot of developers, I went down a rabbit hole when it came to how to handle authentication when supporting a single page app. It turns out, the tried and tested methods not only work just fine here, but they have significant security advantages over the new kids on the block.

Hope this helps and if you have questions or comments, hit me up on [twitter](https://twitter.com/msukmanowsky)!
