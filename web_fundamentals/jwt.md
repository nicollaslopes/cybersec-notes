# JWT

### How the JWT Signature Works 

So if the header and signature of a JWT can be accessed by anyone, what _actually_ makes a JWT secure? The answer lies in how the third portion (the signature) is generated.

Consider an application that wants to issue a JWT to a user (for example, `user1`) that has successfully signed in.

Making the header and payload are pretty straightforward: The header is fixed for our use case, and the payload JSON object is formed by setting the user ID and the expiry time in unix milliseconds.

The application issuing the token will also have a key, which is a secret value, and known only to the application itself.

The base64 representations of the header and payload are then combined with the secret key and then passed through a hashing algorithm (in this case its `HS256`, as mentioned in the header)
