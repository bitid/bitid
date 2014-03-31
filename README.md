BitID
=====

*Bitcoin Authentication Open Protocol*

Pure Bitcoin sites and applications shouldn’t have to always rely on artificial identification such as username and password. BitID is an open protocol allowing simple and secure identification by a Bitcoin address, using a private key challenge.

# Acknowledgment

The idea to authenticate using a cryptographic challenge is nothing new and BitID doesn't claim to be an original approach.

Discussion on [Reddit](http://www.reddit.com/r/Bitcoin/comments/1nkoju/bitcoin_core_dev_websites_do_not_need_passwords/) about a need to replace user/pwd with a key based auth. 

[SQRL](https://www.grc.com/sqrl/sqrl.htm) (Secure Quick Reliable Login) is a similar project allowing authentication using public and private keys. It is however much broad in its approach and relies on additional software / app, as BitID relies on wallets which the user must already have.

# How does it work ?

To access a restricted area or just to identify himself, the user is presented with the following UX :

![](http://i.imgur.com/CvuXijh.png)

The QRcode contains the following data :

```
bitid://?s=NONCE&c=https://www.site.com/callback
```

NONCE must always be unique, could be the session ID.

User scans the code with a compatible wallet :

| Step 1 | Step 2 | Step 3 |
|--------|--------|--------|
|![](http://i.imgur.com/6KlZFGe.png)|![](http://i.imgur.com/8ZNMmdp.png)|![](http://i.imgur.com/630hUsu.png)|

The wallet lets the user pick or create a Bitcoin address, then signs the **full bitid URI** with the address’ private key and POST the result including the public key to the callback url.

Server side, the application verifies the validity of the signature and proceeds to authenticate the user with the Bitcoin address received. For added security, server should check callback is coming less than 5 min after the challenge and should invalidate the nonce immediately to avoid any possibility of replay.

After authentication, if this is her first visit (sign up), the website can for instance ask the user to choose a human readable nickname :

![](http://i.imgur.com/XzcwfTC.png)

For compatibility reasons, as not all wallets will provide support for `bitid://` scheme, a manual challenge is also possible :

![](http://i.imgur.com/Giz0fGQ.png)

With this option, all wallets (including Bitcoin Core) can use the BitID protocol, at the expense of the UX.

# Leveraging the blockchain

By monitoring transactions originating from the identified address, it is possible to add an application layer.

## Spam prevention

When registering with BitID, to validate the account and get full access, X satoshis must be sent to a validation address.

## Access control

Depending of how much or which addresses has been paid by the identified Bitcoin address, access to resources can be easily implemented.

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
