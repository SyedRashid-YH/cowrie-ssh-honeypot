# Cowrie SSH Honeypot: Deployment and Attack Simulation

A hands-on lab deploying [Cowrie](https://github.com/cowrie/cowrie), an open-source SSH/Telnet honeypot, and testing it with simulated attacker activity from a separate Kali Linux machine.

> **Disclaimer:** This project was conducted in an isolated, authorized home lab for educational and defensive security purposes. The honeypot was never port-forwarded or exposed to the internet.

## Objectives

- Deploy and run an SSH honeypot on Ubuntu
- Discover the honeypot's exposed service with Nmap
- Simulate an attacker session over SSH from Kali Linux
- Verify that connections, logins, and commands are captured in Cowrie's logs

## Lab Environment

| Role | System | Notes |
|---|---|---|
| Honeypot host | Ubuntu (VMware VM) | Cowrie 3.0.15, Python 3 venv |
| Testing machine | Kali Linux (VMware VM) | Nmap, OpenSSH client |
| Network | `192.168.1.0/24` | *Update: bridged / NAT / host-only* |

Cowrie listens on **port 2222** (default), so it does not conflict with the real SSH service on port 22.

## Tools

Cowrie · Ubuntu · Kali Linux · Python 3 (venv) · Nmap · OpenSSH

## Implementation

### 1. Environment setup

```bash
mkdir cowrie && cd cowrie
python3 -m venv cowrie-env
source cowrie-env/bin/activate
```

### 2. Install Cowrie

```bash
pip install cowrie
```

### 3. Initialize and start

```bash
cowrie init     # generates etc/cowrie.cfg and the var/ directory structure
cowrie start
```

![Install, init and start](screenshots/01-install-init-start.jpg)

### 4. Scan and connect from Kali

```bash
nmap 192.168.1.16
ssh -p 2222 ubuntu@192.168.1.16
```

Nmap reported ports 443, 2222, and 8080 open. Port 2222 is Cowrie (Nmap labels it `EtherNetIP-1` because it guesses services from port numbers). Inside the SSH session, Cowrie presented a fake host (`svr04`) with a decoy `phil` home directory, and I ran `ls` and `cd` to simulate attacker reconnaissance.

![Nmap scan and SSH session](screenshots/02-nmap-ssh.jpg)

### 5. Review the logs

```bash
cd ~/cowrie/var/log/cowrie
```

Cowrie recorded the connection, login, and every command:

```text
New connection: 192.168.1.14:57228 (192.168.1.16:2222) [session: b51e68c00069]
Remote SSH version: SSH-2.0-OpenSSH_10.4p1 Debian-5
login attempt [ubuntu/...] succeeded
Initialized emulated server as architecture: linux-x64-lsb
CMD: ls
CMD: cd ..
CMD: cd phil
```

![Cowrie logs](screenshots/03-cowrie-logs.jpg)

## Results

- Deployed Cowrie and confirmed it accepts SSH connections on port 2222
- Discovered the honeypot service using Nmap
- Established a simulated attacker session from Kali Linux
- Verified that Cowrie logged the source IP, client SSH version, login attempt, and each command entered

## Key Learnings

- How SSH honeypots emulate a real system to capture attacker behavior
- Why honeypots run on non-standard ports and what Nmap does and doesn't reveal about them
- Deploying a Python service in a virtual environment
- Reading connection and command logs for security monitoring

## Limitations

- All traffic came from my own test machine, so this shows how Cowrie captures activity rather than real-world attacker behavior.
- Default Cowrie configuration was used; no custom filesystem or credentials.

## Security Notes

- Run Cowrie as a non-root user.
- Never expose a honeypot to the internet without proper isolation (dedicated network, no access to other devices).
- Cowrie logs the credentials entered, so don't use real passwords when testing.

## Next Steps

- Analyze `var/log/cowrie/cowrie.json` with Python or `jq`
- Simulate brute-force attempts (e.g., Hydra) and review the login logs
- Build a simple visualization of sessions, commands, and source IPs
- Test with `nmap -sV -p 2222` for service fingerprinting
