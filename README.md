# DPQ Chat

A secure peer-to-peer chat application built with Rust that implements post-quantum cryptography to protect against both classical and quantum computer attacks. The system uses CRYSTALS-Dilithium for digital signatures and CRYSTALS-Kyber for key exchange, providing quantum-resistant security for all communications.

Features a terminal-based interface with real-time messaging, decentralized architecture (no central servers), and hybrid cryptographic protocols that combine classical and post-quantum algorithms for maximum security.

> **⚠️ Development Notice**  
> This project is currently under active development. Features and APIs may change.

## Architecture Overview

**Transport Layer Security**
- Hybrid TLS using [X25519](https://datatracker.ietf.org/doc/rfc7748/) + [ML-KEM-768](https://csrc.nist.gov/pubs/fips/203/final) key exchange
- NIST-approved post-quantum algorithms via [rustls-post-quantum](https://github.com/rustls/rustls/tree/main/rustls-post-quantum)

**Application Layer Security**  
- [CRYSTALS-Kyber](https://pq-crystals.org/kyber/) for session key derivation
- [CRYSTALS-Dilithium](https://pq-crystals.org/dilithium/) for peer authentication
- [AES-256-GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode) for message encryption

**Key Characteristics**
- No message persistence - all data is volatile and destroyed on disconnect
- No password storage - passwords only used for key derivation
- Decentralized P2P mesh network with no central servers
- Terminal-based interface for maximum compatibility

## Usage Flow

### Initial Setup
1. Generate cryptographic identity using [CRYSTALS-Dilithium](https://pq-crystals.org/dilithium/) keypair
2. Encrypt private key with user password using [Argon2id](https://datatracker.ietf.org/doc/rfc9106/) + [AES-256-GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode)  
3. Store encrypted identity locally in ~/.dpq-chat/identities/

### Starting a Session
1. User enters password to decrypt private key (password never stored)
2. Application loads [CRYSTALS-Dilithium](https://pq-crystals.org/dilithium/) keypair into memory
3. Choose network interface and port for P2P listening
4. Begin accepting peer connections

### Network Flow
1. **Peer Discovery**: UDP multicast announces presence on local network
2. **Direct Connection**: Peers can connect directly via IP:PORT
3. **Mesh Formation**: Each peer maintains direct connections to all others
4. **Resilient Topology**: Network continues functioning even if original peer leaves

## Connection Flow: Handshake to Message Encryption

### Step 1: TLS Transport Establishment
```
Peer A ←→ Peer B
   ↓
X25519 + ML-KEM-768 Hybrid Key Exchange
   ↓
Encrypted TLS 1.3 Transport Channel
```

### Step 2: Application-Layer Authentication
```
Over TLS Channel:
1. A → B: CRYSTALS-Kyber Public Key + Identity Info
2. B → A: CRYSTALS-Kyber Ciphertext + Identity Info  
3. Both derive shared secret from Kyber exchange
4. A → B: CRYSTALS-Dilithium signature of (identity + kyber_data)
5. B → A: CRYSTALS-Dilithium signature of (identity + kyber_data)
6. Both verify signatures using peer's public key
```

### Step 3: Session Key Derivation
```
CRYSTALS-Kyber Shared Secret → SHA-256 → AES-256 Session Key
```

### Step 4: Message Encryption Flow
```
Plain Text → AES-256-GCM(session_key, nonce) → Encrypted Payload → TLS Transport
```

### Data Volatility and Privacy

**No Message Persistence**
- Messages exist only in memory during active session
- All chat history destroyed when peer disconnects
- No database, no logs, no message recovery possible

**No Password Storage**
- User passwords never stored anywhere on disk
- Passwords only used for real-time key derivation via Argon2id
- Private keys encrypted at rest, decrypted only when needed

**Session Isolation**
- Each peer-to-peer connection has unique session keys
- Keys expire after 1 hour and are securely erased from memory
- Past communications remain secure even if current keys compromised

## Getting Started

### Build from Source
```bash
git clone <repository-url>
cd dpq-chat
cargo build --release
```

### Generate Identity  
```bash
cargo run -- generate-key
# Enter username and password when prompted
```

### Start Chat Session
```bash
# Interactive mode (recommended)
cargo run

# Direct CLI mode
cargo run -- p2p -u <username> --host <interface>
cargo run -- p2p -u <username> -b <peer_ip:port>  # Join existing peer
```

### Example: Local Network Chat
```bash
# User A creates room
cargo run -- p2p -u Alice --host 192.168.1.100

# User B joins  
cargo run -- p2p -u Bob -b 192.168.1.100:40000
```

## License

This project is licensed under the [MIT License](LICENSE).
