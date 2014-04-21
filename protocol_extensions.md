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