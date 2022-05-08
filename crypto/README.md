# Introduction

- Where is Cryptography needed?
- How do I encrypt data the right way?
- How do I make it easier? What tools do we have on node to make this operation eaiser?

**four ways to use Cryptography**

- Protecting passwords
- Protecting data at rest
- Protecting data in transit
- Two factor authentication

**Required Knowledge**
You should have a good grasp on Cryptography before continuing this write-up

## Tour of Cryptography Modules on Node

We have `crypto` module on node, on that module we've:

- Cipher/Decipher Classes: which do most of the heavy lifting of encrypting and decrypting 
- DiffiHellman/ECDH: methods for exchanging keys
  DiffieHellman is a method of exchanging keys
- Hash: one way hashing
- HMAC: generate hash-based message authentication codes, those codes are sent along with a message to verify the identity of sender, HMAC is very powerful when used to protect data in transmission
- Sign/verify: Singing and verifying the integrity of a document
- Crypto methods: there are some methods that are not ties to specific classes

### Protecting passwords

Why Passwords need to be protected?

- Dictionary attacks 
- Duplication avoidance
- Data Breaches

**Hashing**
