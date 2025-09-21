# Vault 
You can then download any required build tools by bootstrapping your environment:

```sh
$ make bootstrap
...
```

To compile a development version of Vault, run `make` or `make dev`. This will
put the Vault binary in the `bin` and `$GOPATH/bin` folders:

```sh
$ make dev
...
$ bin/vault
...
```

## Testing Kyber Post-Quantum Encryption

Vault now supports Kyber, a NIST-standardized post-quantum Key Encapsulation Mechanism (KEM) for quantum-resistant encryption. Here's how to test the Kyber functionality:

### Prerequisites

1. Start Vault in development mode:
```sh
$ vault server -dev
```

2. Set your Vault token (use the token displayed when starting dev server root token):
```sh
$ export VAULT_TOKEN="your-dev-token-here"
```

### API Examples

#### 1. Enable Transit Engine
```bash
curl -X POST \
    http://127.0.0.1:8200/v1/sys/mounts/transit \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"type": "transit"}'
```

#### 2. Create Kyber Keys

Create keys with different parameter sets (security levels):

**Kyber-512** (~128-bit quantum security):
```bash
curl -X POST \
    http://127.0.0.1:8200/v1/transit/keys/kyber-512-key \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"type": "kyber", "parameter_set": "kyber-512"}'
```

**Kyber-768** (~192-bit quantum security):
```bash
curl -X POST \
    http://127.0.0.1:8200/v1/transit/keys/kyber-768-key \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"type": "kyber", "parameter_set": "kyber-768"}'
```

**Kyber-1024** (~256-bit quantum security):
```bash
curl -X POST \
    http://127.0.0.1:8200/v1/transit/keys/kyber-1024-key \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"type": "kyber", "parameter_set": "kyber-1024"}'
```

#### 3. Encrypt Data

```bash
# Encrypt plaintext data (base64 encoded "Hello World")
curl -X POST \
    http://127.0.0.1:8200/v1/transit/encrypt/kyber-512-key \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"plaintext": "SGVsbG8gV29ybGQ="}'
```

#### 4. Decrypt Data

```bash
# Decrypt the ciphertext (replace with actual ciphertext from encrypt response)
curl -X POST \
    http://127.0.0.1:8200/v1/transit/decrypt/kyber-512-key \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"ciphertext": "vault:v1:your-ciphertext-here"}'
```

#### 5. Key Operations

**Get key information:**
```bash
curl -X GET \
    http://127.0.0.1:8200/v1/transit/keys/kyber-512-key \
    -H "X-Vault-Token: $VAULT_TOKEN"
```

### Parameter Sets Comparison

| Parameter Set | Security Level | Public Key Size | Ciphertext Overhead | Performance |
|---------------|----------------|-----------------|-------------------|-------------|
| kyber-512     | ~AES-128       | 800 bytes       | ~768 bytes        | Fastest     |
| kyber-768     | ~AES-192       | 1,184 bytes     | ~1,088 bytes      | Balanced    |
| kyber-1024    | ~AES-256       | 1,568 bytes     | ~1,568 bytes      | Most Secure |
