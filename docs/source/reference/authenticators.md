# Authenticators

The [Authenticator][] is the mechanism for authorizing users to use the
Hub and single user notebook servers.

## The default PAM Authenticator

JupyterHub ships only with the default [PAM][]-based Authenticator,
for logging in with local user accounts via a username and password.

## The OAuthenticator

Some login mechanisms, such as [OAuth][], don't map onto username and
password authentication, and instead use tokens. When using these
mechanisms, you can override the login handlers.

You can see an example implementation of an Authenticator that uses
[GitHub OAuth][] at [OAuthenticator][].

JupyterHub's [OAuthenticator][] currently supports the following
popular services:

- Auth0
- Bitbucket
- CILogon
- GitHub
- GitLab
- Globus
- Google
- MediaWiki
- Okpy
- OpenShift

A generic implementation, which you can use for OAuth authentication
with any provider, is also available.

## Additional Authenticators

- ldapauthenticator for LDAP
- tmpauthenticator for temporary accounts

## Technical Overview of Authentication

### How the Base Authenticator works

The base authenticator uses simple username and password authentication.

The base Authenticator has one central method:

#### Authenticator.authenticate method

    Authenticator.authenticate(handler, data)

This method is passed the Tornado `RequestHandler` and the `POST data`
from JupyterHub's login form. Unless the login form has been customized,
`data` will have two keys:

- `username`
- `password`

The `authenticate` method's job is simple:

- return the username (non-empty str) of the authenticated user if
  authentication is successful
- return `None` otherwise

Writing an Authenticator that looks up passwords in a dictionary
requires only overriding this one method:

```python
from tornado import gen
from IPython.utils.traitlets import Dict
from jupyterhub.auth import Authenticator

class DictionaryAuthenticator(Authenticator):

    passwords = Dict(config=True,
        help="""dict of username:password for authentication"""
    )

    @gen.coroutine
    def authenticate(self, handler, data):
        if self.passwords.get(data['username']) == data['password']:
            return data['username']
```

#### Normalize usernames

Since the Authenticator and Spawner both use the same username,
sometimes you want to transform the name coming from the authentication service
(e.g. turning email addresses into local system usernames) before adding them to the Hub service.
Authenticators can define `normalize_username`, which takes a username.
The default normalization is to cast names to lowercase

For simple mappings, a configurable dict `Authenticator.username_map` is used to turn one name into another:

```python
c.Authenticator.username_map  = {
  'service-name': 'localname'
}
```

#### Validate usernames

In most cases, there is a very limited set of acceptable usernames.
Authenticators can define `validate_username(username)`,
which should return True for a valid username and False for an invalid one.
The primary effect this has is improving error messages during user creation.

The default behavior is to use configurable `Authenticator.username_pattern`,
which is a regular expression string for validation.

To only allow usernames that start with 'w':

```python
c.Authenticator.username_pattern = r'w.*'
```

### How to write a custom authenticator

You can use custom Authenticator subclasses to enable authentication
via other mechanisms. One such example is using [GitHub OAuth][].

Because the username is passed from the Authenticator to the Spawner,
a custom Authenticator and Spawner are often used together.

See a list of custom Authenticators [on the wiki](https://github.com/jupyterhub/jupyterhub/wiki/Authenticators).

If you are interested in writing a custom authenticator, you can read
[this tutorial](http://jupyterhub-tutorial.readthedocs.io/en/latest/authenticators.html).


## JupyterHub as an OAuth provider

Beginning with version 0.8, JupyterHub is an OAuth provider.


[Authenticator]: https://github.com/jupyterhub/jupyterhub/blob/master/jupyterhub/auth.py
[PAM]: https://en.wikipedia.org/wiki/Pluggable_authentication_module
[OAuth]: https://en.wikipedia.org/wiki/OAuth
[GitHub OAuth]: https://developer.github.com/v3/oauth/
[OAuthenticator]: https://github.com/jupyterhub/oauthenticator