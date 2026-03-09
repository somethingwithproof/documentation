# Basic Authentication

## Overview

Basic Auth settings leverage both Apache's and NGINX Authentication modules
to grant a user access to the Cacti Website.  The various configuration steps
for these modules will not be covered in the Cacti documentation.  Therefore,
if you wish to use this authentication method, it is recommended that
you test your configuration on a development server before putting this
method into production.

> **NOTE**: Before enabling the **Basic Authentication Method**, you must
> always verify that you are prompted for a password before getting the
> normal Cacti login prompt, and that you have setup a new user account
> with Admin privileges that matches your login account.  Otherwise
> when you enable Basic Authentication, you can lock yourself out of the
> Cacti Website.

## Special Users

The image below shows the settings for **Web Basic Authentication** which
includes the `Special Users` sub-section.

![Basic Auth Settings](images/settings-auth-basic.png)

Those settings include:

- **Primary Admin** - This is the Cacti primary administrative account
  this account will receive Cacti notifications and can not be deleted.
- **Guest Account** - This is a special account that allows users to
  access certain pages without being prompted for a login and password.
  Setting to `None`, disables this feature.
- **User Template** - All new users who login for the first time will
  have their initial settings based upon this **User Template**.
- **Basic Auth Login Failure Message** - This textbox can be customized
  in order to provide a useful message to users who have no access to
  Cacti via Basic Authentication and how to resolve the issue.
- **Basic Auth Mapfile** - If your basic users do not match OS users
  you can force a basic account to login with an alternate account.

## Basic Auth Mapfile

This settings is important for plugins that grant certain features
to accounts, but those accounts do not match the login account.  For
example, your basic account maybe: joe.schmoe@company.com, and your
UNIX login account may be `jschmoe`, which grants you certain
plugin permissions, like controlling your own workload.

The mapfile is a file in CSV format, with the first column
being the Basic account (aka joe.schmoe@company.com), and the second
column being the login account (jschmoe).  It is up-to the Cacti
administrator to manage and update this file per their local
site practices.

## Using Web Basic Auth for SSO (Header-Based Authentication)

Cacti's Web Basic Authentication mode reads the authenticated username
from the web server and creates or maps a local Cacti account. This makes
it suitable for header-based SSO setups where a reverse proxy or Apache
module authenticates the user before the request reaches Cacti.

### How it works

When Web Basic Auth is enabled, Cacti reads `$_SERVER['PHP_AUTH_USER']`
(populated by Apache mod_auth) or, if that is empty, falls back to
`$_SERVER['REMOTE_USER']`. Some SSO setups pass the authenticated user
as an HTTP header instead (e.g. `X-Remote-User`); for those, configure
your reverse proxy to set the `REMOTE_USER` CGI variable rather than an
arbitrary header, or use Apache's `RequestHeader` directive to rewrite
the header into one Cacti can read.

### Apache — Kerberos/GSSAPI (Windows AD single sign-on)

Install `mod_auth_gssapi` and configure the Cacti virtualhost or directory:

```apache
<Location /cacti>
    AuthType GSSAPI
    AuthName "Cacti (Kerberos)"
    GssapiCredStore keytab:/etc/apache2/cacti.keytab
    Require valid-user
</Location>
```

On successful authentication, Apache sets `REMOTE_USER` to the
Kerberos principal (e.g. `jsmith@EXAMPLE.COM`). Use a Basic Auth Mapfile
(see above) to strip the realm if your Cacti usernames do not include it.

### Apache — Reverse proxy header passthrough

If a reverse proxy (nginx, HAProxy, Shibboleth SP) authenticates users
and forwards the identity as an HTTP header, use `mod_rewrite` to
convert it to `REMOTE_USER`:

```apache
<Location /cacti>
    RewriteEngine On
    RewriteCond %{HTTP:X-Remote-User} ^(.+)$
    RewriteRule .* - [E=REMOTE_USER:%1]
    Require all granted
</Location>
```

> **Security**: Never trust `X-Remote-User` (or any user-controlled
> header) from untrusted clients. Restrict access to the Cacti
> `Location` block to the proxy's IP, or strip the header at the
> network boundary before it reaches Apache.

### Per-user access control

Web Basic Auth grants access to any user your web server authenticates.
To restrict which authenticated users can reach Cacti, use one of:

- **Apache `Require`**: `Require ldap-group cn=cacti-users,ou=groups,dc=example,dc=com`
  (with `mod_authnz_ldap` or `mod_authnz_sspi`)
- **User Template set to 'No User'**: In Cacti Authentication settings,
  set **User Template** to `No User`. Only users with a pre-existing local
  account whose **Realm** is set to `Web Basic Authentication` will be
  permitted. All others are rejected even if the web server authenticates them.

---
Copyright (c) 2004-2026 The Cacti Group
