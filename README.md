BitID
=====

*Bitcoin Authentication Open Protocol*

Pure Bitcoin sites and applications shouldn’t have to rely on artificial identification methods such as usernames and passwords. BitID is an open protocol allowing simple and secure authentication using public-key cryptography.

# Why ?

When they need to deal with Bitcoin services, users already own at least one public and private key-pair: their Bitcoin addresses. Using their wallet for authentication purposes has many benefits :
- "one-click" registration and login procedures
- no need to remember or duplicate passwords
- the server only knows and stores the users's public key
- authentication by a Bitcoin address allows the service to use it (ie: Mining pool payment address)
- optionally, connect to a decentralized identification system in order to populate registration fields (nickname, email ...)

Of course, these benefits mostly apply for Bitcoin related services, leveraging the fact that users already have a wallet and presumably took all the necessary steps to protect and back it up. For non-cryptocurrency-related services, other authentication services such as OpenID or Facebook connect may be better suited.

# Acknowledgment

Authenticating using a cryptographic challenge isn't a new idea and BitID doesn't claim to be an original approach. The goal is to propose common specifications and best practices as well as offer a seamless user experience.

Discussion on [Reddit](http://www.reddit.com/r/Bitcoin/comments/1nkoju/bitcoin_core_dev_websites_do_not_need_passwords/) about a need to replace user/pwd authentication with a public-key cryptographic signature. 

[SQRL](https://www.grc.com/sqrl/sqrl.htm) (Secure Quick Reliable Login) is a similar project allowing authentication using public-key cryptography. It has however a much broader scope and relies on additional software for storing the key pairs, while BitID relies on crypto-wallets that users already own.

# How does it work ?

In order to access a restricted area or authenticate oneself against a given service, the user is shown the following UX :

![](http://i.imgur.com/CvuXijh.png)

The QR code contains the following data :

```
bitid://login?x=NONCE&c=https://www.site.com/callback
```

The NONCE must always be unique, and will be the user's session ID on the site the callback is redirected to.

The user has to confirm that she wants to authenticate herself on the target website, and has to choose which Bitcoin private key will sign the QR code contents.

If the user’s Bitcoin wallet is located on the same computer, a click on the QR code should launch the wallet application and supply it with the signature request. If the wallet is located on a mobile phone, the user should scan the QR code using the application. In both cases, the following dialog options should be shown :


| Step 1 | Step 2 | Step 3 |
|--------|--------|--------|
|![](http://i.imgur.com/6KlZFGe.png)|![](http://i.imgur.com/8ZNMmdp.png)|![](http://i.imgur.com/630hUsu.png)|

After a Bitcoin address is chosen, or created on the fly, the **full bitid URI** is signed with the address’ private key. The signature with the signed challenge is then POSTed to the callback url.

The receiving server verifies the validity of the signature and proceeds to authenticate the user. Server-side, only the user's public key is stored. A timeout for the validity of the nonce should be implemented by the server in order to prevent replay attacks.

After authentication, if this is the user's first visit (sign up), the website may for instance ask her to choose a human readable nickname : 

![](http://i.imgur.com/XzcwfTC.png)

This step is optional. If not needed by the server, the user experience will be identical for registration and login purposes.

For compatibility reasons, since not all wallets will provide support for the proposed `bitid://` scheme, a manual challenge is also possible :

![](http://i.imgur.com/Giz0fGQ.png)

All wallets (including Bitcoin Core) provide manual signing capabilities, therefore any user may use the BitID protocol at the expense of the UX.

# Application examples

## Website authentication
* any bitcoin core website (mining pools, related services, otc exchanges…)
* content websites such as forums and wikis, including access control

## Real world applications
* door access controls
* lockers rental

## E-commerce
* dead simple e-commerce services: one QR code scan for authentication/cart creation, one QR code scan for payment

# Security concerns

BitID offers a secure authentication method :
* As secure as sending funds through Bitcoin
* out-of-band, keyless authentication using a smartphone wallet, allowing login through an untrusted computer
* anti-phishing protection when using a desktop wallet (IP address matching verification)
* no third party, no external compromission possible, no storage of user sensitive data on the server
* resistant to arbitrary signature requests: challenges are syntaxically verified by the wallet as valid bitid URIs
* resistant to brute force or dictionary attacks

However many responsibilities are in the hands of the user :
* the user must protect his private keys and make backups (this should already be the case)
* the user must pay attention to the URL shown in authentication requests in order to avoid man-in-the-middle attacks; the out-of-band authentication process does not allow any protection against these attacks.

* Finally, a major drawback of this protocol is the absence of revocation procedures. If the user loses her private key or if it is compromised, there is no native possibility of revoking the authentication access. The only way to revoke the user's identity is then to to establish a back-channel communication with the website using email, security questions, or a password.

# General remarks

BitID is not a general-purpose identification system. It should mainly be used when a Bitcoin address is paramount to the usage of the site or application, and when this usage is a long-term one - it wouldn’t make sense for a tipping service.

Therefore, BitID doesn’t claim to be a superior authentication system, just one well-fitted to Bitcoin and altcoins-related applications.

# Diagrams

![](http://i.imgur.com/t56gW1w.png)

# Implementation

## Wallet implementation requirements

To be compatible with the BitID protocol, a wallet must implement the following:
* register the bitid:// scheme
* throw a bitid:// intent when scanning a BitID QR code (if applicable)
* decode the URI and verify its format
* display a request for authentication showing the domain name callback and ask for validation
* ask the user to pick up or create a Bitcoin address for the authentication (show the last Bitcoin address used if this is a known callback address)
* sign the BitID URI with the private key
* POST the signature and the signed challenge to the callback URL
* completion dialog : success/retry/cancel

Specs and UX should be the most simple possible in order to bring on board the developers of the most used wallets.

## Backend implementation requirements

To be compatible with the BitID protocol, a server application must implement the following :
* Create a callback route
* Generate a complex enough nonce (match it to the session and timestamp it)
* Generate an URI with the nonce and the callback URL
* Display the corresponding QR code
* On receiving the POST, verify the signature with the user's public key
* If correct, login the session with the Bitcoin address and notify the front-end using sockets
* Store the public key as the user identity, allowing for storing user preferences and additional data
* Alternate version for zero-day compatibility :
    * display a form with the following fields: generated nonce, Bitcoin public address and signature
    * when submitting, verify and return to login or error

Protocol extensions
=====

## Decentralized certification authority

One of the weaknessess of BitID is the lack of authentication revocation capabilities, such as when a user loses her private key. By adding a decentralized certification authority (CA) to the protocol, this security feature becomes feasible.

Storage of the revocation information is done using the Namecoin blockchain under the `bitid/` namespace, which effectively acts as a decentralized CA. In a raw implementation, the user must therefore have a Namecoin wallet and some NMC to pay for the data recording.

### Principle

First, the user needs to associate her Namecoin address with her Bitcoin address, by publishing a proof of ownership on the Namecoin blockchain :

**entry key** `bitid/[bitcoin_address]`
```
{
  "signature" : [entry key signed with bitcoin_address' private key] 
}
```

If the bitcoin address needs to be revocated, the user must update the Namecoin record as such :

**entry key** `bitid/[bitcoin_address]`
```
{
  "revocated" : true,
  "signature" : [entry key signed with bitcoin_address' private key] 
}
```

This record publication is possible even if the user lost her Bitcoin address private key, because the proof of ownership has been delegated to her Namecoin address.

After a successful BitID authentication, the back-end service must check for the existence and validity of a record on the Namecoin blockchain, and if one exists, it checks if address has been revocated. If these conditions are met, access to the service must be denied.

Delegating revocation to a Namecoin address offers the user a security mechanism in case her Bitcoin private key is lost or stolen. For maximum security, Namecoin keys can be kept in a safe and used only in case of revocation.

## Decentralized identity

When signing in with a new service, a user may want to automatically share some data such as an avatar, nickname, Twitter handle, and so on. [OneName](http://www.onename.io) and other services offer such functionality, however the key to retrieve data is the user's nickname, not their Bitcoin address.

Using the same proof of trust as previously, user can store his data in the Namecoin Blockchain :

**entry key** `bitid/[bitcoin_address]`
```
{ 
  "username" : "EricLarch", 
  "avatar" : "http://img.com/bla",
  "twitter" : "EricLarch",
  "proof" : [entry key signed with bitcoin_address' private key] 
}
```

When the server checks the Bitcoin address for revocation, it can also fetch the user data and use it to automatically populate some registration fields. If the address has been revocated, any associated user data must be considered as invalid by the service.

All fields are public, so the user may not filter which fields may be fetched by a given website. As is, this service cannot be trusted for sensitive information such as an email address.

## Third-party service

The whole process of registering a Namecoin address, buying NMC and publishing this data is quite cumbersome. A third-party service adding a layer of abstraction would greatly simplify usage without compromising security.

Such a service could also add privacy features such as encrypting the user data and providing only subsets of the data to a given service, per the user's request.

# Author

Eric Larchevêque (elarch@gmail.com)
