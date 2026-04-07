# Cryptographic Hash Functions & Integrity
### What is a Hash Function? (The Digital Fingerprint)

A hash function is a mathematical algorithm that maps an input of any size to a fixed-length output (the hash or "digest").

- Key Properties:

    - Deterministic: The same input will always produce the exact same hash.

    - Efficiency: It is computationally fast to generate the hash for any given data.

    - Pre-image Resistance: It is computationally infeasible to "reverse" a hash to find the original input.

    - Avalanche Effect: A change in even a single bit of the input results in a completely different and uncorrelated hash.

## Why Do We Care? (Use Cases)

- Use Case,Real-World Example,Benefit

    - Integrity Verification,Downloading a Linux ISO file.,Ensures the file wasn't corrupted or modified by a Man-in-the-Middle (MitM).

    - Password Hashing,Storing $argon2id$... in a database.,"Protects users if the database is leaked; attackers only see hashes, not plaintext."

    - Digital Signatures,Signing a security audit report.,Proves that the document has not been altered since it was signed.

The Vulnerability: Length Extension Attacks (LEA)

Many hash functions (such as MD5, SHA-1, and the SHA-2 family) rely on the Merkle–Damgård construction. This structure allows an attacker to append new data to a message without knowing the original secret prefix, provided they know the original hash and the message length.

xploitation Scenario:

Imagine a web application that authenticates users by hashing a SECRET_KEY + USER_DATA.

    Original State: hash(SECRET + "user=victor&role=guest") → Produces abc123...

    The Attack: An attacker intercepts the hash and uses a Length Extension Attack tool to calculate a new valid hash that includes &role=admin.

    The Result: The server receives hash(SECRET + "user=victor&role=guest" + PADDING + "&role=admin").

    Impact: The server validates the hash as correct, granting the attacker Administrative privileges.

## Advanced Summary: Hash Security & Applications

### The Three Pillars of Hash Security

For a hash function to be considered "Cryptographically Secure," it must satisfy these three properties:

- Property,Definition,Real-World Context
Pre-image Resistance,"It is a ""one-way street."" You cannot derive the input from the hash.","Protecting Passwords. Even if a hacker steals the database, they don't see ""P@ssword123."""

- Second Pre-image Resistance,"Given a specific input, you cannot find another input that produces the same hash.",Protecting Software Updates. Prevents an attacker from creating a malicious update that matches the hash of the official one.

- Collision Resistance,It is impossible to find any two different inputs that result in the same hash.,Protecting Digital Certificates. Ensures two different websites can't share the same digital identity.

### Where do we use them? (Practical Examples)
1. Integrity Verification (Checksums)

    Example: When you download a tool like Burp Suite, the vendor provides an SHA-256 hash.

    Action: You run sha256sum burp_install.sh on your terminal. If the output matches the website, the file is safe. If one bit was changed by a Man-in-the-Middle, the hashes will be completely different.

2. Secure Password Storage

    Example: A website never knows your password. It only knows that hash("Victor123") = "xy78...".

    Action: When you log in, the server hashes your attempt. If the hashes match, you're in. This limits the blast radius of a Data Breach.

3. Digital Signatures

    Example: When you send a signed email or a Bitcoin transaction.

    Action: The system hashes the message and encrypts that hash. This proves that the message came from you and hasn't been altered.

#### How Hash Functions Process Data
1. Block-by-Block Processing

Cryptographic hashes like MD5, SHA-1, and SHA-256 don't process a file as one giant unit. Instead, they divide the data into fixed-size chunks (typically 512 bits).
2. The Necessity of Padding

If your data isn't a perfect multiple of the block size, the algorithm uses Padding. This isn't just "junk" data; it follows a strict mathematical rule:

    Alignment: A "1" bit is added, followed by "0" bits to fill the block.

    Length Encoding: The very last 64 bits of the final block are reserved to store the original message length.

### Example: The 512-bit Block

Imagine a message that is exactly 448 bits long. To reach the required 512-bit block size:

    Data: 448 bits.

    Separator: +1 bit (the "1").

    Filler: +63 bits (the "0"s).

    Total: 512 bits.

     Security Note: If the message + padding exceeds the block limit, the algorithm simply creates a new block to hold the length encoding.

3. Internal States & Registers

As each block is processed, the hash function updates its Internal State—a set of fixed-size values called registers.

    MD5 uses 4 registers (A, B, C, D).

    SHA-256 uses 8 registers (A through H).

Think of the internal state as a running total. Each block "scrambles" the registers using bitwise operations (AND, OR, XOR, Shifts). Once the last block is processed, the final values in these registers are the hash.

- Feature: 

    - MD5: 
    Status: Broken (Collisions found)
    Block Size: 512 bits
    Internal State: 128bits
    Rounds: 64 rounds


    - SHA-256: 
    Status: Secure (Industry Standard)
    Block Size: 512 bits
    Internal State: 254 bits
    Rounds: 64 rounds

The "Vulnerability" Connection: Length Extension

Because these functions update a "state" and move to the next block, an attacker who knows the final hash actually knows the final state of the registers.

- By starting a new block with that "final state," an attacker can add their own data and continue the hashing process as if they were the original sender. This is the foundation of the Length Extension Attack you are documenting in your repo.
Security+ Tip (Domain 2.0)

- The exam might ask about Hashing Salts. While padding is a mathematical requirement for the algorithm to function, Salting is a security measure added before hashing to prevent Rainbow Table attacks. Don't confuse the two!

![Hash Function Data Processing, Padding, and State Updates](assets/hash_processing.png)
Diagram illustrating three stages of SHA hash function processing: (1) Message Padding showing 512-bit block alignment with 448-bit message data, 1-bit separator, 0-bit filler, and 64-bit length encoding; (2) Block-by-Block Processing depicting padded blocks entering hash rounds of bitwise and XOR operations with state updates from State N to State N+1; (3) Internal State Updates showing State 0 registers A, B, C, D progressing through State N after processing blocks, culminating in final hash State N+1. Technical diagram with blue, green, and gray color coding for data components and arrows indicating data flow.

### Why it's vulnerable:
 MDS's internal state and padding are predictable, making it vulnerable to attacks like length extension, where an attacker can append data to a message and still generate a valid hash. This hash function is also obsolete for security-sensitive applications due to collision vulnerabilities.

 ### SHA-1

    Block size: 512 bits
    Internal state: 160 bits, split into 5 words (A, B, C, D, E)
    Rounds: SHA-1 processes each block through 80 rounds of transformations, much like, but with more complexity.
- ‘This architecture ensures that even the slightest alteration to the 512-bit input block results in a completely different 160-bit hash, due to the avalanche effect generated across the 80 transformation rounds.’
![SHA-1 Hash Function Architecture showing 512-bit block, 160-bit state (words A-E), and 80 rounds of processing](assets/sha1_architecture.png)

![SHA-1 Hash Function Architecture displaying a 512-bit input block containing 16 message words (W0 through W15) that expand into 80 rounds of message words through a message schedule expansion process. The architecture shows a 160-bit internal state comprising five 32-bit registers labeled A, B, C, D, and E. Each of the 80 rounds performs bitwise operations including shifts, XOR, and a non-linear function (f) combined with round constants (k), updating the state sequentially until the final hash output emerges from the five state registers after all 80 rounds complete. Technical diagram with blue, green, and gray color coding showing data flow from input block through expansion, processing rounds, and final hash generation.]

