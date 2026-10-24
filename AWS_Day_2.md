# AWS DevOps Practical Series – Day 2: EC2 Instance Types and Elastic IPs

## 🎥 Practical Demo — Day 2

[![Day 2 - Launched an EC2 t3.micro, Explored Instance Types, and Configured Elastic IPs](https://img.youtube.com/vi/b4ErGnGkDaQ/hqdefault.jpg)](https://www.youtube.com/watch?v=b4ErGnGkDaQ)

## 1. Summary

| Item | Detail |
| --- | --- |
| Day | 2 |
| AWS Service | Amazon EC2 and Elastic IP |
| Goal | Launch an instance, understand instance types, and see how an Elastic IP keeps a public IP fixed |
| Instance name | ec2-eip |
| Instance type | t3.micro |
| Key pair | None (SSH was not needed for this lab) |
| Concepts covered | Instance types and naming, public IP behaviour, Elastic IP allocate / associate / disassociate / release |

## 2. Objective

To prove, by experiment, that the normal public IP of an EC2 instance changes every time the instance is stopped and started, and that an Elastic IP fixes this. Also to understand how to read EC2 instance type names and choose the right family.

## 3. Part A: Launching the instance

1. Log in to the **AWS Management Console**.
2. Open **EC2** and click **Launch instance**.
3. **Name:** `ec2-eip`.
4. **AMI:** select the AMI (for example Amazon Linux, free tier eligible).
5. **Instance type:** `t3.micro`.
6. **Key pair:** *Proceed without a key pair*.
7. Leave network, storage and other settings at default. Note that a new security group is created automatically (named like `launch-wizard-1`).
8. Click **Launch instance** and wait for **Running** and **2/2 checks passed**.


### Was launching without a key pair correct?

**Yes, for this lab it was fine.** The goal was only to observe how the public IP behaves, which you can see in the console. You did not need to log in to the server, and a key pair is only for SSH authentication.

Things to know:

- Without a key pair you **cannot SSH with your own key**. The key pair is chosen only at launch and cannot be added later to a running instance through the console.
- You can still get a shell without a key pair using **EC2 Instance Connect** from the browser (Amazon Linux supports it when port 22 is allowed), or **Session Manager** (needs an IAM role with SSM permissions).
- Rule of thumb: if you plan to log in, create or select a key pair at launch. If you are only observing the console, skipping it is fine.

## 4. Part B: Elastic IP experiment

### Step 1: Allocate an Elastic IP

1. In the EC2 left menu, go to **Network & Security → Elastic IPs**.
2. Click **Allocate Elastic IP address**.
3. Keep the default (Amazon's pool of IPv4 addresses) and click **Allocate**.

### Step 2: Associate it with the instance

1. Select the new Elastic IP and click **Actions → Associate Elastic IP address**.
2. Resource type: **Instance**. Choose `ec2-eip`.
3. Click **Associate**.
4. Open the instance details and confirm the **Public IPv4 address** now equals the Elastic IP.

### Step 3: Test persistence (with Elastic IP attached)

1. Select the instance → **Instance state → Stop instance**. Wait until it is *Stopped*.
2. **Start instance** again.
3. Result: the public IP is **the same** as the Elastic IP. ✅

### Step 4: Test without the Elastic IP

1. Go to **Elastic IPs**, select the address, **Actions → Disassociate Elastic IP address**.
2. Stop the instance, then start it again.
3. Result: the public IP is **different** from before. This repeats every stop/start cycle. ❌

| State | Public IP after stop and start |
| --- | --- |
| Elastic IP associated | Same every time |
| Elastic IP disassociated (default auto-assigned IP) | Changes every time |


## 5. Why do we need Elastic IPs?

### The problem

By default an EC2 instance gets a **dynamic public IPv4 address** from AWS's pool. That address is borrowed only while the instance is running. When you **stop** the instance, AWS takes it back, and when you start it again you receive a different one. (A simple **reboot** keeps the same IP; **terminating** loses it forever.)

The **private IP** stays the same in the VPC. Only the public IP changes.

### The solution

An **Elastic IP (EIP)** is a static public IPv4 address that belongs to **your account** until you release it. You attach it to an instance, and it stays the same through stop/start cycles. You can also detach it and attach it to a different instance in seconds.

### Real-world uses

- **DNS records:** a domain such as `app.example.com` points to one IP. If the IP keeps changing, the website breaks every restart.
- **Whitelisting:** partners, banks or databases often allow traffic only from specific IPs. A stable IP means the whitelist never needs updating.
- **Quick failover:** if a server fails, move the Elastic IP to a standby instance and traffic goes to the new server without changing DNS.
- **Stable SSH or RDP access:** you always connect to the same address.
- **Email servers and APIs** where the sending IP's reputation matters.

### Things to remember

| Point | Detail |
| --- | --- |
| Scope | Regional. It can be moved between instances in the same region only |
| Cost | AWS charges for **all public IPv4 addresses**, including Elastic IPs, per hour. Check the current EC2 pricing page. The free tier gives limited free hours for the first 12 months |
| Idle EIP | An Elastic IP that is **not associated** with a running resource is still billed. Always release it when finished |
| Quota | Default limit of 5 Elastic IPs per region (can be raised) |
| Better alternatives | For web traffic, a Load Balancer with a DNS name is often preferred over a single EIP |

## 6. EC2 Instance Types

An instance type is a combination of **CPU, memory, storage and network capacity**. AWS offers many so that you can match the server to the workload and pay only for what you need.

### How to read the name

Example: **m5dn.2xlarge**

| Part | Meaning | In this example |
| --- | --- | --- |
| First letter(s) | **Family** (what it is optimised for) | m = general purpose |
| Number | **Generation** (higher is newer, usually better value) | 5th generation |
| Extra letters | **Attributes / capabilities** | d = local NVMe disk, n = enhanced network |
| After the dot | **Size** | 2xlarge |

Common attribute letters:

| Letter | Meaning |
| --- | --- |
| a | AMD processor |
| g | AWS Graviton (ARM) processor |
| i | Intel processor |
| d | Local NVMe instance storage |
| n | Enhanced networking |
| e | Extra memory or storage |
| b | Higher EBS bandwidth |

**Our instance, t3.micro:** t = burstable general purpose family, 3 = third generation, micro = size.

### Sizes (small to large)

`nano → micro → small → medium → large → xlarge → 2xlarge → 4xlarge → … → metal`

Each step up in a family roughly doubles the vCPU and memory (and price). `metal` gives direct access to the physical server.

### Instance families and what they are for

| Category | Families | Best for | Typical use |
| --- | --- | --- | --- |
| General purpose | T (burstable), M | Balanced CPU, memory and network | Web servers, small databases, dev/test, microservices, code repos |
| Compute optimised | C | High-performance CPUs | Batch processing, gaming servers, video encoding, scientific modelling, high-traffic web |
| Memory optimised | R, X, z | Large amounts of RAM | In-memory databases and caches (Redis), real-time analytics, big SAP/HANA workloads |
| Storage optimised | I, D, H | Very fast or very large local storage, high IOPS | NoSQL databases, data warehouses, log processing |
| Accelerated computing | P, G, Inf, Trn, F | GPUs and custom chips | Machine learning training and inference, graphics, video rendering |
| HPC optimised | Hpc | Tightly coupled, compute-heavy jobs | Weather simulation, fluid dynamics |

### About T-series (our t3.micro): burstable performance

- T instances run at a low **baseline** CPU level and can **burst** above it using **CPU credits** that build up while the instance is idle.
- Ideal for workloads that are quiet most of the time with occasional spikes (dev servers, small sites, learning labs).
- Not suitable for sustained high CPU, where an M or C family is a better fit.
- t3.micro has 2 vCPUs and 1 GiB of memory.

### Example sizes in the T3 family

| Type | vCPU | Memory (GiB) |
| --- | --- | --- |
| t3.nano | 2 | 0.5 |
| t3.micro | 2 | 1 |
| t3.small | 2 | 2 |
| t3.medium | 2 | 4 |
| t3.large | 2 | 8 |

### How to choose

1. Identify the bottleneck: CPU, memory, disk or network.
2. Pick the family that matches (C for CPU, R for RAM, I for disk, G/P for GPU, T/M if balanced or unsure).
3. Start small, monitor with CloudWatch, then resize. You can change the instance type of a **stopped** instance.
4. Prefer newer generations and consider Graviton (g) for better price and performance if your software supports ARM.

## 7. Cleanup (in the order performed)

Order matters, because some resources cannot be deleted while others still use them.

1. **Release the Elastic IP:** Elastic IPs → select the address → **Actions → Release Elastic IP addresses**. (If still associated, disassociate first.) This stops the charges for it.
2. **Stop and terminate the instance:** Instances → select `ec2-eip` → **Instance state → Stop**, then **Terminate**. Terminated instances disappear from the list after a short time.
3. **Delete the security group:** Network & Security → Security Groups → select the `launch-wizard-x` group created at launch → **Actions → Delete security groups**. It can only be deleted after the instance is terminated.

> Note: you can terminate directly without stopping first. Stopping first is optional.

**Final check:** Instances shows *Terminated*, Elastic IPs is empty, and no leftover launch-wizard security group remains.

## 8. Troubleshooting

| Problem | Cause | Fix |
| --- | --- | --- |
| Cannot delete the security group | Still attached to a running or recently terminated instance | Wait for the instance to show *Terminated*, then retry |
| Public IP changed after restart | No Elastic IP attached | Associate an Elastic IP |
| Unexpected charges | Idle Elastic IP or a stopped instance with attached storage | Release the EIP, terminate the instance |
| Cannot SSH into the instance | No key pair or port 22 blocked | Launch with a key pair, or use EC2 Instance Connect |

## 9. Key takeaways

- The default public IP is temporary. It changes on stop/start (not on reboot).
- An Elastic IP is a static, account-owned public IPv4 address that you can remap between instances.
- Instance names follow the pattern **family + generation + attributes . size**.
- Choose the family by workload: T/M general, C compute, R/X memory, I/D storage, P/G accelerated.
- Always release unused Elastic IPs, terminate instances and delete leftover security groups.

