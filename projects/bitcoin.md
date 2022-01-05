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

Next, the 512-bit blocks are broken up into 16 32-bit words (W0 to W15), ran through 64 rounds of 'compression' along with the prior hash values and constants k[0...63] and generate another 8 output hash values.

During compression, a 64-entry array of words called w[0...63] is created. We know W0 to W15 is the original message in the block so we use those to create w[16...63] through a process called 'Word Expansion'.

![Word Expansion Algorithm](/projects/assets/img/word-expansion.JPG)

Now we use w[0...63], the prior hash values, and k[0...63] to do the 64 rounds of 'compression'. The prior hash values are used as the initial values of a, b, c, d, e, f, g

![Compression Algorithm](/projects/assets/img/compression.JPG)

Afterwards, the compression values are added to the prior hash values to create new hash values to use for the next stage.

This step is repeated until the last hash values are generated. Those values are added back with the initial hash values to generate the final output hash.

### Developing the SHA-256 RTL model

We decided that the easiest way to implement the SHA-256 model was to implement it as a state machine. The input message of the SHA-256 was also limited to up to 20 words for the scope of this project. We are to read the input message from memory and store the final output hash back into memory.

![SHA-256 RTL model](/projects/assets/img/sha256-model.JPG)

First we initialize the hash constants k[64] and create the functions:
determing_num_blocks that calculates the number of 512-bit blocks we’ll have from the
number of words given, sha256_op that does the sha256 hash calculation for one
round, and rightrotate which does a rotate right on the input.

In the IDLE state, we initialize the 8 hash values to the given constants and also
initialize the inputs into sha246_op to the given constants. We also set the hash round
counter and block counter to 0 and set mem_we to 0 so that we can start reading from
memory. If start == 1, we set mem_addr to message_addr and move to the READ state.
If not, we stay in the IDLE state.

In the READ_BUF state, we need to wait one cycle in order to read from the memory
address so we use this state to wait one cycle before going into the READ state.

In the READ state, we are using ( i ) as a counter for the amount of words which in total
is 20 words. we are assigning the mem_read_data into the message for each word and
incrementing i by 1 and assigning mem_addr to be the next mem_addr by incrementing
by i + 1. Once all the mem_addrs are read into mem_read_data we move on to the next
state BLOCK.

In the BLOCK state, we set the values of the w[16] array to use in the COMPUTE state.
If j < num_blocks - 1, the first 16 words are assigned to w. If we are not in the first block,
we assign the last 4 messages, the end of message signifier, padding, and the size of
the message to w. For our implementation of BLOCK in sha_256, it only supports up to
2 blocks.

In the COMPUTE state, we use the w values calculated in the BLOCK state to calculate
the input for our sha256_op algorithm. In the first 16 rounds of calculations, wt is simply
equal to w[current round]. For calculations after the first 16 rounds, S0 and S1 are
calculated by using a series of rightrotates and right shifts. The S0 and S1 values are
then used to calculate for a new wt value. The current round, wt, and the a-h values are
then passed into the sha256_op algorithm to compute the a-h for the next state. Once
64 rounds of calculations are complete, the previous hash value is added with the newly
computed a-h values to create a new set of hash values and a-h values. Lastly, we
check if COMPUTE was run for enough cycles to compute every block. If yes, then we
go to the WRITE state. If not, then we return to the BLOCK state to retrieve the next
block.

In the WRITE state, we are writing h0 through h7 to the memory addresses
(mem_addr). We are taking the start output_addr and adding the h0 to h7 and inputting
those values to the mem_addr. We do this by making a case statement for each
h(number) and using a counter to increment and loop through write until all the h0
through h7 values are added into the mem_addr. Once all is done go back to IDLE.

### Optimizing the SHA-256 RTL model

One way of optimizing the SHA-256 model was to change the 'Word Expansion' algorithm. Originally, we needed to create an array of 64 words to use for the 64 rounds of compression. However, we can modify the algorithm so that we ever only need an array of 16 words. This is because when calculating a new word for the expansion, we only every use 16 of the 64 words to calculate. Another reason the original algorithm needed an array of 64 words was that it did 'Word Expansion' and 'Compression' separately. Thus it needed to calculate every new word before moving on to the actual compression. By doing one round of both at a time, we only ever need the new word to do the compression. This optimization frees up alot of space wasted by the original algorithm.

(Insert comparison of original algorithm and new algorithm)

### Developing the bitcoin hashing RTL model

![Bitcoin RTL model](/projects/assets/img/bitcoin-model.JPG)

For the bitcoin hashing algorithm, we used a modified version of the simplified SHA256
algorithm. We introduced three different phases into our algorithm. The first phase is
used to compute the hash values of the first 16 words. Then we moved onto phase two
and three, which involves the hashing of the final 3 words in the message as well as the
nonce. In phase 2, we calculate the hash output of the second block using the hash
output of the first block or phase. The output of the second phase is then passed to the
third phase, where we calculate new hash values using the initial constants.

In the IDLE state, we initialize all of our variables to appropriate values. Both the hash
values as well as a-h are set to the correct constants. Here, we also initialize several of
our counter variables (i, j, nonces, nonces_p3) to 0. If start ==1, then we transition to the
READ_BUF state.

In the READ_BUF state, we need to wait one cycle in order to read from the memory
address so we use this state to wait one cycle before going into the READ state.

In the READ state, we are using ( i ) as a counter for the amount of words which in total
is 19 words. we are assigning the mem_read_data into the message for each word and
incrementing i by 1 and assigning mem_addr to be the next mem_addr by incrementing
by i + 1. Once all the mem_addrs are read into mem_read_data we move on to the next
state BLOCK.

In the BLOCK state, we set the values of the w[16] array to use in the COMPUTE state.
If j < num_blocks - 1 or phase 1, the first 16 words are assigned to w. Then we call the
COMPUTE state immediately. If we are not in the first block, then we start assembling
our blocks with the 16 nonces. In this state, we have 2 different counters, nonces and
nonces_p3. We first assemble the second block with the last three words of message,
the nonce itself, the padding, and the ending message size. Then we increment nonces
by 1 and call the COMPUTE state. This is phase 2 in our hashing algorithm. After the
COMPUTE state, we prepare hash values for phase 3. The initial hash values for phase
3 are set to the initial constant hash values from the IDLE state. The first 8 w values are
set to the previous hash values from phase 2. Then padding and message size is added
to the rest of the w values.Then nonces_p3 is incremented by one and the COMPUTE
state is called. When the algorithm returns to the BLOCK state after the COMPUTE
state, phase 2 for the next nonce is started. The algorithm will loop through phase 2 and
3 for each nonce until both nonces and nonces_p3 are both 16. At this point, all of the
nonce values have been computed and we go to the WRITE state.

In the COMPUTE state, we do 64 rounds of compression where the first step is word
expansion. In the first 16 iterations of the compression, wt is simply the first 16 words.
For the other iterations, we calculate wt using the current values in w[16] from the
BLOCK state. The next step in compression is the actual sha_256 operation where we
input the values set in the BLOCK state (an,bn,cn,dn,en,fn,gn,hn) along with wt and the
current iteration value i. The result of sha_256 is then put into the same an - hn values
to use in the next iteration. After 64 iterations are done, the hash values are overwritten
with their current value + an - hn. If we are calculating the first block, we save the final
hash values into h0 - h7 so we can use them in calculating block 2. If not, we simply
store the final h0 value of the current nonce into hout[nonce - 1].

In the WRITE state, it is similar to the SHA256 where we have to write to mem_addr
with output_addr + i where ( i ) is the h0 values of the nonces. So we increment each
cycle to increment the different nonces where we would add h0[0], h0[1], …, h0[15] into
the mem_addr. Once this is done, we would go to IDLE.

### Optimizing the bitcoin hashing RTL model

**Under Construction**