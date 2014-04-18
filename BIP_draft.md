<pre>
BIP: TBD
Title: Bitcoin address authentication protocol (BitID)
Author: Eric Larcheveque, @EricLarch
Status: Pre-draft
Type: Process
Created: TBD
</pre>

# Abstract

The following BIP is an open protocol proposal allowing simple and secure 
authentication based on public key cryptography. By authentication we mean 
to prove to a service/application that we control a specific Bitcoin address 
by signing a challenge, and that all related data and settings may securely 
be linked to our session.

# Motivation

Bitcoin related sites and applications shouldn’t have to rely on artificial 
identification methods such as usernames and passwords. Using a wallet for 
authentication purposes has many benefits :

- "one-click" registration and login procedures
- no need to remember or duplicate passwords
- the server only knows and stores the users's Bitcoin public address
- services always know the return address
- optionally, connect to a decentralized identification system in order to populate registration fields (nickname, email ...)

See complete BitID presentation : http://bit.ly/bitid-slides

# Specification

In order to access a restricted area or authenticate oneself against a given service, 
the user is shown the following UX :

![](http://i.imgur.com/CvuXijh.png)

The QR code contains the following data :

```
bitid://www.site.com/callback?x=NONCE
```

- **bitid** is the protocol scheme
- **www.site.com/callback** is the callback URL (https mandatory, cannot have arguments)
- **x** is the NONCE must always be unique, and will be a link to the user's session ID on the site the callback is redirected to.

In order to have a `http` callback, add `&u=1`. This would be recommended for development
purposes only.

The user has to confirm that she wants to authenticate herself on the target website, and has 
to choose which Bitcoin private key will sign the QR code contents.

If the user’s Bitcoin wallet is located on the same computer, a click on the QR code should 
launch the wallet application and supply it with the signature request. If the wallet is located 
on a mobile phone, the user should scan the QR code using the application. In both cases, the 
following dialog options should be shown :

| Step 1 | Step 2 | Step 3 |
|--------|--------|--------|
|![](http://i.imgur.com/6KlZFGe.png)|![](http://i.imgur.com/8ZNMmdp.png)|![](http://i.imgur.com/630hUsu.png)|

After a Bitcoin address is chosen, or created on the fly, the full bitid URI is signed with 
the address’ private key. The signature and public key are then POSTed to the callback url.

**Note :** the signature must comply to the `\x18Bitcoin Signed Message:\n#{message.size.chr}#{message}` format

<pre>
\x18Bitcoin Signed Message:
%bitid://www.site.com/callback?x=NONCE
</pre>

The receiving server verifies the validity of the signature and proceeds to authenticate the user. 
Server-side, only the user's public key is stored. A timeout for the validity of the nonce should 
be implemented by the server in order to prevent replay attacks.

# Rationale

Classical password authentication is an insecure process that could be solved with public key cryptography. 
The problem however is that it theoretically offloads a lot of complexity and responsibility on the user. 
Managing private keys securely is complex. However this complexity is already being addressed in the 
Bitcoin ecosystem. So doing public key authentication is practically a free lunch to bitcoiners.

# Backward compatibility

Since not all wallets will provide support for the proposed `bitid` scheme, 
a manual challenge is also possible :

![](http://i.imgur.com/Giz0fGQ.png)

All wallets (including Bitcoin Core) provide manual signing capabilities, therefore any user may use the 
BitID protocol at the expense of the UX.

# Reference implementation

A demonstration of the workflow is available here :
http://bitid.bitcoin.blue

Right now, only manual signatures are available as there is not yet a wallet implementing this BIP.

Source code of the demonstration service :
https://github.com/bitid/bitid-demo

Ruby gem implementing challenge and verification :
https://github.com/bitid/bitid-ruby

# See also

Reddit thread on the need of such a protocol :
http://www.reddit.com/r/Bitcoin/comments/1nkoju/bitcoin_core_dev_websites_do_not_need_passwords/

SQRL, a similar proposal not limited to Bitcoin :
https://www.grc.com/sqrl/sqrl.htm
