# projectsato
Project Sato - C2

This project is way too big for me to do it myself - so i thought to just release this design.

The smart contract has only two return functions:
- Snapshot of the merkle tree to save for the client 
- Insert/update .onion address
- Insert/update contract address (must be the same logic as application)

Smart contract ZKP functions & helpful code:
- RISC Zero Rust Starter Template
    - Risc0-circuits
    - Risc0-prover
    - Risc0-types
- TC Solidity source code
- Emit events for proof verified only
- Hyperion (QRL, Solidity based) support

Onion:
- Transaction Relay source code
- Communicate with clients & anti-spam/DoS techniques
- Post-quantum secure communications
- Zero-knowledge proof verifications
- Public/private keys for the host and clients

Application:
- Custom malware functions
- Ethereum, Solana, QRL supported transaction types
- PoW anti spam / DoS
- Verify commands from host (and host verifies from client)
- Listens to contract

Quantum resistance:
- Smart contract is written in Solidity/Hyperion to make it compatible with EVM - Solana and QRL to make the transition to quantum resistance smooth (if necessary in the future)
- Bidirectional (post-quantum secure) communication between the host and the clients uses Kyber, Dilithium and AES-256 

Note: C2 server is behind Tor and hosted with the official opsec recommendations

Flow of the protocol

Phase 1 – Your Software (user side)
- App is available on users device
- LOCALLY: The app:
    - Generates a wallet
    - Has a built-in public key of botnet
    - Wallet verification: signature = sign("...", walletPrivKey) (to verify they have access to their wallet)
    - Signs a transaction with the wallet to send via POST request to reach relay and transmit to RPC node:
        - Instructions of transaction:
            - For submission: hash(nullifier || pubkey) to contract address
        - Checklist of instructions transaction within the application:
            - Does this transaction send to the correct contract address?
            - Is the submit a hash?
            - If it is all YES -> Sign!
- HTTPS (Onion Website):
    - Sends a POST request to your website with:
        - {
        -   "walletAddress": "0x...",
        -   “publicKeyBotnet”: “0x...”,
        -   “walletSignature": "0x…”,
        -   “signedTransaction”: “…”
        - }

Phase 2 – Transaction Relay Onion Website (backend)
- Website receives POST request at Transaction Relay:
    - Checklist of instructions transaction:
        - Does this transaction send to the correct contract address?
        - Is the submit a hash?
        - Does this transaction not try to drain the wallet?
        - If it is all YES -> Sign!
- Send relaySignedTransaction to RPC node.

Phase 3 - Solidity Smart contract
- Hash of the Nullifier and Public Key is inserted into the Merkle tree
- Once the user’s Hash of Nullifier and Public Key is inserted into the Merkle tree, the Merkle tree’s root is calculated, and this becomes a snapshot of the tree at that point in time.
- This snapshot (the Merkle root) is returned to the user. It essentially represents the entire state of the tree at the time of their registration.
- The Merkle root is a compact representation of the entire tree, which allows the user to prove they are part of the tree (without needing the entire tree) by submitting the relevant hashes (the proof) in the voting process.
- IMPORTANT VERIFICATION (POST-REGISTRATION) - Verify proof & access string:
    - When the user wants to access the string (e.g. onion address), which requires authentication in the contract, they submit a ZKP proving that they are part of the snapshot.
    - The user has a valid zero-knowledge proof that verifies that they belong to the snapshot.
    - Once the ZKP is verified, the contract allows the verified user to access the string.
    - In order to do this, the client needs a listener and the contract needs to emit an event.

Phase 4 - Post-quantum secure communications between Onion C2 & clients:
- Kyber, Dilithium, AES-256 encrypted communications
- The malware has logic that looks for specific conditions or patterns in the server's response in order to be executed
- Only the host can send commands from x onion address (verified from the smart contract) with:
- CMD:<command> | SIG: <signature>
- The signature the bot verifies with the hosts public key before execution
- When the client connects with their zero-knowledge proof, the server returns a challenge and difficulty
- Client bruto-forces the correct nonce
- Client submits the correct nonce
- For each guess, the client checks:
    - sha256(challenge + nonce)
- If success: respond with command/data
- If failure or too fast: increase difficulty (difficulty *= 2)
- Cooldown over time: Server resets difficulty to base if enough time passes.
    - if time_since_last_seen > 60 mins:
    -     difficulty = BASE_DIFFICULTY
- Optionally Sliding Dificulty Window, by decaying difficulty gradually:
    - difficulty = BASE_DIFFICULTY + log2(spam_score)
    - spam_score -= decay_rate * (now - last_seen)

-----------------
Advantages:
- Censorship resistance through smart contracts & modular clients
- Post-quantum secure C2 communications
- Smart contract adaptability to post-quantum secure smart contract
- Anonymity for the host/client through Tor & anonymous wallets
- Taken-down onion address doesn't mean the death of the botnet (onion can be updated to regain connection with the clients)
- Authentication through censorship resistant zero-knowledge proofs
- Anti-spam / DoS for the host with PoW challenges (potentially) from the clients
- Difficult to get to the gas fees or drain gas fees of the host
- Very difficult/impossible to see the total number of clients
- Not linkable to registration wallets or actions of the wallet and actions to a specific user.
- Blockchain metadata privacy (from the zero-knowledge proofs or relay)
- Additional customization may be done such as obfuscation techniques, etc. as of right now the source code does not provide this by default and there are no plans to add these at all.

Limitations:
- Tor timing attacks (might reveal the host/you) unless the VPS is not linkable to you. The longer your host is down for whatever reason may expose the host server ip because the adversary can build up a profile or pattern. 
- Compromised relay might cause fake registrations/clients. Which means the malicious client will always be able to follow you - as in always get an updated onion or contract address.
- Compromised host can drain your gas fees. This can be mitigated by limiting the amount of gas fees on your account.
- Contract vulnerabilities (not audited)
- Deposit(s) of gas fees is linkable back to you - unless Tornado Cash is being used to remove the link for example. 
- Predictable ways the host/attacker infects computers. If the adversary knows this and if they get their hands on a single client - they gain a lot of information from it (by e.g. reverse engineering, testing the c2, DDosing the c2 (to do a Tor timing attack), etc. An understanding of how your specific malware works in general in order to build out (possible new) attack vectors. 
- Having millions of clients can raise red flags from the spike in Tor users. 
- Higher gas fees if the number of clients go higher. Might be mitigated by mining cryptocurrencies in clients. 
- No mining to fund the gas fees in the current source code.

Upcoming protocol design update features (soon):
- Dont include public key from the user, send signature to relay and with users signature for registration and relays signature (or message), send it to the RPC node. This will make sure the relays public key is only seen by the node.
- Clients send signatures to the relay, from there they get in a batch (mixed with dummy registrations), then the contract checks whether its real or dummy, slight delay in seconds, and it inserts it into the merkle tree, and then updates the merkle root only once (also saving gas fees)
