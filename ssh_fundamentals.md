# SSH Fundamentals

## What is SSH?

SSH (Secure Shell) is a network protocol for securely accessing and controlling a remote machine over an unsecured network (like the internet). It encrypts the entire session — commands, passwords, and any data transferred — so nothing can be read in transit by someone snooping on the network.

At a basic level, here's what happens when you connect:

1. A client connects to a server on **port 22**.
2. The client and server negotiate which encryption algorithms to use.
3. The server proves its identity to the client.
4. An encrypted tunnel is established over the connection.
5. The client authenticates (proves who *it* is) — either with a password or, more securely, a key pair.

---

## Ports: What Are They?

A port is a number that identifies a specific service running on a machine, so multiple programs can share one network connection without interfering with each other.

**Analogy:** an IP address is like a building's street address. A port is like an apartment number inside that building. The IP gets you to the building; the port tells you which door to knock on. A single server can run a web server, an email server, and an SSH server simultaneously — all reachable at the same IP, but on different ports.

Port numbers range from 0–65535. Common reserved ports:

| Port | Service |
|------|---------|
| 22   | SSH |
| 80   | HTTP (web traffic) |
| 443  | HTTPS (encrypted web traffic) |
| 25   | SMTP (email) |
| 3306 | MySQL |

When an SSH client connects to "port 22," it's specifically reaching the SSH daemon (`sshd`) — the program listening for SSH connections — not any other service running on that machine.

Firewalls work on this same logic: you can block or allow traffic per port (e.g., open port 22 for SSH while keeping everything else closed).

---

## IP Addresses: Servers Have Them Too

Every machine reachable over a network — including servers — has an IP address. It's how other machines find it and send it data.

A few practical notes:

- **Cloud servers** (e.g., AWS EC2) typically get a **public IP** (reachable from anywhere on the internet) and often a **private IP** (for communication within the same internal network, like an EC2 instance talking to a database in the same VPC).
- **Domain names** (like `google.com`) are human-friendly aliases for IP addresses. **DNS** is the system that translates a domain name into the actual IP, since typing `142.250.183.14` isn't practical.
- Servers usually have a **static IP** (fixed, doesn't change), since clients need a stable address to reliably find them. Personal devices like laptops typically get a **dynamic IP** that changes depending on the network they join.

Putting it together for SSH: running `ssh user@server-ip` (or a domain name that DNS resolves to that IP) sends the connection to that address on port 22, where the handshake and authentication described above take place.

---

## Passwordless SSH (Public Key Authentication)

Instead of typing a password every time, SSH supports authentication via a cryptographic key pair.

**How it works:**

1. **Generate a key pair** locally using `ssh-keygen -t ed25519`. This creates two files:
   - A **private key** — stays on your machine, never shared with anyone.
   - A **public key** — can be shared freely; it's useless without the matching private key.
2. **Copy the public key** to the server, into `~/.ssh/authorized_keys` for the account you want to log into. The `ssh-copy-id` command automates this step.
3. **On login**, the server sends a challenge that can only be answered correctly using the private key.
4. **Your client decrypts/answers** the challenge using the private key — proving you have it, without ever transmitting the private key itself over the network.
5. If the cryptographic check passes, the server grants access. No password is typed or sent.

This is more secure than a password because the private key never leaves your machine, and it's typically far longer and more random than any password a human would choose — making it effectively impossible to brute-force. It's also why this approach is standard for things like pushing code to GitHub or SSHing into cloud servers: automated systems can't type a password, but they can hold a key file.

> Note: `ssh-keygen -t <type>` requires a valid algorithm name — `ed25519` is the modern, recommended choice. Something like `evdaa` is not a valid type and will error out.

---

## The Passphrase Prompt

When you run `ssh-keygen`, it asks if you want to set a **passphrase**. This is a separate layer of protection on top of the key pair itself.

**Why it matters:** your private key normally sits unencrypted on disk. If someone steals your laptop or gains access to your filesystem, they could grab that private key file and use it to log into every server you've set up with it — no password needed, since that's the entire point of key-based auth. A passphrase encrypts the private key file itself, so even if it's stolen, it's useless without that passphrase.

**Two reasonable options:**

- **Set a passphrase** — more secure. You'll be prompted to enter it each time you use the key, unless you run `ssh-agent` to cache it for the session (common practice).
- **Leave it empty** (press Enter twice) — fully passwordless and convenient for automation, but riskier if the key file is ever exposed.

**Rule of thumb:** use a passphrase for personal use on your own servers. Leave it empty for automated contexts like CI/CD pipelines or deploy scripts, where there's no human present to type one.

---

## Finding the `.ssh` Folder

`.ssh` is a **hidden folder** — anything starting with a dot is hidden by default on Linux/Mac, and plain `ls` skips hidden files and folders.

**To see it:**

```bash
ls -a ~
```

This lists everything in your home directory, including hidden items like `.ssh`.

**To look inside it with details:**

```bash
ls -la ~/.ssh
```

The `-l` flag shows permissions and file sizes — relevant here because SSH is strict about file permissions. Your private key (e.g., `id_ed25519`) should be readable only by you (permission `600`), or SSH will refuse to use it.

If `~/.ssh` doesn't exist yet, it simply means no key has been generated there yet — `ssh-keygen` creates the folder automatically on first use.
