# BitID - Protocol requests


## GetChallenge

Request sent by the browser for the generation of a challenge

### Request

- Http method
	GET / POST

- Parameters
	None

### Response

- Process completed

	- Response format
		A BitId uri encoded in a suitable form (text, qrcode...)

	- Http code
		200

	- Headers
		Set-Cookie : stores a session id

- Process failed
	- Response format
		Free

	- Http code
		Free
	

## Callback

Request sent by the browser or the wallet as a response to a challenge
Request can be sent in two formats: json or html form

### Request

- Http method
	POST

- Parameters
	- uri		: string - required - bitid uri used as the challenge
	- signature	: string - required - signature of the bitid uri
	- address	: string - required - address used for authentication

### Response

- Process completed

	- Response format
		- address	: string - required - address used for authentication
		- nonce		: string - required - session id associated to the authentication

	- Http code
		200

- Process failed

	- Response format
		- status  : int - required - status code describing the error
		- message : string - required(?) - message describing the error

	- Http codes / status codes / messages
		- Invalid adress
			- Http code 	: 401
			- status		: 10
			- message		: Address is invalid or not legal
		- Address is valid but not associated to an existing account
			- Http code 	: 401
			- status		: 11
			- message		: Address is invalid or not legal
		- Invalid BitId uri
			- Http code 	: 401
			- status		: 20
			- message		: BitID URI is invalid or not legal
		- Invalid signature
			- Http code 	: 401
			- status		: 30
			- message		: Signature is incorrect
		- Illegal nonce
			- Http code 	: 401
			- status		: 40
			- message		: NONCE is illegal
		- Expired nonce
			- Http code 	: 401
			- status		: 41
			- message		: NONCE has expired
		- Server error (pb with db, ...)
			- Http code 	: 500
			- status		: 50
			- message		: ???
		- Misc error (specific to implementations by websites)
			- Http code 	: 401
			- status		: 51
			- message		: [...]


## CheckAuth

Request sent by the browser to check if authentication has succeeded 

### Request

- Http method
	GET

- Parameters
	None

- Headers
	Cookie : stores the session id

### Response

- Process completed

	- Response format
		- auth : int - required - 1 if authentication successfully completed, otherwise 0

	- Http code
		200





