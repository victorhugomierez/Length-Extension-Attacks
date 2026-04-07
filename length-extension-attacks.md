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

