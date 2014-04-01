BitID
=====

*Bitcoin Authentication Open Protocol*

Pure Bitcoin sites and applications shouldn’t have to rely on artificial identification such as usernames and passwords. BitID is an open protocol allowing simple and secure authentication by a Bitcoin address, using a cryptographic signature challenge.

# Why ?

When dealing with Bitcoin services, user has most of the time a wallet with at least one address. Using this wallet for authentication purpose has many benefits :
- seamless registration / login experience
- no need to remember password, added security
- authentication by Bitcoin address, allowing service to know return address if needed
- possibility of connecting with a decentralized identification system to populate registration fields (name, email ...)

Of course, these benefits apply only for Bitcoin related services. It leverages the fact that users already have a wallet and already took all steps to protect/backup it. For mainstream services, other auth services such as OpenID, Facebook connect, etc are much more suited.

# Acknowledgment

Authenticating using a cryptographic challenge isn't a new idea and BitID doesn't claim to be an original approach. The goal is to propose common specifications and best practices as well as provide a seamless user experience.

Discussion on [Reddit](http://www.reddit.com/r/Bitcoin/comments/1nkoju/bitcoin_core_dev_websites_do_not_need_passwords/) about a need to replace user/pwd authentication with a key-based signature. 

[SQRL](https://www.grc.com/sqrl/sqrl.htm) (Secure Quick Reliable Login) is a similar project allowing authentication using public-key cryptography. It has however a much broader scope and relies on additional software for storing the key pairs, while BitID relies on crypto-wallets that users already have access to.

# How does it work ?

In order to access a restricted area or just to identify herself, the user is shown the following UX :

![](http://i.imgur.com/CvuXijh.png)

The QRcode contains the following data :

```
bitid://?s=NONCE&c=https://www.site.com/callback
```

The NONCE must always be unique, and will be the user's session ID on the site the callback is redirected to.

The user has to confirm that she wants to authenticate herself on the target website, and has to choose which Bitcoin private key will sign the QR code contents.

If the user’s Bitcoin wallet is located on the same computer, a click on the QR code should launch the wallet application and supply it with the signature request. If the wallet is located on a mobile phone, the user should scan the QR code using the application. In both cases, the following dialog options should be shown :


| Step 1 | Step 2 | Step 3 |
|--------|--------|--------|
|![](http://i.imgur.com/6KlZFGe.png)|![](http://i.imgur.com/8ZNMmdp.png)|![](http://i.imgur.com/630hUsu.png)|

After a Bitcoin address is chosen, or created on the fly, the **full bitid URI** is signed with the address’ private key. The signature and public key are then POSTed to the callback url.

The receiving server verifies the validity of the signature and proceeds to authenticate the user. Server-side, only the user's public key is stored. A timeout for the validity of the nonce should be implemented by the server in order to prevent replay attacks.

After authentication, if this is her first visit (sign up), the website can for instance ask the user to choose a human readable nickname :

![](http://i.imgur.com/XzcwfTC.png)

For compatibility reasons, as not all wallets will provide support for `bitid://` scheme, a manual challenge is also possible :

![](http://i.imgur.com/Giz0fGQ.png)

With this option, all wallets (including Bitcoin Core) can use the BitID protocol, at the expense of the UX.

# Application examples

## Website authentication
* any bitcoin core website (mining pools, services, otc exchanges…)
* content website such as wikis, including access control

## Real world applications
* door access control
* locker rental

## E-commerce
* dead simple digital content commerce service

# Security concerns

BitID claims to have a secure authentication method :
* out of band authentication when using a smartphone wallet
* anti phishing protection when desktop wallet (IP verification)
* no third party, no external compromission possible
* resistant to arbitrary signature, as challenges are syntaxically verifiable by the wallet

However many responsibilities are in the hand of the user :
* user must protect his private key and make backups (this should already be the case anyway)
* user must pay attention to the URL in authentication requests (man in the middle attack) ; it is not possible to auto-detect phishing attempts.

Also, a major flaw of this protocol is the absence of revocation. If you lose your private address or if it becomes compromised, you have no native possibility of revoking the access. The only way is to establish a back channel communication with the website (email, secondary address…)

# General remarks

BitID is not a general purpose identification system. It should only be used when a Bitcoin address is paramount to the usage of the site or application, and when this usage is long term (wouldn’t make sense for a tipping service).

Therefore, BitID doesn’t claim to be a superior authentication system, just one better fitted to specific Bitcoin applications.

# Diagrams

![](http://i.imgur.com/t56gW1w.png)

# Implementation

## Wallet implementation requirements

To be compatible with the BitID protocol, a wallet must implement the following :
* register the bitid:// scheme
* throw a bitid:// intent when scanning a BitID QRcode (if applicable)
* decode the URI and verify its legality
* show a request for authentication showing the domain name callback, ask for validation
* ask the user to pick up or create a Bitcoin address for the authentication (show the last Bitcoin address used if this is a known callback address)
* sign the URI with the private key
* POST to the callback URL
* completion dialog : success / retry

To successfully get the maximum number of wallet to implement BitID, specs and UX must be the most simple possible.

## Backend implementation requirements

To be compatible with the BitID protocol, a server application must implement the following :
* Create a callback route
* Create a nonce (match it to the session and timestamp it)
* Build URI with a nonce and the callback
* Build and show QRcode
* On callback, verify the signature
* If correct, login the session with the Bitcoin address and notify the front end using sockets
* Alternate version for zero day compatibility :
    * show a form with the nonce to sign
    * two required fields : Bitcoin address and signature
    * when submitting, verify and return to login or error

Protocol extension
=====

## Decentralized certificate authority

One of the weaknessess of BitId is the impossibility of access revocation when losing its private key. By adding a decentralized CA to the protocol, this security feature becomes possible.

Storage of the CA is done using the Namecoin blockchain under the `bitid/` namespace. In a raw implementation, user must therefore have a Namecoin wallet and some NMC to pay the registration of the datas. 

Implementation of authority and identity storage is done using Namecoin.

### Principle

First, user needs to register proof of Bitcoin address ownership in the Namecoin blockchain.

**key** `bitid/[bitcoin_address]/proof`
```
{ "nonce" : nonce, 
  "signature": [nonce signed with address' private key] }
```

At challenge signature verification time on the website back end, BitID service checks for the proof of trust and its validity. If the proof is legal (signature matches), then it is checked for existence of a revocation at `bitid/[bitcoin_address]`

If the value is

```
{ "revoke" : true }
```

and if the transaction comes from the same Namecoin address than the proof of trust, then authentication must fail and access cannot be granted.

#### Why using two key/value ?
Proof of trust must be shown, so if we had only one key we would need to add a signature of the data. However, if you just lost your private key you cannot do that. Proof that the Namecoin address publishing the revocation order is trusted has been previously stored in the blockchain. Establishment of trust must of course be done in advance.

#### What happens if you lose access to your Namecoin wallet ?
If you lose access when you still have your Bitcoin private key, then you can add a new proof (`/proof-1`). If you lose it in the same time than your Bitcoin private key, then you are doomed.

## Decentralized identity

When you are signing in a new service, you may want to automatically share some data such as your avatar, username, Twitter handle, etc. Some services doing that exist, such as [OneName](http://www.onename.io) but the key to retrieve data is your name, not your Bitcoin address.

First, you need to establish trust (see previous section), and publish the data you want to share :

**key** `bitid/[bitcoin_address]`
```
{ "username" : "EricLarch", 
  "avatar" : "http://img.com/bla",
  "twitter" : "EricLarch" }
```

When the server checks for revocation, it can fetch in the same time your data and use it to populate some registration fields.

All fields are public, and you cannot filter what you give to whom. As is, this service cannot be trusted for sensitive information such as your email.

## Third party service

The user experience of having the necessity to register a Namecoin address, buy some NMC and do all the publication of the data is quite cumbersome. A third party service adding a layer of abstraction would greatly simplify usage without compromising security.

It could also add some nice privacy features such as encrypting the identity data and reveal only extracts per the user's will.

# Author

Eric Larchevêque (elarch@gmail.com)
