## The client verifies the server's host key
When you SSH into a server, the server also presents its own public key, called the host key. SSH shows the host key's fingerprint so you can verify the server's identity and **avoid man-in-the-middle attacks**. Accepting the host key adds it to your known_hosts file for future trust. 
  
![first-connect-host-public-key](./screenshots/first-connect-host-public-key.png)

- **Host key fingerprint**: A hashed summary of the server's host key (e.g., `SHA256:lLzhZc7YzRBDchm02qTX0qsLqeeiTCJg5ipOT0E/YM8`) shown the first time you connect.  
- **Known hosts**: When you accept the host key (`yes`), it is stored in `~/.ssh/known_hosts` to trust that server in future connections.

## Server authenticates using the client's private key

### 1) What SSH keys are and why use them

- SSH key authentication replaces username/password logins with public/private key pairs.
- The private key stays on the client machine; the public key goes to the remote server. This is more secure than password authentication.

### 2) Generating keys

Generate keys on your own machine (so the private key never exists on the target):

```bash
ssh-keygen -t rsa -b 4096 -C "<your_email@example.com>"
```

- `-t` rsa chooses the RSA algorithm (RSA is the historical default).
- `-b` 4096 increases key size for stronger security.
- You'll be prompted for a passphrase to encrypt the private key - strongly recommended.

![ssh-keygen](./screenshots/man-ssh-keygen.png)
![ssh-keygen-example](./screenshots/ssh-keygen-example.png)

### 3) Copying the public key to the server

Use `ssh-copy-id` to append your public key to the server's authorized_keys safely:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
```

Or manually append:

```bash
cat ~/.ssh/id_rsa.pub | ssh user@host 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

### 4) File locations & permissions

- Client private key: `~/.ssh/id_rsa` (or other name you chose).
  - Permissions: **only owner** should read/write: `chmod 600 ~/.ssh/id_rsa`
- Client .ssh directory: `chmod 700 ~/.ssh`
- Server trusted keys: `~/.ssh/authorized_keys`
  - Ensure `~/.ssh` is `700` and authorized_keys is `600` on the server. If permissions are too permissive, OpenSSH may **ignore** the key and warn you.

### 5) Using a particular private key with ssh

```bash
ssh -i /path/to/privateKeyFile user@host
```

### 6) Changing or adding a passphrase

Add/change passphrase for an existing private key:

```bash
ssh-keygen -p -f ~/.ssh/id_rsa
```

### 7) Checking what algorithm a key uses

- For a **private** key file, the header can indicate the type:
  - `-----BEGIN RSA PRIVATE KEY-----` → RSA
  - `-----BEGIN OPENSSH PRIVATE KEY-----` → newer OpenSSH format (still may contain an RSA key inside)
- To print public-key details (fingerprint, bit length, algorithm):
  - `ssh-keygen -lf /path/to/public_or_private_key`

Example: `ssh-keygen -lf ~/.ssh/id_rsa.pub` will show the bit size and fingerprint; algorithm is typically shown implicitly (RSA, ECDSA, ED25519, etc.).

### 8) SSH keys in CTFs / pentesting

- If a user account on a target has an authorized public key you control, you can SSH in and get a stable, interactive shell (better than an unstable reverse shell).
- Note: some service accounts (e.g., www-data) may be restricted from SSH logins; regular users and root generally work if SSH login is enabled.
- Leaving a public key in `authorized_keys` acts as a persistent backdoor if you control the corresponding private key.

### 9) Security recommendations

- Use a passphrase for private keys and store them securely (use an agent like ssh-agent or a hardware token for frequent use).
- Disable password authentication for SSH if feasible and enforce key-only authentication, especially for root access.
- Prefer modern algorithms where appropriate (e.g., ed25519) for smaller keys with strong security, unless RSA is required for compatibility.

### Task Questions

**Q:** Check the SSH Private Key in ~/Public-Crypto-Basics/Task-5. What algorithm does the key use?

![rsa-key](./screenshots/answer-ssh-rsa-key.png)

**Answer:** RSA
**Recommended Approach:** Use `ssh-keygen -lf /home/user/Public-Crypto-Basics/Task-5/id_rsa_1593558668558.id_rsa` to identify the key algorithm.
