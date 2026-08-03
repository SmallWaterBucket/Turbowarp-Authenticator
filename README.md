# Turbowarp-Authenticator
A project for authenticating the sender of ScratchAttach Requests on TurboWarp using Scratch comments and cryptographic challenge-response verification.

## Why does this exist?

Scratch projects can easily identify the currently logged-in user because they run on the Scratch website. TurboWarp, however, is independent from Scratch and does not have access to the user's Scratch login session.

As a result, a TurboWarp project cannot determine which Scratch account is using it. Anyone can simply type any username and claim to be another user.

TurboWarp-Authenticator solves this problem by proving ownership of a Scratch account through Scratch comments. Once the initial verification succeeds, the project can continue proving the user's identity without requiring additional comments.

It is important to understand that **both Scratch Cloud Variables and TurboWarp Cloud Variables are completely public**. Anyone can read their contents, and users can modify the values sent from their own projects. Because of this, cloud variables alone **cannot be used to prove a user's identity**.

TurboWarp-Authenticator is designed with this assumption in mind. It treats every value sent through the cloud as public information and uses a cryptographic challenge-response protocol to verify that the sender knows information that cannot be feasibly derived from the publicly visible data within the verification window.

> [!IMPORTANT]
> * TurboWarp-Authenticator does **not** encrypt any data sent through Scratch or TurboWarp Cloud Variables. Any requests sent through TurboWarp-Authenticator and ScratchAttach should be considered public and readable by **anyone**
> * This project is intended only to authenticate **ScratchAttach Cloud Requests**. It does **not** make Scratch or TurboWarp Cloud Variables secure or suitable for proving a user's identity.
> * Authenticating a user once does **not** automatically authenticate all future communication. Every piece of information that requires a trusted identity must be sent as a **verified request** and validated by the ScratchAttach server. Simply placing TurboWarp-Authenticator at the beginning of your project does not make subsequent cloud variable accesses or requests trustworthy.

## Features
* Authenticate Scratch users on TurboWarp
* Supports authenticated ScratchAttach requests
* Continuous authentication after initial verification
* Works for New Scratchers too
* Uses TurboWarp Cloud Variables for fast and reliable communication

## Installation
### Scratch setup
1. Create a scratch project to post the verification codes to
2. Publish it
3. Download the .sb3 file from this repository
4. Upload it to Scratch
5. Locate the ```Authenticate through comment - 1st phase``` function and change the verification project URL.
6. Publish it

### Python server
1. Set up a python environment with ScratchAttach
2. Clone this repository (or fork it first if you want to modify it)
3. Change the project id on the 9th and 12th lines to match your projects
4. Start the ```tw_authenticator.py``` python file
5. Now add any features you need

## How to add my own verified request:
1. In tw_authenticator.py add
```
@client.requests
@verified_request
def your_function(username, argument_1, argument_2): # The username variable is always injected and your arguments are passed on from scratch
  #never use arguments of name: prime1, prime2, next_product, function_name
  your code...
  return your_return
```
2. In scratch use the ```Send user verified request (request) with the arguments (join them with & eg. "apple&banana&orange") (arguments)```

<ins>For example:</ins>
<img width="1056" height="83" alt="image" src="https://github.com/user-attachments/assets/26c27038-6f80-433d-8215-3a884192dca3" />

## Security
This protocol is designed to prevent attackers from impersonating another Scratch user by observing or modifying public cloud variables. It does not provide confidentiality or protect data transmitted through the cloud.
> [!CAUTION]
> Violating any of the following assumptions may allow impersonation attacks:
> * Share the primes stored in the project in any way
> * Reuse the primes
> * Pre program the primes into the project
> * Change the length of primes to be less than 50 digits
> * Use or modify the authentication variables unless you fully understand the protocol
> * Send the primes to the server before the product has been posted
> * Automate the 2nd phase of comment verification

> [!WARNING]
> Please study how the project works or ask someone who knows before changing any of the existing functions

## How it works
The inner workings of this project can be pretty complicated. Thus I will provide an example to better illustrate how it works. I will use very small primes which are easy to factor in reality they are longer and take much more time to factorize.

### Initial Authentication
1. The server is pinged to check connection
2. Scratch generates/is provided with a list of 50 character long primes: {7,3,11,5,13,17...}
3. 1st two primes (current_prime_1 = 7, current_prime_2 = 3) are multiplied together: product = 7*3 = 21
4. The user is asked to paste the product (21) into the project comments
5. The user presses a key to signal that they have posted the comment
6. The client takes next two primes (next_prime_1 = 11, next_prime_2 = 5) and multiplies them: next_product = 11*5 = 55
7. The client sends a request to the server containing (username, current_prime_1 (7), current_prime_2 (3), next_product(55))
8. The server after receiving this request checks the latest comment from the user.

```
if current_prime_1(7) * current_prime_2(3) == comment(21):
  the user is legitimate
  previous_product = next_product (55)
else:
   the user is illegitimate
```
> [!NOTE]
> Why?
> The security assumption is that factoring a product of two random 50-digit primes is computationally infeasible within the verification window. Therefore an attacker who only sees the product cannot recover the original primes before the challenge expires.

### Verification
After the user has been authenticated and we need to verify a new request from them it works like this:
1. The client sets the current primes to the value of the next primes (current_prime_1 = next_prime_1(11), current_prime_2 = next_prime_2(5))
2. The client takes next two primes (next_prime_1=13, next_prime_2=17)
3. The client sets next_product to next_prime_1(13) * next_prime_2(17) = 221
4. The client sends the verified request request name(username,current_prime_1,current_prime_2,next_product, request_arguments)
5. The server compares the previous product with the current primes
```
if current_prime_1 (11) * current_prime_2 (5) == previous_product (55):
  user is legitimate
  previous_product = next_product
else:
  user is illegitimate
```
>[!NOTE]
>This project also introduces a custom new type, called the Verificator. All it is is just the next_product together with the expiry date. Then those verificators are stored in a dictionary with the username used as a key.

> [!IMPORTANT]
> I don't want to overcomplicate this description, but this project also uses timed expiry. Basically the huge prime products can still be factored with enough time. To prevent malicious actors from using this to their advantage, the project automatically rejects and deletes Verificators older than 10 minutes.

## In conclusion

Every authenticated request advances a one-time cryptographic challenge.

The client proves knowledge of the previous prime pair while simultaneously committing to the next challenge by sending the product of a newly generated prime pair. The server verifies the current challenge, stores the next one, and invalidates old challenges after a short expiration period.

This creates a continuous authentication chain in which each request authorizes the next one, making previously observed messages useless once they have expired.

## Acknowledgements

Parts of this README were generated or refined with the assistance of AI and have been reviewed and adapted for this project.
