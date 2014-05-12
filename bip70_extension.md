# BitId extension for Payment Protocol (BIP70)

A proposal to extend the payment protocol (BIP70) with additional data allowing negociation of a bitid token (basically a bitcoin address later used to access resources related to the goods or services bought by the user).


## Motivation

Let's have a short thought experiment: After a long week, you decide to have some fun and go to the movie theater.
You: Hi ! May I have 2 seats for the wolf of wall street ?
Cashier: Sure ! May you fill this form with your civility, firstname, lastname, address, phone number, credit card number, expiry date and CVV2 ?
You: ...
Cashier: ...?
You: wtf ?!!!

In real life, payment is the only thing required to finalize a transaction with a merchant. Sometimes it makes sense to disclose personal data but these cases are exceptions (when you expect a delivery, when you rent an expensive good, ...). In the digital world, disclosing personal data to access bought goods or services has always been the rule but this model has several drawbacks:
- it's conceptually wrong: it introduces the concept of identity in processes which should not rely on identity,
- it requires customers give personal information without any additional gain for them,
- it requires e-merchants act as secure data hosts, when their core business is selling products or services, 
- it frequently results in data leaks producing nuisances like hacked accounts, phishing, spam,...


## Rationale

Bitcoin (the app) has been designed as an electronic payment system. There's an asymetry in bitcoin: addresses (actually key pairs) are used to unlock unspent transactions but the protocol does not have the concept of a sender (address sending a transaction). This asymetry is not a flaw but is by design. On the other hand, commerce at its core is a symetric activity: customer gives money and receives a good (a service, ...). Per se, the bitcoin protocol does not allow to fix issues related to the current model.

BitId has been proposed as a solution to enrich bitcoin wallets with an open protocol allowing simple, fast and secure authentication. 
The protocol has many use cases and is sometimes talked as an additional tool to authenticate user's identity. For this proposal, we are interested in a very different aspect of BitId as a tool allowing to get rid of the concept of identity. 

Payment protocol (BIP70) has been proposed as a standardized way to negociate payments between 2 entities. Payment protocol has been designed to be extensible and seems the natural place for articulating bitcoin & bitid protocols.


## BitId in the payment protocol

### PaymentDetails 

<pre>
    message PaymentDetails {
        optional string network = 1 [default = "main"];
        repeated Output outputs = 2;
        required uint64 time = 3;
        optional uint64 expires = 4;
        optional string memo = 5;
        optional string payment_url = 6;
        optional bytes merchant_data = 7;
        optional string bitid_challenge = [tbd];
    }
</pre>

Additional fields:
- bitid_challenge : bitid uri proposed as a challenge. May be used to negociate a bitid token which will be used by the customer to access resources. 

Note: 
- Server needs to manage a extended delay for this kind of challenge (more than default 10 minutes)


### Payment

<pre>
    message Payment {
        optional bytes merchant_data = 1;
        repeated bytes transactions = 2;
        repeated Output refund_to = 3;
        optional string memo = 4;
        optional string bitid_sign = [tbd];
        optional string bitid_addr = [tbd];
    }
</pre>

Additional fields:
- bitid_sign : signature of the bitid_challenge.
- bitid_addr : address associated to the key pair used to sign the challenge and which will be used later to authenticate the customer

Note:
- Initial bitid_challenge is not sent again. Merchant retrieves it from the PaymentDetails message.
- Do we need to send the address or can we rely on the signature to extract the address ?


### PaymentACK

<pre>
    message PaymentACK {
        required Payment payment = 1;
        optional string memo = 2;
        optional string bitid_addr = [tbd];
        optional int bitid_status = [tbd];
        optional int bitid_msg = [tbd];
    }
</pre>

Additional fields:
- bitid_addr : address registered by the merchant as the bitid token associated to this payment
- bitid_status : error code if validation of signature has failed
- bitid_msg : error message if validation of signature has failed

Note:
- An error during the validation of the bitid_challenge shouldn't prevent the completion of the payment but should fail gracefully, giving instructions to the user on how to to authenticate manually from the website. 


### Remarks

- Should we allow the user to request several addresses ? (multivalued fields in Payment and PaymentACK messages)
