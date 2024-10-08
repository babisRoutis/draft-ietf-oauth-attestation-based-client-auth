---
title: "OAuth 2.0 Attestation-Based Client Authentication"
category: info

docname: draft-ietf-oauth-attestation-based-client-auth-latest
submissiontype: IETF  # also: "IETF", "IAB", or "IRTF"
number:
date:
v: 3
venue:
  group: "oauth-wg/draft-ietf-oauth-attestation-based-client-auth"
  latest: "https://oauth-wg.github.io/draft-ietf-oauth-attestation-based-client-auth/draft-ietf-oauth-attestation-based-client-auth.html"

author:
 -
    fullname: Tobias Looker
    organization: MATTR
    email: tobias.looker@mattr.global
 -
    fullname: Paul Bastian
    email: paul.bastian@bdr.de
 -
    fullname: Christian Bormann
    organization: Robert Bosch GmbH
    email: chris.bormann@gmx.de


normative:
  RFC3986: RFC3986
  RFC7800: RFC7800
  RFC7591: RFC7591
  RFC7519: RFC7519
  RFC8414: RFC8414
  RFC8725: RFC8725
  RFC9110: RFC9110
  RFC9112: RFC9112
  IANA.HTTP.Fields:
    author:
      org: "IANA"
    title: "Hypertext Transfer Protocol (HTTP) Field Name Registry"
    target: "https://www.iana.org/assignments/http-fields/http-fields.xhtml"
informative:
  RFC6749: RFC6749
  ARF:
  	title: "The European Digital Identity Wallet Architecture and Reference Framework"


--- abstract

This specification defines an extension to the OAuth 2 protocol as defined in {{RFC6749}} which enables a Client Instance to include a key-bound attestation in interactions with an Authorization Server or a Resource Server. This new method enables Client Instances involved in a client deployment that is traditionally viewed as a public client, to be able to utilize this key-bound attestation to authenticate.

--- middle

# Introduction

The following diagram depicts the overall architecture and protocol flow.

~~~ ascii-art
                    (3)
                 +-------+
                 |       |
                 |      \ /
             +---------------+
             |               |
             |    Client     |
             |    Backend    |
             |               |
             +---------------+
                / \      |
            (2)  |       |  (4)
                 |      \ /
             +---------------+           +---------------+
      +----->|               |           |               |
  (1) |      |    Client     |    (6)    | Authorization |
      |      |   Instance    |<--------->|    Server     |
      +------|               |           |               |
             +---------------+           +---------------+
                / \      |
                 |       |
                 +-------+
                    (5)

~~~

The following steps describe this OAuth flow:

(1) The Client Instance generates a key (Client Instance Key) and optional further attestations (that are out of scope) to prove its authenticity to the Client Backend.

(2) The Client Instance sends this data to the Client Backend in request for a Client Attestation JWT.

(3) The Client Backend validates the Client Instance Key and optional further data. It generates a signed Client Attestation JWT that is cryptographically bound to the Client Instance Key generated by the Client. Therefore, the attestation is bound to this particular Client Instance.

(4) The Client Backend responds to the Client Instance by sending the Client Attestation JWT.

(5) The Client Instance generates a Proof of Possession (PoP) with the Client Instance Key.

(6) The Client Instance sends both the Client Attestation JWT and the Client Attestation PoP JWT to the authorization server, e.g. within a token request. The authorization server validates the Client Attestation and thus authenticates the Client Instance.

Please note that the protocol details for steps (2) and (4), particularly how the Client Instance authenticates to the client Backend, are beyond the scope of this specification. Furthermore, this specification is designed to be flexible and can be implemented even in scenarios where the client does not have a backend server. In such cases, each Client Instance is responsible for performing the functions typically handled by the backend on its own.

This approach acknowledges the evolving landscape of OAuth 2 deployments, where the ability for public clients to authenticate securely and reliably has become increasingly important.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

Client Attestation JWT:
:  A JSON Web Token (JWT) generated by the client backend which is bound to a key managed by a Client Instance which can then be used by the instance for client authentication.

Client Attestation Proof of Possession (PoP) JWT:
:  A Proof of Possession generated by the Client Instance using the key that the Client Attestation JWT is bound to.

Client Instance:
: A deployed instance of a piece of client software.

Client Instance Key:
:  A cryptographic asymmetric key pair that is generated by the Client Instance where the public key of the key pair is provided to the client backend. This public key is then encapsulated within the Client Attestation JWT and is utilized to sign the Client Attestation Proof of Possession.

# Client Attestation

This draft introduces the concept of client attestations to the OAuth 2 protocol, using two JWTs: a Client Attestation and a Client Attestation Proof of Possession (PoP). These JWTs are transmitted via HTTP headers in an HTTP request from a Client Instance to an Authorization Server or Resource Server. The primary purpose of these headers is to authenticate the Client Instance.

## Client Attestation HTTP Headers {#headers}

A Client Attestation JWT and Client Attestation PoP JWT is included in an HTTP request using the following request header fields.

OAuth-Client-Attestation:
: A JWT that conforms to the structure and syntax as defined in [](#client-attestation-jwt)

OAuth-Client-Attestation-PoP:
: A JWT that adheres to the structure and syntax as defined in [](#client-attestation-pop-jwt)

The following is an example of the OAuth-Client-Attestation header.

~~~
OAuth-Client-Attestation: eyJhbGciOiAiRVMyNTYiLCJraWQiOiAiMTEifQ.eyJ\
pc3MiOiJodHRwczovL2NsaWVudC5leGFtcGxlLmNvbSIsInN1YiI6Imh0dHBzOi8vY2x\
pZW50LmV4YW1wbGUuY29tIiwibmJmIjoxMzAwODE1NzgwLCJleHAiOjEzMDA4MTkzODA\
sImNuZiI6eyJqd2siOnsia3R5IjoiRUMiLCJ1c2UiOiJzaWciLCJjcnYiOiJQLTI1NiI\
sIngiOiIxOHdITGVJZ1c5d1ZONlZEMVR4Z3BxeTJMc3pZa01mNko4bmpWQWlidmhNIiw\
ieSI6Ii1WNGRTNFVhTE1nUF80Zlk0ajhpcjdjbDFUWGxGZEFnY3g1NW83VGtjU0EifX1\
9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
~~~

The following is an example of the OAuth-Client-Attestation-PoP header.

~~~
OAuth-Client-Attestation-PoP: eyJhbGciOiJFUzI1NiJ9.eyJpc3MiOiJodHRwc\
zovL2NsaWVudC5leGFtcGxlLmNvbSIsImF1ZCI6Imh0dHBzOi8vYXMuZXhhbXBsZS5jb\
20iLCJuYmYiOjEzMDA4MTU3ODAsImV4cCI6MTMwMDgxOTM4MH0.coB_mtdXwvi9RxSMz\
bIey8GVVQLv9qQrBUqmc1qj9Bs
~~~

Note that per {{RFC9110}} header field names are case-insensitive; so OAUTH-CLIENT-ATTESTATION, oauth-client-attestation, etc., are all valid and equivalent
header field names. Case is significant in the header field value, however.

The OAuth-Client-Attestation and OAuth-Client-Attestation-PoP HTTP header field values uses the token68 syntax defined in Section 11.2 of {{RFC9110}} (repeated below for ease of reference).

~~~
OAuth-Client-Attestation       = token68
OAuth-Client-Attestation-PoP   = token68
token68                        = 1*( ALPHA / DIGIT / "-" / "." /
                                     "_" / "~" / "+" / "/" ) *"="
~~~

It is RECOMMENDED that the authorization server validate the Client Attestation JWT prior to validating the Client Attestation PoP.

## Client Attestation JWT {#client-attestation-jwt}

The following rules apply to validating the Client Attestation JWT. Application of additional restrictions and policy are at the discretion of the Authorization Server.

1. The JWT MUST contain an "iss" (issuer) claim that contains a unique identifier for the entity that issued the JWT. In the absence of an application profile specifying otherwise, compliant applications MUST compare issuer values using the Simple String Comparison method defined in Section 6.2.1 of {{RFC3986}}.

2. The JWT MUST contain a "sub" (subject) claim with a value corresponding to the "client_id" of the OAuth client.

3. The JWT MUST contain an "exp" (expiration time) claim that limits the time window during which the JWT can be used.  The authorization server MUST reject any JWT with an expiration time that has passed, subject to allowable clock skew between systems.

4. The JWT MUST contain an "cnf" claim conforming {{RFC7800}} that conveys the key to be used for producing the client attestation pop for client authentication with an authorization server. The key MUST be expressed using the "jwk" representation.

5. The JWT MAY contain an "nbf" (not before) claim that identifies the time before which the token MUST NOT be accepted for processing.

6. The JWT MAY contain an "iat" (issued at) claim that identifies the time at which the JWT was issued.

7. The JWT MAY contain other claims.

8. The JWT MUST be digitally signed using an asymmetric cryptographic algorithm. The authorization server MUST reject the JWT if it is using a Message Authentication Code (MAC) based algorithm. The authorization server MUST reject JWTs with an invalid signature.

9. The authorization server MUST reject a JWT that is not valid in all other respects per "JSON Web Token (JWT)" {{RFC7519}}.

The following example is the decoded header and payload of a JWT meeting the processing rules as defined above.

~~~
{
  "alg": "ES256",
  "kid": "11"
}
.
{
  "iss": "https://client.example.com",
  "sub": "https://client.example.com",
  "nbf":1300815780,
  "exp":1300819380,
  "cnf": {
    "jwk": {
      "kty": "EC",
      "use": "sig",
      "crv": "P-256",
      "x": "18wHLeIgW9wVN6VD1Txgpqy2LszYkMf6J8njVAibvhM",
      "y": "-V4dS4UaLMgP_4fY4j8ir7cl1TXlFdAgcx55o7TkcSA"
    }
  }
}
~~~

## Client Attestation PoP JWT {#client-attestation-pop-jwt}

The following rules apply to validating the Client Attestation PoP JWT. Application of additional restrictions and policy are at the discretion of the Authorization Server.

1. The JWT MUST contain an "iss" (issuer) claim with a value corresponding to the "client_id" of the OAuth client.

2. The JWT MUST contain an "exp" (expiration time) claim that limits the time window during which the JWT can be used.  The authorization server MUST reject any JWT with an expiration time that has passed, subject to allowable clock skew between systems.  Note that the authorization server may reject JWTs with an "exp" claim value that is unreasonably far in the future.

3. The JWT MUST contain a "jti" (JWT ID) claim that provides a unique identifier for the token.  The authorization server MAY ensure that JWTs are not replayed by maintaining the set of used "jti" values for the length of time for which the JWT would be considered valid based on the applicable "exp" instant.

4. The JWT MUST contain an "aud" (audience) claim containing a value that identifies the authorization server as an intended audience. The {{RFC8414}} issuer identifier URL of the authorization server MUST be used as a value for an "aud" element to identify the authorization server as the intended audience of the JWT.

5. The JWT MAY contain an "nonce" claim containing a String value that is provided by the authorization server to associate the Client Attestation PoP JWT with a particular transaction and prevent replay attacks.

6. The JWT MAY contain an "nbf" (not before) claim that identifies the time before which the token MUST NOT be accepted for processing.

7. The JWT MAY contain an "iat" (issued at) claim that identifies the time at which the JWT was issued.  Note that the authorization server may reject JWTs with an "iat" claim value that is unreasonably far in the past.

8. The JWT MAY contain other claims.

9. The JWT MUST be digitally signed using an asymmetric cryptographic algorithm. The authorization server MUST reject the JWT if it is using a Message Authentication Code (MAC) based algorithm. The authorization server MUST reject JWTs with an invalid signature.

10. The public key used to verify the JWT MUST be the key located in the "cnf" claim of the corresponding Client Attestation JWT.

11. The Authorization Server MUST reject a JWT that is not valid in all other respects per "JSON Web Token (JWT)" {{RFC7519}}.

The following example is the decoded header and payload of a JWT meeting the processing rules as defined above.

~~~
{
  "alg": "ES256"
}
.
{
  "iss": "https://client.example.com",
  "aud": "https://as.example.com",
  "nbf":1300815780,
  "exp":1300819380,
  "jti": "d25d00ab-552b-46fc-ae19-98f440f25064"
}
~~~

## Checking HTTP requests feature client attestations {#checking-http-requests-with-client-attestations}

To validate an HTTP request which contains the client attestation headers, the receiving server MUST ensure the following with regard to a received HTTP request:

1. There is precisely one OAuth-Client-Attestation HTTP request header field, where its value is a single well-formed JWT conforming to the syntax outlined in []{client-attestation-jwt}.
2. There is precisely one OAuth-Client-Attestation-PoP HTTP request header field, where its value is a single well-formed JWT conforming to the syntax outlined in []{client-attestation-pop-jwt}.
3. The signature of the Client Attestation PoP JWT obtained from the OAuth-Client-Attestation-PoP HTTP header verifies with the Client Instance Key contained in the `cnf` claim of the Client Attestation JWT obtained from the OAuth-Client-Attestation HTTP header.

# Client Attestation at the Token Endpoint

While usage of the the client attestation mechanism defined by this draft can be used in a variety of different HTTP requests to different endpoints, usage within the token request as defined by {{RFC6749}} has particular additional considerations outlined below.

The Authorization Server MUST perform all of the checks outlined in [](#checking-http-requests-with-client-attestations) for a received access token request which is making use of the client attestation mechanism as defined by this draft.

The following example demonstrates usage of the client attestation mechanism in an access token request (with extra line breaks for display purposes only):

~~~
POST /token HTTP/1.1
Host: as.example.com
Content-Type: application/x-www-form-urlencoded
OAuth-Client-Attestation: eyJhbGciOiAiRVMyNTYiLCJraWQiOiAiMTEifQ.eyJ\
pc3MiOiJodHRwczovL2NsaWVudC5leGFtcGxlLmNvbSIsInN1YiI6Imh0dHBzOi8vY2x\
pZW50LmV4YW1wbGUuY29tIiwibmJmIjoxMzAwODE1NzgwLCJleHAiOjEzMDA4MTkzODA\
sImNuZiI6eyJqd2siOnsia3R5IjoiRUMiLCJ1c2UiOiJzaWciLCJjcnYiOiJQLTI1NiI\
sIngiOiIxOHdITGVJZ1c5d1ZONlZEMVR4Z3BxeTJMc3pZa01mNko4bmpWQWlidmhNIiw\
ieSI6Ii1WNGRTNFVhTE1nUF80Zlk0ajhpcjdjbDFUWGxGZEFnY3g1NW83VGtjU0EifX1\
9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
OAuth-Client-Attestation-PoP: eyJhbGciOiJFUzI1NiJ9.eyJpc3MiOiJodHRwc\
zovL2NsaWVudC5leGFtcGxlLmNvbSIsImF1ZCI6Imh0dHBzOi8vYXMuZXhhbXBsZS5jb\
20iLCJuYmYiOjEzMDA4MTU3ODAsImV4cCI6MTMwMDgxOTM4MH0.coB_mtdXwvi9RxSMz\
bIey8GVVQLv9qQrBUqmc1qj9Bs

grant_type=authorization_code&
code=n0esc3NRze7LTCu7iYzS6a5acc3f0ogp4
~~~

# Implementation Considerations

## Reuse of a Client Attestation JWT

Implementers should be aware that the design of this authentication mechanism deliberately allows for a Client Instance to re-use a single Client Attestation JWT in multiple interactions/requests with an Authorization Server, whilst producing a fresh Client Attestation PoP JWT. Client deployments should consider this when determining the validity period for issued Client Attestation JWTs as this ultimately controls how long a Client Instance can re-use a single Client Attestation JWT.

## Refresh token binding

Authorization servers issuing a refresh token in response to a token request using the client attestation mechanism as defined by this draft MUST bind the refresh token to the Client Instance, and NOT just the client as specified in section 6 {{RFC6749}}. To prove this binding, the Client Instance MUST use the client attestation mechanism when refreshing an access token. The client MUST also use the same key that was present in the "cnf" claim of the client attestation that was used when the refresh token was issued.

### Web Server Default Maximum HTTP Header Sizes

Because the Client Attestation and Client Attestation PoP are communicated using HTTP headers, implementers should consider that web servers may have a default maximum HTTP header size configured which could be too low to allow conveying a Client Attestation and or Client Attestation PoP in an HTTP request. It should be noted, that this limit is not given by the HTTP {{RFC9112}}, but instead web server implementations commonly set a default maximum size for HTTP headers. As of 2024, typical limits for modern web servers configure maximum HTTP headers as 8 kB or more as a default.
## Rotation of Client Instance Key

This specification does not provide a mechanism to rotate the Client Instance Key in the Client Attestation JWT's "cnf" claim. If the Client Instance needs to use a new Client Instance Key for any reason, then it MUST request a new Client Attestation JWT from its Client Backend.

# Privacy Considerations

## Client Instance Tracking Across Authorization Servers

Implementers should be aware that using the same client attestation across multiple authorization servers could result in correlation of the end user using the Client Instance through claim values (including the Client Instance Key in the `cnf` claim). Client deployments are therefore RECOMMENDED to use different Client Attestation JWTs with different Client Instance Keys across different authorization servers.

# Security Considerations

The guidance provided by {{RFC7519}} and {{RFC8725}} applies.

## Replay Attack Detection

The following mechanisms exist within this client authentication method in order to allow an authorization server to detect replay attacks for presented client attestation PoPs:

- The client uses "jti" (JWT ID) claims for the Client Attestation PoP JWT and the authorization server maintains a list of used (seen) "jti" values for the time of which the JWT would be considered valid based on the applicable "exp" claim. If any Client Attestation PoP JWT would be replayed, the authorization server would recognize the "jti" and respond with an authentication error.
- The authorization server provides a nonce for the particular transaction and the client uses it for the "nonce" claim in the Client Attestation PoP JWT. The authorization server validates that the nonce matches for the transaction. This approach may require an additional roundtrip in the protocol. The authorization server MUST ensure that the nonce provides sufficient entropy.
- The authorization server may expect the usage of a nonce in the Client Attestation PoP JWT, but instead of providing the nonce explicitly, the client may implicitly reuse an existing artefact, e.g. the authorization code. The authorization server MUST ensure that the nonce provides sufficient entropy.

The approach using a nonce explicitly provided by the authorization server gives stronger replay attack detection guarantees, however support by the authorization server is OPTIONAL to simplify mandatory implementation requirements. The "jti" method is mandatory and hence acts as a default fallback.

# Appendix A IANA Considerations

## Registration of attest_jwt_client_auth Token Endpoint Authentication Method

This section registers the value "attest_jwt_client_auth" in the IANA "OAuth Token Endpoint Authentication Methods" registry established by OAuth 2.0 Dynamic Client Registration Protocol {{RFC7591}}.

* Token Endpoint Authentication Method Name: "attest_jwt_client_auth"
* Change Controller: IESG
* Specification Document(s): TBC

## HTTP Field Name Registration
This section requests registration of the following scheme in the "Hypertext Transfer Protocol (HTTP) Field Name Registry" {{IANA.HTTP.Fields}} described in {{RFC9110}}:

* Field Name: OAuth-Client-Attestation
* Status: permanent
* Reference: [](#headers) of this specification

<br/>

* Field Name: OAuth-Client-Attestation-PoP
* Status: permanent
* Reference: [](#headers) of this specification

--- back

# Additional Examples

## Wallet Instance Attestation

This non-normative example shows a client attestations used as an wallet instance attestation in the context of eIDAS 2.0 {{ARF}}, e.g. to secure a Type-1 configuration credential. The additional claims describe the wallet's device binding und user binding capabilities and the achievable level of assurance.

~~~
{
	"typ": "wallet-attestation+jwt",
	"alg": "ES256",
	"kid": "1"
}
.
{
	"iss": "https://attestation-service.com",
	"sub": "https://wallet-provider.com",
	"iat": 1541493724,
	"exp": 1516247022,
	"attested_security_context" : "https://eu-trust-list.eu/asc/high",
	"cnf": {
		"jwk" : {
			"kty": "EC",
			"crv": "P-256",
			"x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
			"y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
		},
		"key_type" : "STRONGBOX",
		"user_authentication" : "SYSTEM_PIN"
	}
}

~~~

# Document History

-04

* add iana http field name registration

-03

* remove usage of RFC7521 and the usage of client_assertion
* add new header-based syntax introducing Oauth-Client-Attestation and OAuth-Client-Attestation-PoP
* add Client Instance to the terminology and improve text around this concept

-02

* add text on the inability to rotate the Client Instance Key

-01

* Updated eIDAS example in appendix
* Removed text around jti claim in client attestation, refined text for its usage in the client attestation pop
* Refined text around cnf claim in client attestation
* Clarified how to bind refresh tokens to a Client Instance using this client authentication method
* Made it more explicit that the client authentication mechanism is general purpose making it compatible with extensions like PAR
* Updated acknowledgments
* Simplified the diagram in the introduction
* Updated references
* Added some guidance around replay attack detection

-00

* Initial draft

# Acknowledgments
{:numbered="false"}

We would like to thank
Brian Campbell,
Francesco Marino,
Guiseppe De Marco,
Kristina Yasuda,
Michael B. Jones,
Takahiko Kawasaki
and
Torsten Lodderstedt
for their valuable contributions to this specification.
