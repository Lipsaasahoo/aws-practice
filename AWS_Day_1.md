# AWS DevOps Practical Series – Day 1: Launch an EC2 Instance and Connect via SSH

## 🎥 Practical Demo — Watch on YouTube

[![Day 1 - Launched an EC2 Instance and Connected via SSH](https://img.youtube.com/vi/Mri6HcK5iQk/hqdefault.jpg)](https://www.youtube.com/watch?v=Mri6HcK5iQk)

## 1. Summary

| Item | Detail |
| --- | --- |
| Day | 1 |
| AWS Service | Amazon EC2 (Elastic Compute Cloud) |
| Goal | Launch a Linux virtual machine and connect to it over SSH |
| Instance name | ec2-1 |
| Instance type | t3.micro |
| Key pair | ec2-1-kp |
| Default username | ec2-user (Amazon Linux) |
| Connection port | 22 (SSH) |
| Tools used | AWS Management Console, MobaXterm |

## 2. Objective

To understand how a virtual server is created in AWS and how an engineer securely logs in to it from a local machine. This is the foundation for every later topic in the series (networking, storage, load balancers, automation and so on).

## 3. What is EC2?

Amazon EC2 provides resizable virtual servers in the cloud. Instead of buying hardware, you choose an operating system image (AMI), a size (instance type) and a login method (key pair), and AWS gives you a running machine in a few minutes. You pay only for the time it runs.

## 4. Prerequisites

- An AWS account with permission to use EC2
- MobaXterm installed on the local machine (optional if using the console connect method)

## 5. Part A: Launching the EC2 instance

1. Log in to the **AWS Management Console**.
2. In the search bar, type **EC2** and open the EC2 dashboard.
3. Click **Launch instance** (Create instance).
4. **Name:** enter `ec2-1`.
5. **AMI (Application and OS Image):** select the AMI (for example Amazon Linux). Choose a *Free tier eligible* image where shown.
6. **Instance type:** select `t3.micro`.
7. **Key pair (login):** click **Create new key pair**.
   - Name: `ec2-1-kp`
   - Key pair type: RSA
   - Private key file format: `.pem` (works with MobaXterm and OpenSSH)
   - Click **Create key pair**. The file downloads automatically. **Store it safely. It cannot be downloaded again.**
8. Leave **Network settings**, **Storage** and **Advanced details** at their defaults.
9. Click **Launch instance**.
10. Open **Instances** and wait until **Instance state = Running** and **Status check = 2/2 checks passed**.


## 6. Part B: Connecting to the instance

There are two ways to connect.

### Method 1: From the AWS Console (browser)

1. Go to **EC2 → Instances**.
2. Select `ec2-1`.
3. Click **Connect**.
4. Choose the **EC2 Instance Connect** tab, confirm the username `ec2-user` and click **Connect**.
5. A browser-based terminal opens.

### Method 2: From the local machine using MobaXterm (SSH)

1. Select the instance in the console and copy its **Public IPv4 address**. The **Connect → SSH client** tab also shows the username.
2. Open **MobaXterm**.
3. Click **Session → SSH**.
4. **Remote host:** paste the public IP address.
5. Tick **Specify username** and enter `ec2-user`.
6. **Port:** 22.
7. Open **Advanced SSH settings**, tick **Use private key** and browse to the `ec2-1-kp.pem` file.
8. Click **OK**. A terminal opens and you are now logged in to the EC2 machine over SSH.

### Quick verification commands

```
whoami          # shows ec2-user
hostname        # shows the private DNS name
uname -a        # shows the Linux kernel info
df -h           # shows disk usage
```

## 7. What is a Key Pair and why do we need it?

**In one line:** a key pair is the digital "lock and key" that proves who you are when you log in to your EC2 instance, without using a password.

### The two halves

- **Public key:** stored by AWS on the instance (in `~/.ssh/authorized_keys`). Think of it as the **lock**. It is safe for others to see.
- **Private key:** the `.pem` file you downloaded. Think of it as the **key**. Only you should have it. AWS does not keep a copy.

### How the login works

1. Your SSH client (MobaXterm) connects to the instance on port 22 and says "I am ec2-user".
2. The server challenges the client to prove it owns the private key matching the stored public key.
3. The client signs the challenge with the private key. The private key itself is **never sent** over the network.
4. The server verifies the signature using the public key. If it matches, you are logged in.

### Why not just use a password?

- Passwords can be guessed or brute-forced; a 2048-bit RSA key practically cannot.
- Nothing secret travels over the network.
- It is the standard, more secure approach for servers and enables automation (scripts, Ansible, CI/CD) without typing passwords.

### Important rules

- **Never share or upload the .pem file** (not to GitHub, not in a video, not in chat).
- **If you lose it, you cannot download it again.** You would have to create a new key pair and use another recovery method to regain access.
- On Linux/Mac, restrict permissions: `chmod 400 ec2-1-kp.pem`. SSH rejects keys that are readable by others.
- A key pair is chosen at launch. One key pair can be reused for many instances.
- Format note: `.pem` is for OpenSSH and MobaXterm; `.ppk` is for PuTTY.

## 8. How the pieces fit together

| Component | Role in this lab |
| --- | --- |
| AMI | The operating system image the server boots from |
| Instance type (t3.micro) | The CPU and memory size of the server |
| Key pair (ec2-1-kp) | Authentication: proves you are allowed to log in |
| Security group | The firewall. The default launch wizard rule allows inbound SSH on port 22 |
| Public IPv4 | The address you connect to from outside AWS |
| SSH (port 22) | The encrypted protocol used to get a remote terminal |

## 9. Troubleshooting

| Problem | Likely cause | Fix |
| --- | --- | --- |
| Connection timed out | Security group does not allow port 22 from your IP, or instance not running | Check inbound rules, confirm Running state |
| Permission denied (publickey) | Wrong username or wrong key file | Use `ec2-user` for Amazon Linux (`ubuntu` for Ubuntu) and select `ec2-1-kp.pem` |
| Unprotected private key file | File permissions too open | `chmod 400 ec2-1-kp.pem` |
| IP address changed after stop/start | Public IPs are not static by default | Copy the new IP, or use an Elastic IP later |

## 10. Best practices noted

- Restrict the SSH rule in the security group to **My IP** instead of `0.0.0.0/0` for real use.
- Keep the private key in a secure folder and back it up safely.
- Use clear naming (`ec2-1`, `ec2-1-kp`) so resources are easy to track.

## 11. Cleanup (avoid surprise charges)

1. Go to **EC2 → Instances**, select `ec2-1`.
2. Click **Instance state → Terminate instance**.
3. Optionally delete the key pair under **EC2 → Key Pairs** (this removes only AWS's public copy).

> Stopping an instance is not the same as terminating it. Stopped instances still incur storage charges.

## 12. Key takeaways

- EC2 gives you a virtual server in minutes.
- A key pair (public key on the server, private key with you) is how AWS authenticates SSH logins.
- You can connect through the browser console or from your own machine with an SSH client.
- SSH runs on port 22 and needs the security group to allow it.

