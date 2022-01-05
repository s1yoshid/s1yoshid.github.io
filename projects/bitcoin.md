---
layout: page
title: Bitcoin Hashing
description: >
  How you install Hydejack depends on whether you start a new site,
  or change the theme of an existing site.
hide_description: true
sitemap: false
permalink: /bitcoin
---

**Under Construction**

This project was for my ECE 111 class at UCSD. This project consisted of two parts; the first part being developing a SHA-256 RTL model and the second part being developing a bitcoin hashing RTL model using the SHA-256 hash function. After successfully developing each model, we then tried to optimize the models in terms of latency, throughput, and area * latency.

### What is the SHA-256 algorithm

SHA stands for "Secure Hash Algorithm". It is a cryptographic method of converting input data of any kind and size, into a string of fixed number of characters. The output is unique for any input and no matter the size of the input, the output size is fixed. The SHA-256 algorithm supports input messages up the $$ 2^{64} $$ bits and produces a 256 bit output hash value. A few applications of using the SHA-256 algorithm is for **verifying file integrity** or **authentication**.

A cyptographic hashing function need to have these properties in order to be completely secure:
* **Compression**
  * The output hash should be a fixed size regardless of the size of the input.
* **Avalanche Effect**
  * A minimal change in the input should change the output hash value dramatically.
* **Determinism**
  * The same input must always generate the same output.
* **Pre-Image Resistant (One-way function)**
  * There should be no ways to reverse the hashing process to retrieve the original input.
* **Collision Resistance**
  * Should be practically impossible to find two different inputs that product the same output.
* **Efficient (Quick Compression)**
  * Creating the output hash should be a fast and lightweight process.

![SHA-256 Algorithm Diagram](/projects/assets/img/sha256.JPG)

If the message is larger than 512 bits, we break down the message into 512-bit blocks. If the blocks are smaller than 512 bits, we add a padding of '0's to the block to make it 512 bits. 64 bits is reserved for storing the input message size. Therefore, most often, the last 512-bit block of the input will consist of the last bits of the input, padding, and the 64 bits of message size.

Next, the 512-bit blocks are broken up into 16 32-bit words (W0 to W15), ran through 64 rounds of compression along with 8 initial hash values and generate another 8 output hash values. This step is repeated until the last hash values are generated. Those values are added back with the initial hash values to generate the final output hash.

### Developing the SHA-256 RTL model



### Optimizing the SHA-256 RTL model



### Developing the bitcoin hashing RTL model



### Optimizing the bitcoin hashing RTL model

