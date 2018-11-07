# Shamir's Secret Sharing

__Shamir's Secret Sharing__ is an algorithm in cryptography created by __Adi Shamir__. 
It is a form of secret sharing, where a secret is divided into parts, giving each participant its own unique part.

To reconstruct the original secret, a minimum number of parts is required. 
In the threshold scheme this number is less than the total number of parts. 
Otherwise all participants are needed to reconstruct the original secret.

<img src='https://upload.wikimedia.org/wikipedia/commons/thumb/5/5a/Lagrange_polynomial.svg/440px-Lagrange_polynomial.svg.png'/>

## Scheme

### Preparation

secret is `s` = `a0`

threahold is `(k, n)`

create `f(x) = a0 + a1·x^1 + a2·x^2 + ... + a(k-1)·x^(k-1)`

where `a0 = s`, `a1,a2,...,a(k-1) <---$--- Zp`

calculate D(xi) = (xi, f(i))

where i is 1, 2, ... n

D(0) = (0, f(0)) is ths secret

send D(i) to each participant.

### Reconstruction

suppose there are `m` parties wants to work 
together to recover the secret `s`

party `i` has its secret `D(xi) = (xi, f(i))` 

then party `i` calculates
`li(x)` = `MulAll((x-xj)/(xi-xj))` for `j = 1 to m`,
and ignore when `j=i`.

so we get `f(x)` = `AddAll(f(xi)*li(x))`
for `i = 1 to m`.

let x = 0, `f(0)` is `s`

### An Efficient Way

method above uses some irrelavent values, we can get L(0) by

`s = f(0)` = `AddAll((f(xj))*MulAll(xi/(xi-xj)))`

where set `x=0` directly.

## [wiki](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing)

## Application

### Threshold Signature

- `Dealer`

    generate `secret` and `shares` for every `player`. send shares secretly.

- `player`

    when player wants to sign a message with secret key `s`, they recover the secret and use it to sign a message `m`.

    `s` can be konwn for players. 

    but with `elliptic`, players can sign a message without exposing the secret `s`  

### Distributed Key Generation(DKG)

compared to Threshold Signature, there is no `Dealer` in DKG.

- `Key Genaration`

    all `n` players work together to generate a `random secret`. 

    communication complexity: `2*(n-1)` for one player. 

- `key recovery`

    same like `Threshold Signature`.

### Attribute Based Encryption

this scheme has used `LSSS`

## Repository

### [dedis/kyber](https://github.com/dedis/kyber/tree/master/share)

-  verifiable secret sharing scheme (vss)

    "Non-Interactive and Information-Theoretic Secure Verifiable Secret Sharing"

    by Torben Pryds Pedersen.
    
    https://link.springer.com/content/pdf/10.1007/3-540-46766-1_9.pdf

-  verifiable secret sharing scheme (vss)

    "Provably Secure Distributed 
    Schnorr Signatures and a (t, n) Threshold
    Scheme for Implicit Certificates"

    by Douglas R. S   tinson, Reto Strobl

    https://link.springer.com/chapter/10.1007/3-540-47719-5_33

-  public verifiable secret sharing scheme (pvss)

    "A Simple Publicly 
    Verifiable Secret Sharing Scheme and its Application to
    Electronic Voting"
    
    by Berry Schoenmakers.

    In comparison to regular verifiable
    secret sharing schemes, 
    PVSS enables any third party 
    to verify shares
    distributed by a dealer using 
    zero-knowledge proofs.

    https://www.win.tue.nl/~berry/papers/crypto99.pdf

- distributed key generation (dkg)

    "Secure Distributed Key Generation
    for Discrete-Log
    Based Cryptosystems" 
    
    by
    R. Gennaro,
    S. Jarecki,
    H. Krawczyk,
    and T. Rabin.

    https://www.researchgate.net/publication/225722958_Secure_Distributed_Key_Generation_for_Discrete-Log_Based_Cryptosystems

- distributed key generation (dkg)

    "A threshold cryptosystem
    without a trusted party"

    by
    Torben Pryds Pedersen

    https://dl.acm.org/citation.cfm?id=1754929.