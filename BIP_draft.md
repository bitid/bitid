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
bitid:www.site.com/callback?x=NONCE
```

- **bitid** is the protocol scheme (can also be bitid://)
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

**Normalization:** the URI shouldn't be changed at all before signature, and the callback should also be an exact reflection of the URI (including uppercases, escaped characters, etc)

<pre>
\x18Bitcoin Signed Message:
%bitid:www.site.com/callback?x=NONCE
</pre>

The receiving server verifies the validity of the signature and proceeds to authenticate the user. 
Server-side, only the user's public key is stored. A timeout for the validity of the nonce should 
be implemented by the server in order to prevent replay attacks.

## Alternative currencies

In general, user may elect to authenticate using other currency than Bitcoin, or even using a public key authentication method not related to a cryptocurrency. The client should indicate the authentication method used by setting the `Bitid-Method` header. If the client makes a request using an unsupported authentication method, the server should respond with `501 Not Implemented` status code.

The server should indicate supported methods in the `Accept-Bitid-Method` header when responding to `OPTIONS` request. The server may demand authentication using a particular method by generating an URI using a `bitid+METHOD_LOWERCASE` scheme. For example, to request authentication using Bitcoin, the `bitid+bitcoin` URI scheme should be used.

## HD wallet derivation path

For maximum compatibility with other identification scheme We follow the [https://github.com/satoshilabs/slips/blob/master/slip-0013.md](SLIP0013) structure from TREZOR connect.

* `URI` is the **callback** URI (not the BitID URI)
* `index` (32 bit unsigned integer) : used so one can generate more keys corresponding to the same URI. If not set, by default the index should be `0`

**HD structure**

1. Let’s concatenate the little endian representation of index with the URI.
2. Compute the SHA256 hash of the result.
3. Let’s take first 128 bits of the hash and split it into four 32-bit numbers A, B, C, D (each in little endian notation)
4. Set highest bits of numbers A, B, C, D to 1.
5. Derive the HD node m/13’/A’/B’/C’/D’ according to BIP32.

**Test vectors**

BIP39 seed: `inhale praise target steak garlic cricket paper better evil almost sadness crawl city banner amused fringe fox insect roast aunt prefer hollow basic ladder`    
Callback URI: `http://bitid.bitcoin.blue/callback`    
Index: `0`

To hash: `00000000687474703a2f2f62697469642e626974636f696e2e626c75652f63616c6c6261636b` (binary hash)    
Hash result: `123155becf82afc03bfb614337bfd2eddae7046183a6d1a6dfb02b1966fdb321`    
BIP32 path: `13'/0xbe553112'/0xc0af82cf'/0x4361fb3b'/0xedd2bf37'`    
BitID address: `1J34vj4wowwPYafbeibZGht3zy3qERoUM1`    

## HD wallet derivation path (legacy)

This is the first proposal, before we decided to follow the SLIP0013 from Trezor. You should **NOT** implement the following HD structure, it is given for reference only.

<pre>
m/0'/0xb11e'/sha32uri/n

Where:
uri = "bitid:www.site.com/callback" (no parameters)
sha32uri = 32 highest bits of sha256(uri)
n = identity index, in case multiple ids are needed for this uri (default = 0)
</pre>

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

Source code of the demonstration service :  
https://github.com/bitid/bitid-demo

Ruby implementation :  
https://github.com/bitid/bitid-ruby

Python implementation :  
https://github.com/LaurentMT/pybitid

Javascript implementation :  
https://github.com/porkchop/bitid-js  

PHP implementation :  
https://github.com/conejoninja/bitid-php  

Android Bitcoin wallet fork including support of BitID :  
https://github.com/bitid/bitcoin-wallet

# See also

Reddit thread on the need of such a protocol :
http://www.reddit.com/r/Bitcoin/comments/1nkoju/bitcoin_core_dev_websites_do_not_need_passwords/

SQRL, a similar proposal not limited to Bitcoin :
https://www.grc.com/sqrl/sqrl.htm
