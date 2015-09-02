BitID metadata
=====

By maintaining metadata associated to public addresses and payment requests (BIP70),
wallets can leverage BitID and provide added value to the user.

# Metadata format

Metadata can be stored using a local SQL database, in one table.

````
TABLE bitid_metadata
  address STRING,
  callback_host STRING,
  is_default BOOLEAN,
  auto_connect BOOLEAN,
  amount INTEGER,
  merchant_name STRING,
  memo STRING,
  created_at INTEGER,
  updated_at INTEGER
````

Queries are done on `address` and `callback_host`, so indexes must be created on these two columns.

`created_at` and `updated_at` are timestamps managed automaticaly by the db accessor.

Wallet must have an export / import function for this database on a common simple format so
it can be shared between different wallets for the user's convenience. Private key transfers
should be done using the wallet's own implementation.

## Request for authentication

When a request for authentication is received, `bitid_metadata` is queried on `callback_host`.  
If no address is found, then a new one is created and added in the metadata (only `address` and
`callback_host`).  
If one or more addresses are found, the user can choose the one he wants to authenticate itself,
or create a new one.

`is_default` is the first address to be shown for a callback host.  
`memo` is editable by the user and is just plain text.  
If `auto_connect` and `auto_connect` are true, the wallet should skip the UI and auto sign the
auth request (see 2FA).

## Payment request

When a BIP70 payment request is received and paid by the user, an entry must be added in the database.

`address` is the first originating address.  
`callback_host` is the host of the payment url.  
`amount` is the paid amount in satoshis.  
`merchant_name` is the merchant name extracted from the payment request.  
`memo` is `memo` from the payment request.

# Use cases

## Authentication

By maintaining an address book, the wallet can easily fetch correct identities depending
on the callback url. So user doesn't have to remember which address was used for which
service.

However, this implies BitID auth are scoped to hostname and will ignore the path.
Therefore there can be only one BitID auth per host.

## Decentralized 2 factors authentification

Using the `auto_connect` feature, the BitID UX is extremely quick. This setup would be
most useful for a 2 factors authentification where switching address is not relevant.

For instance, if the auth request URI has `a=1` then at creation time the user has
the opportunity so set the generated address as a 2FA and save it in auto connect mode.

After logging in with normal credentials (for instance email / password), a BitID
auth challenge is presented to the user. A simple scan of the QRcode using the wallet
will auto sign the challenge and validate the 2FA.

## Door control

The `auto_connect` feature can also be used to very easily control access of a connected
lock, such as a hotel room or a locker.

## E-commerce

Using BitID in conjonction with the BIP70 payment requests allows to greatly smooth and
facilitate the online shopping experience.

Is it possible to allow a direct purchase without the need of creating an account. This
is very important for the seller, as less effort from the user results in higher conversion
rates.

Let's say I'm browsing an e-book merchant. I see a title I want to buy, I just need to
activate the BIP70 payment request and pay from my wallet, which gives me access to the
file.  
Now, if I want to come back and access again the file, I select the BitID auth on the
merchant website, pick up the correct purchase from the wallet database and auth myself
with the correct address.

It will work for any kind of purchase, paying without the friction of having to create
and account, and accessing after my order any time using BitID.

# Scope & permissions

If you use BitID to sign up on websites, you may also need to provide with personal 
information such as username, email, profile picture etc. Using BitID and wallets, 
it is possible to devise a simple and secure way to communicate these information to 
the service.

Using key/values local database (or another SQL table), user optionaly fills in all
the personal information fields he thinks necessary.

| Field | Permission scope |
| --- | --- |
| Email | e |
| Username | u |
| First name | n |
| Last name | l |
| Date of birth | d |
| Gender | g |
| Twitter | t |
| Facebook | f |
| Homepage | u |
| Profile picture url | p |
| Address Line 1 | a1 |
| Address Line 2 | a2 |
| City | c |
| State / Province | s |
| Postal Code | z |
| Country | y |

Permissions to get fields are requested through the param `s` (for scope). Example :

```
bitid://bitid.bitcoin.blue/callback?x=NONCE&s=eup
```

This would request email, username and profile picture url. If this is a first time
authentication to this host, user will have the possibility to check or uncheck the
three fields.

All granted fields are sent with the callback :

```
{ "e": "me@email.com",
  "u": "Me me",
  "p": "http://www.img.com/me.jpg" }
```

Personal information are hosted on the wallet, and shared on a case
by case basis with a full control of the user. This ensures maximum privacy, whilst
providing with a convenient solution to quickly fully signup on services.
