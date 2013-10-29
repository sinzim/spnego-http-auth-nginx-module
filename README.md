Warning: This is a hacked version of the module and should not be used by
anyone lightly. For the real deal go to
https://github.com/stnoonan/spnego-http-auth-nginx-module

Nginx module for HTTP SPNEGO auth
=================================

This module implements adds [SPNEGO](http://tools.ietf.org/html/rfc4178)
support to nginx(http://nginx.org).  It currently supports only Kerberos
authentication via [GSSAPI](http://en.wikipedia.org/wiki/GSSAPI)


Prerequisites
-------------

Authentication has been tested with (at least) the following:

* Nginx 1.2.6, 1.5.x
* Internet Explorer 8.0.7600.16385
* Firefox 10.0.6, 22.0
* Chrome 20.0.1132.57, 28.0.1500.63
* Curl 7.19.5 (GSS-Negotiate), 7.27.0 (SPNEGO/fbopenssl)

The underlying kerberos library used for these tests was MIT KRB5 v1.8.


Installation
------------

1. Download [nginx source](http://www.nginx.org/en/download.html)
1. Extract to a directory
1. Clone this module into the directory
1. Follow the [nginx install documentation](http://nginx.org/en/docs/install.html)
and pass an `--add-module` option to nginx configure:

    ./configure --add-module=spnego-http-auth-nginx-module


Configuration reference
-----------------------

You can configure GSS authentication on a per-location and/or a global basis:

These options are required.
* `auth_gss`: on/off, for ease of unsecuring while leaving other options in
  the config file
* `auth_gss_keytab`: absolute path-name to keytab file containing service
  credentials

These options should ONLY be specified if you have a keytab containing
privileged principals.  In nearly all cases, you should not put these
in the configuration file, as `gss_accept_sec_context` will do the right
thing.
* `auth_gss_realm`: Kerberos realm name
* `auth_gss_service_name`: service principal name to use when acquiring
  credentials.

If you would like to authorize only a specific set of users, you can use the
`auth_gss_authorized_principal` directive.  The configuration syntax supports
multiple entries, one per line.

    auth_gss_authorized_principal <username>@<realm>
    auth_gss_authorized_principal <username2>@<realm>

If you are using SPNEGO without SSL, it is recommended you disable basic
authentication fallback, as the password would be sent in plaintext.  This
is done by setting `auth_gss_allow_basic_fallback` in the config file.

    auth_gss_allow_basic_fallback off


Troubleshooting
---------------

###
Check the logs.  If you see a mention of NTLM, your client is attempting to
connect using [NTLMSSP](http://en.wikipedia.org/wiki/NTLMSSP), which is
unsupported and insecure.

### Verify that you have an HTTP principal in your keytab ###

#### MIT Kerberos utilities ####

    $ KRB5_KTNAME=FILE:<path to your keytab> klist -k

or

    $ ktutil
    ktutil: read_kt <path to your keytab>
    ktutil: list

#### Heimdal Kerberos utilities ####

    $ ktutil -k <path to your keytab> list

### Obtain an HTTP principal

If you find that you do not have the HTTP service principal,
are running in an Active Directory environment,
and are bound to the domain such that Samba tools work properly

    $ env KRB5_KTNAME=FILE:<path to your keytab> net ads -P keytab add HTTP

If you are running in a different kerberos environment, you can likely run

    $ env KRB5_KTNAME=FILE:<path to your keytab> krb5_keytab HTTP


Debugging
---------

The module prints all sort of debugging information if nginx is compiled with
the `--with-debug` option, and the `error_log` directive has a `debug` level.


NTLM
----

Note that the module does not support [NTLMSSP](http://en.wikipedia.org/wiki/NTLMSSP)
in Negotiate. NTLM, both v1 and v2, is an exploitable protocol and should be avoided
where possible.


