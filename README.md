# SCP-Lite

**Educational secure client-server communication protocol created by Ahmad Araja (ARAJAA).**

SCP-Lite demonstrates a complete encrypted TCP session in Python: RSA-OAEP transports a fresh AES session key, AES-GCM protects authentication and application messages, and length-prefixed framing preserves message boundaries over TCP.

> SCP-Lite is an educational prototype, not a production protocol. The server public key is not certificate-authenticated, the shared token is embedded in source, and replay protection is incomplete.

## Protocol architecture

![SCP-Lite protocol sequence](SCP-LiteDiagram.png)

## Protocol flow

1. Server generates an ephemeral RSA-2048 key pair and listens on TCP.
2. Client connects and receives the server public key in PEM format.
3. Client generates an AES-128 session key and encrypts it with RSA-OAEP and SHA-256.
4. Server decrypts the session key with its RSA private key.
5. Client sends the pre-shared authentication token protected by AES-GCM.
6. Server verifies the token before accepting an application message.
7. Client and server exchange AES-GCM ciphertexts using a fresh 12-byte nonce for each encryption.

Every binary field is transported using a four-byte, big-endian length prefix:

```text
[4-byte length][data bytes]
```

## Security properties demonstrated

| Goal | Mechanism |
|---|---|
| Session-key confidentiality | RSA-OAEP with SHA-256 |
| Message confidentiality | AES-GCM with a 128-bit key |
| Message integrity | AES-GCM authentication tag |
| Client knowledge check | AES-GCM encrypted pre-shared token |
| Nonce freshness | New random 12-byte nonce per encryption |
| TCP framing | Four-byte big-endian length prefix |

## Repository structure

```text
.
|-- client.py          # Connects, establishes a key, authenticates, and exchanges data
|-- server.py          # Listens, creates RSA keys, authenticates, and exchanges data
|-- crypto_utils.py    # RSA, AES-GCM, framing, and exact-receive helpers
|-- test_crypto.py     # Local cryptographic round-trip check
|-- protocol.md        # Detailed protocol specification
|-- SCP-LiteDiagram.png
|-- requirements.txt
`-- README.md
```

## Requirements

- Python 3.10 or newer
- `cryptography`

```bash
python -m pip install -r requirements.txt
```

## Run

Start server in first terminal:

```bash
python server.py
```

Start client in second terminal:

```bash
python client.py
```

Default endpoint:

```text
127.0.0.1:9999
```

## Test cryptographic primitives

```bash
python test_crypto.py
```

Expected final line:

```text
Test Passed: Encryption, decryption, and authentication token work correctly.
```

The test confirms RSA session-key round-trip, AES-GCM message round-trip, and authentication-token round-trip. It is not a complete security proof or network integration test.

## Trust boundary and limitations

Current design does not provide:

- Certificate validation or authenticated server identity
- Protection against an active man-in-the-middle during initial key delivery
- Mutual authentication
- Secure secret provisioning or persistent key storage
- Complete replay protection
- Multi-client concurrency
- Production-grade error handling, logging, or protocol negotiation

For real systems, use a reviewed protocol such as TLS rather than designing a custom transport protocol.

## Documentation

Read [protocol.md](protocol.md) for message order, algorithms, framing, authentication, limitations, and planned improvements.

## Author

Created by **Ahmad Araja (ARAJAA)** for educational study of secure communication protocols.

[Connect with me on LinkedIn](https://www.linkedin.com/in/ahmadaraja)

## Authorized use

Use this project only for education, local experimentation, and authorized security research.

## License

SCP-Lite source code and documentation are licensed under the [MIT License](LICENSE).
