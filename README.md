# Fandjango

## About

Fandjango is a Python library that makes it easier to make Facebook canvas
applications powered by Django.

## Requirements

Django

## Usage

Fandjango populates the request object with data derived from the signed
request Facebook sends canvas applications (see the [documentation on signed requests][1]).

[1]: http://developers.facebook.com/docs/authentication/canvas

If the client has authorized your application, `request.facebook` is a dictionary
with these keys:

* `user_id` - An integer describing the user's id on Facebook.
* `oauth_token` - A string describing the OAuth access token.
* `issued_at` - A datetime object describing when the OAuth token was issued.
* `expires_at` - A datetime object describing when the OAuth access token expires, or false if it doesn't.

If the user has not authorized your application, `request.facebook` is `False`.

You can require a user to authorize your application before accessing a view with the
`facebook_authorization_required` decorator.

    from fandjango.decorators import facebook_authorization_required
    
    @facebook_authorization_required
    def foo(request, *args, **kwargs):
      [...]
      
This will redirect the request to the Facebook authorization dialog, which will in
turn redirect back to the original URI.

You can also redirect the request in a control flow of your own by using the
`redirect_to_facebook_authorization` function.

    from fandjango.utils import redirect_to_facebook_authorization
    
    def foo(request, *args, **kwargs):
      if not request.facebook:
        return redirect_to_facebook_authorization(redirect_uri='http://www.example.org/')
        
## Installation

* `pip install git+http://github.com/jgorset/fandjango`
* Add `fandjango.middleware.FacebookCanvasMiddleware` to your middleware classes.

## Configuration

Fandjango requires some constants to be set in your settings.py file:

* `FACEBOOK_APPLICATION_ID` - Your Facebook application's ID.
* `FACEBOOK_APPLICATION_SECRET_KEY` - Your Facebook application's secret key.
* `FACEBOOK_APPLICATION_URL` - Your application's canvas URI (ex. http://apps.facebook.com/my_application)
* `FACEBOOK_APPLICATION_INITIAL_PERMISSIONS` - An array of [extended permissions][2] to request upon authorizing the application (optional).

[2]: http://developers.facebook.com/docs/authentication/permissions

## Frequently asked questions

**Q:** *Do I need to pass the signed request around?*

**A:** No. Fandjango caches the latest signed request in a cookie so you don't have to worry about it.

**Q:** *Why does Fandjango set a new header called "P3P"?*

**A:** P3P (or *Platform for Privacy Preferences*) is a W3 standard that enables websites to express
their privacy practices in a standard format that can be retrieved automatically and interpreted easily
by user agents. While this is largely ignored by most browsers, Internet Explorer will ignore cookies
set by third-party websites (ie. websites loaded in iframes) unless it specifies some P3P policies.

You can read more about P3P at [w3.org][3].

[3]: http://www.w3.org/TR/P3P/

**Q:** *What happens when the OAuth token expires?*

**A:** Fandjango will automatically renew the signed request once the OAuth token
expires. It does this by hijacking the request and redirecting the client to Facebook, which
in turn redirects the client back to the URI it was originally retrieving with a new signed
request attached.