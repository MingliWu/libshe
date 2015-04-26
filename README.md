# libshe

[![Build Status](https://travis-ci.org/bogdan-kulynych/libshe.svg?branch=master)](https://travis-ci.org/bogdan-kulynych/libshe) [![Coverage Status](https://coveralls.io/repos/bogdan-kulynych/libshe/badge.svg?branch=master)](https://coveralls.io/r/bogdan-kulynych/libshe?branch=master)

Symmetric homomorphic encryption library.


## Introduction

Homomorphic encryption is a kind of encryption that allows to execute functions over the ciphertexts without decrypting them. This library implements a symmetric variant of originally asymmetric homomorphic encryption scheme over the integers by van Dijk et al. [(DGHV10)][DGHV10] using ciphertext compression techniques from [(CNT11)][CNT11]. The symmetricity of the scheme means that only the private key is used to encrypt and decrypt ciphertexts. A relatively small public element, however, is used in homomorphic operations, but it is not a real public key.

Such scheme is useful in secure function evaluation setting, where a client encrypts an input to an algorithm using their private key, sends it to a server which executes an algorithm homorphically, and sends the output back to the client. The client then obtains the output of the algorithm by decrypting server response using the private key.

See the following diagram for visual explanation.

- Let _f_ be an algorithm to be evaluated on a server.
- Let _a[1], a[2], ... a[n]_ be inputs of _f_ that client provides to the server.
- Let _b[1], b[2], ... b[n]_ be inputs of _f_ that server possesses.
- Let _p_ be the client's private key, and _x[0]_ be the corresponding public element

![SFE](misc/sfe.png)

See [technical report][Kul15] (draft) for details.


## Status

This is experimental software and **should not be used in mission-critical applications.**

### Roadmap

- [x] CI and coverage reports
- [ ] Installation
- [ ] Timing and memory estimates
- [ ] Comparison with [HElib](https://github.com/shaih/HElib)
- [ ] Documentation
- [ ] Additional homomorphic operations


## Installation

### Requirements

- gcc >= 4.8
- [boost](http://www.boost.org/) >= 1.55
- [GMP](https://gmplib.org/) >= 6.0.0
- [lcov](http://ltp.sourceforge.net/coverage/lcov/readme.php) >= 1.11 (optional)

### Building

Run tests

```
make tests
```

Produce a library in `build` directory

```
make
```

_Note_. Default path to boost libraries is `/usr/local/lib`. Set `BOOSTDIR` env variable to change.


## Usage

```cpp
using she::ParameterSet;
using she::PrivateKey;
using she::HomomorphicArray;

// Generate parameter set so that the encryption scheme:
// - Has medium security level (62-bit)
// - Allows to evaluate at least 1 multiplications for every bit in plaintext
// - Seeds the non-secure random number generator with 42
const ParameterSet params = ParameterSet::generate_parameter_set(62, 1, 42);

// Generate private key
const PrivateKey sk(params);

// Encrypt plaintext
const vector<bool> plaintext = {1, 0, 1, 0, 1, 0, 1, 0};
const auto compressed_ciphertext = sk.encrypt();

// ...
// Serialize and send compressed ciphertext
// ...

// Expand the ciphertext to perform operations
const auto ciphertext = compressed_ciphertext.expand();

// Execute some algorithm
const vector<bool> another_plaintext = {1, 1, 1, 1, 1, 1, 1, 1};
const auto response = ciphertext ^ HomomorphicArray(another_plaintext);

// ...
// Serialize and send back the response
// ...

// Decrypt to obtain the algorithm output
const auto decrypted_response = sk.decrypt(response);
const vector<bool> expected_result = {0, 1, 0, 1, 0, 1, 0, 1};
assert(decrypted_response == expected_result);

```

## License

The code is distributed under the GPLv3 license.

Copyright © 2015 Bogdan Kulynych. `hello [at] bogdankulynych.me`



[DGHV10]: http://eprint.iacr.org/2009/616.pdf
[CNT11]: http://eprint.iacr.org/2011/440.pdf
[Kul15]: http://bogdankulynych.me/papers/vdghv.pdf