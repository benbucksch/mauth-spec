---
title: "mAuth - OAuth2 profile for mail apps and other public clients"
abbrev: "mAuth"
category: info

docname: draft-mauth-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - OAuth2
 - IMAP
 - SMTP
 - POP3
 - XMPP
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "benbucksch/mauth-spec"
  latest: "https://benbucksch.github.io/mauth-spec/draft-mauth.html"

author:
 -
    fullname: "Ben Bucksch"
    organization: Beonex
    email: "ben.bucksch@beonex.com"

normative:

informative:


--- abstract

This document creates a specific OAuth2 profile
that is suitable for mail, chat, calendar and similar clients.

It defines specific parameters of OAuth2, to allow email clients to reliably authenticate using OAuth2 on any mail provider.

--- middle

# Introduction

Mail clients today use mostly username and password to log in to IMAP. OAuth2 can only be used at the biggest providers, but causes major problems for mail clients in its current application.

This document creates a specific OAuth2 profile
that is suitable for mail, chat, calendar and similar clients.

It defines specific parameters of OAuth2, with the purpose
of making OAuth2 as reliable for client and for end users
as password login is today.

It also avoids provider-specific configuration and adjustments. This in turn would allow email clients to authenticate using OAuth2 on any mail provider, without having to specifically register with or test each and every mail provider in the world.

# Scope

To avoid that the client needs to know the provider-specific version of scope names, the server MUST accept the following literal scope names (RFC 6749 3.3), for the following purposes:

| Scope       | Resource protocol or standard |
|--------------|-----------------------|
| IMAP      | RFC 9051 |
| SMTP     | RFC 5321 |
| POP3      | RFC 1939 |
| CalDAV   | RFC 4791 |
| CardDAV | RFC 6352 |
| WebDAV  | RFC 4918 |
| XMPP | RFC 6120 |
| matrix.org/Matrix  | Matrix chat client/server API |
| microsoft.com/EWS  | Microsoft Exchange Web Services |
| microsoft.com/ActiveSync  | Microsoft ActiveSync |

For all RFC references, previous versions and updates of these standards are also included.

If a client requests one or multiple of these scopes, the token issued by the server MUST be valid for this user on the resource servers of the provider which implement the corresponding protocols.

If the provider does not have an resource server which
implements this protocol, the server MUST ignore the unsupported scope and issue the token for the supported scopes. The authorization server response MUST include only the supported scopes. It MAY reject the token request with error `invalid_scope`, if *all* requested scopes are unsupported.
(See RFC 6749 3.3 Paragraph 3)

# Client ID

## Rationale

OAuth2 was created with server-to-server relations in mind, where a contractual relationship between both
parties exist. RFC 6749 states:
'Client ID ... may require a contractual relationship...'
(TODO find quote)

This concept is not compatible with the concept of mail clients. A mail client cannot realistically register with all mail providers in the world.

Last but not least, the client ID currently allows a mail
provider to force a contract on a mail client. This is
directly contrary to a free Internet and IETF values.

Furthermore, given that mail providers typically also offer mail clients and mail clients are therefore almost
always competitors to the mail provider, a mail provider
blocking clients or allowing the clients only under conditions, is inherently anti-competitive.

## Definition Client ID

The server MUST accept the following Client ID
(RFC 6749 2.2):
`open`

The client SHALL use this Client ID.

The server MUST NOT require other authentication or authorization for the client software.

# Client software name and version

The client MUST use the following mechanisms to
identify itself, as appropriate for the protocol used:

| Protocol       | Identification method | Standard |
|--------------|-----------------------|-------|
| OAuth2 | `User-Agent` HTTP header | RFC 2616 14.43 |
| HTTPS | `User-Agent` HTTP header | RFC 2616 14.43 |
| IMAP      | `ID` command | RFC 2971 |

All all cases, the client MUST use as primary
name component a string reflecting the commonly
known name of the end user application, and at least the major version. The client MAY include as secondary name component the software library that implements the protocol.

In the case of HTTP user agent strings, the order is reversed, so a typical user agent string should be,
for mail app "ACME Mail" using "Best Effort IMAP" library:

`User-Agent: Best-Effort-IMAP/2.3 ACME-Mail/4.5`

The OAuth2 authorization server MUST NOT reject or disadvantage clients that send non-browser User-Agent strings. The client SHALL NOT send generic browser user agent strings, unless the authorization server is known to disadvantage a client without such user agent string.

# Username in authentication request

In most cases, the client would like to authenticate to a specific account or email address which is known before the authentication is started. However, RFC 6749 does not specify how the client informs the authorization server which username is intended to log in. Various proprietary provider-specific mechanisms exist, e.g. `login_hint` at Office365, but there is no interoperable standard.

In the Authorization Request, an additional URL query parameter `username` MAY be passed by the client, with the email address or account ID of the account that the client expects to log in. If given, the server MUST skip the interactive username input, SHALL show the username on the login page, and SHALL NOT allow the user to log in with another account.

# Error codes

In the Authorization Response Error Response (RFC 6749 4.1.2.1.), the following more specific `error` codes MUST be used, if their case applies:

[//]: [or: `access_denied` MUST be accompanied with more specific error codes, returned as `error_code`:]

| error code       | Problem |
|--------------|-----------------------|-------|
| auth_failed | The user made a login attempt, but the password or username or second factor was wrong. |
| blocked | The login credentials were valid, but the user account was blocked. The error_description MUST give a detailed cause for the block, and SHALL explain a way for the user to remedy the problem and unblock the account. |
| unknown_user | The user doesn't exist, as far as this server is aware. |
| user_not_here | The user account is known to exist, but must use another OAuth2 server. |
| no_oauth | The user account exists, but is not allowed to use OAuth2. |


# Refresh token

The authorization server access token response MUST contain a working refresh token together with the access token (RFC 6749 5.1. and 6.).

When the client requests a new access token using a valid refresh token, the authorization server MUST return a new refresh token with a renewed expiry time.

The expiry time of the refresh token MUST be long enough for typical uses of an email client that checks email in the background, i.e. at least 24 hours, but preferably 1 year.

The `expires_in` field MUST be returned in the response (in RFC 6749 5.1., it is RECOMMENDED).

## Revokation

The tokens MAY be revoked or invalidated by the authorization server at any time, if there is a concrete reason to believe that this specific account has been compromised.

## Rationale

This avoids that users get logged out of their email silenty and do not get new mail notifications anymore.

It also avoids that users have to re-log in constantly,
which is known to be detrimental to security, due to passwords being written down or re-using the same password on multiple services to remember them.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

## Data protection and privacy

The client MUST act directly on behalf of the user
who is authenticating using OAuth2.

The client MUST keep the data obtained via the obtained access token from the protected resources on the user's computers and MUST not route the data through
servers not under control of the resource server owner
(including avoiding servers of the application vendor),
unless specifically and explicitly requested by the user.
(TODO rephrase with better English)

## Abuse

The server MAY reject access, if the specific connection
or IP address acted on many different users' behalf or has issued an excessive amount of requests that don't
reflect an interactive client usage.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}
