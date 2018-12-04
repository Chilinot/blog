# Building A Micro Service Based System With Rails
This blogpost is not intended as a complete guide to building microservice based systems.
It's intended as a place for me to write down my thoughts and ideas regarding the stuff I'm working on.
Thus, certain information and procedures might be omitted.

## Ruby on Rails
Ruby on Rails has a large community with plenty of tools to make this possible. It is also one of the simpler and faster ways to get started.

I'm going to use the following gems to make this work:
 * [rack-cors](https://github.com/cyu/rack-cors)
 * [simple\_command](https://github.com/nebulab/simple_command)
 * [jwt](https://github.com/jwt/ruby-jwt)

## "Asymmetric" Authentication
I call the authentication and authorization system I will design "asymmetric", because it is based on public/private key based encryption and validation.

Basically, there is an authentication/authorization microservice (auth) responsible for creating tokens which are then signed using the secret private key known only to that microservice.
Other microservices can then validate this token using the well known public key that is shared with all services in the system.

This allows us to build systems that don't have to constantly check with the auth-service for each user request; they are validated directly in the service instead.

### Issues
This system however introduces some key issues that will have to be solved, namely:
 * Token revocation
 * Private/Public key change

#### Token Revocation
Since the services are not actively verifying tokens against the auth-service, tokens can not be revoked once they are given to a user/program.
This is not ideal, what if you have given out a token to a user in error. Maybe your auth-service sent out a token with admin-privileges to a user which then leaves the company for example.
In order to invalidate the token you have sent out, you will have to change the private and public key of your auth-service and all users must then re-authenticate to get new valid tokens.
It's not hard to see that this issue is pretty bad.

The simplest way to mitigate this problem is to only create tokens which expire in a short time. For example, tokens that expire 8 hours after they are created.
Users will have to re-authenticate often. But it is the solution which requires the least changes to your system.

Another solution would be to check tokens against the auth-service for tokens that are older than a certain time. The auth-service could then keep a list of tokens which are no longer valid to use.
The above solution could then be transformed into this:
 * If the token is younger than 8 hours, validate it with the public key and authorize the user.
 * If the token is older than 8 hours, validate it with the auth-service, authorize the users if the auth-service check pass.

Thus you introduce the 8 hour "invalidation", without the introduced overhead of checking tokens for every single user-request. And without requiring your users to re-authenticate daily.

But for this blogpost I'm going with the "8 hour valid" solution since it's the simplest to implement.

#### Public/Private Key Change
