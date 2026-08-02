# Turbowarp-Authenticator
A project that allows to authenticate Scratch users on Turbowarp

## Features
* Scratch authentication on Turbowarp for ScratchAttach requests using Scratch comments
* Continuous authentication without the need to post any more comments
* Works for non-scratchers as well
* Uses TurboWarp cloud which rarely goes down

## Set up
1. Create a scratch project to post the verification codes to
2. Publish it
3. Download the .sb3 file from this repository
4. Upload it to Scratch
5. Change the URL address in the ```Authenticate through comment - 1st phase``` to match your verification codes commenting project URL
6. Publish it
7. Setup a python environment with ScratchAttach
8. Clone/Fork and Clone this repository
9. Change the project id on the 9th and 12th lines to match your projects
10. Start the ```tw_authenticator.py``` python file
11. Now add any features you need

## How to add my own verified request:
1. In tw_authenticator.py add
```
@client.requests
@verified_request
def your_function(username, your_arguments): #never use arguments of name: prime1, prime2, next_product, function_name
  your code...
  return your_return
```
2. In scratch use the ```Send user verified request (request) with the arguments (join them with & eg. "apple&banana&orange") (arguments)```

<ins>For example:</ins>
<img width="1056" height="83" alt="image" src="https://github.com/user-attachments/assets/26c27038-6f80-433d-8215-3a884192dca3" />

> [!CAUTION]
> Never ever do any of the following or it may result in impersonation and authentication being bypassed:
> * Share the primes stored in the project in any way
> * Reuse the primes
> * Pre program the primes into the project
> * Change the length of primes to be less than 50 digits
> * Use any of the existing variables
> * Send the primes to the server before the product has been posted
> * Automate the 2nd phase of comment verification

> [!WARNING]
> Please study how the project works or ask someone who knows before changing any of the existing functions

## How it works
The inner workings of this project can be pretty complicated. Thus I will provide an example to better illustrate how it works. I will use very small primes which are easy to factor in reality they are longer and take much more time to factorize.

### Authentication
1. The server is pinged to check connection
2. Scratch generates/is provided with a list of 50 character long primes: {7,3,11,5,13,17...}
3. 1st two primes (current_prime_1 = 7, current_prime_2 = 3) are multiplied together: product = 7*3 = 21
4. The user is asked to paste the product (21) into the project comments
5. The user presses a key to signal that they have posted the comment
6. The client takes next two primes (next_prime_1 = 11, next_prime_2 = 5) and multiplies them: next_product = 11*5 = 55
7. The client sends a request to the server containing (username, current_prime_1 (7), current_prime_2 (3), next_product(55))
8. The server after receiving this request checks the latest comment from the user.

```if current_prime_1(7) * current_prime_2(3) == comment(21):
   the user is legitimate
else:
   the user is illegitimate
```
> [!NOTE]
> Why?
> A malicious actor won't be able to factor the large product of the two primes and thus the only person who is able to provide the original primes is their author.

### Verification
After the user has been Authenticated and we need to verify a new request from them it works like this:
1. The client sets the current primes to the value of the next primes (current_prime_1 = next_prime_1(11), current_prime_2 = next_prime_2(55))
2. The client takes next two primes (next_prime_1=13, next_prime_2=17)
3. The client sets next_product to next_prime_1(13) * next_prime_2(17) = 221
4. The client sends the verified request request name(username,current_prime_1,current_prime_2,next_product, request_arguments)
5. The server compares the previous product with the current primes
```
if current_prime_1 * current_prime_2 == previous_product:
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
All we do is generate 2 new primes multiply them and send the previous primes and the new product. If the previous primes are right we set the product to new product and say that the user is legitimate.
