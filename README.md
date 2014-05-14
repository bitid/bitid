BitID
=====

*Bitcoin Authentication Open Protocol*

Pure Bitcoin sites and applications shouldn’t have to rely on artificial identification methods such as usernames and passwords. BitID is an open protocol allowing simple and secure authentication using public-key cryptography.

Classical password authentication is an insecure process that could be solved with public key cryptography. The problem however is that it theoretically offloads a lot of complexity and responsibility on the user. Managing private keys securely is complex. However this complexity is already being addressed in the Bitcoin ecosystem. So doing public key authentication is practically a free lunch to bitcoiners.

Video demonstration of the user flow :  
https://www.youtube.com/watch?v=3eepEWTnRTc

Slides presentation of the project :  
http://bit.ly/bitid-slides

Implementation example :  
http://bitid.bitcoin.blue

**The protocol is described on the following BIP draft and is open for discussion :**

https://github.com/bitid/bitid/blob/master/BIP_draft.md

# Application examples

## Website authentication
* any bitcoin core website (mining pools, related services, otc exchanges…)
* content websites such as forums and wikis, including access control
* decentralized 2FA

## Real world applications
* door access controls
* lockers rental

## E-commerce
When paying for something on the internet, the wallet can save meta-data from the transactions (based on BIP70 payment request). If I need to access again to my order (change of address, download again the file...) then I identify using the originating address for this transaction.

There is therefore no need to create an account prior to the payment, reducing the friction and upping the transformation rate for the merchant.

See [BIP70 extension proposition](https://github.com/bitid/bitid/blob/master/bip70_extension.md) as well as [BitID metadata](https://github.com/bitid/bitid/blob/master/bitid_metadata.md) for more information.

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

# Implementation

## Wallet implementation requirements

To be compatible with the BitID protocol, a wallet must implement the following:
* register the `bitid` scheme
* throw a `bitid` intent when scanning a BitID QR code (if applicable)
* decode the URI and verify its format
* display a request for authentication showing the domain name callback and ask for validation
* ask the user to pick up or create a Bitcoin address for the authentication (show the last Bitcoin address used if this is a known callback address)
* sign the BitID URI with the private key (using the \x18Bitcoin Signed Message:\n#{message.size.chr}#{message} convention)
* POST the signature and the public key to the callback URL
* completion dialog : success/retry/cancel

Specs and UX should be the most simple possible in order to bring on board the developers of the most used wallets.

Android Bitcoin wallet implementation :  
https://github.com/bitid/bitcoin-wallet

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

Ruby implementation :  
https://github.com/bitid/bitid-ruby (Gem)  
https://github.com/bitid/bitid-demo

Python implementation :  
https://github.com/LaurentMT/pybitid  
https://github.com/LaurentMT/pybitid_demo  
https://github.com/LaurentMT/pybitid_2fa_demo

Javascript implementation :  
https://github.com/porkchop/bitid-js  
https://github.com/porkchop/bitid-js-demo  

PHP implementation :  
https://github.com/porkchop/bitid-js

# Credits

Eric Larchevêque elarch@gmail.com (Protocol, Ruby)  
@LaurentMT (Python)  
Aaron Caswell aaron@captureplay.com (Javascript)  
@_CONEJO (PHP)  
