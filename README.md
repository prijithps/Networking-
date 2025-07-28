# Networking-
 "Route tables of a server" refer to the data structures maintained by the server's operating system (typically in the kernel) that determine how network traffic is forwarded or routed from the server to other devices or networks.

---

### 🧭 What is a Route Table?

A **route table** is like a roadmap for your server. It tells the server **where to send network packets** based on their destination IP addresses.

---

### 📋 Contents of a Route Table

Each entry in the route table usually includes:

| Field                      | Description                                |
| -------------------------- | ------------------------------------------ |
| **Destination**            | Target network or IP address               |
| **Gateway**                | The next-hop IP address or router          |
| **Genmask** or **Netmask** | Defines the subnet                         |
| **Interface**              | Network interface used (e.g., eth0, wlan0) |
| **Flags**                  | Status flags (e.g., U = up, G = gateway)   |
| **Metric**                 | Cost of the route (lower is preferred)     |

---

### 🖥️ Example Route Table (Linux: `route -n` or `ip route show`)

```bash
$ ip route show
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10
```

#### Explanation:

* **default via 192.168.1.1 dev eth0**: All traffic not matching other rules goes through `192.168.1.1` using interface `eth0`.
* **192.168.1.0/24 dev eth0**: Local network traffic is directly reachable on `eth0`.

---

### 🛠️ How to View Route Tables

#### Linux:

* `ip route show`
* `netstat -rn`
* `route -n`

#### Windows:

* `route print`

#### macOS:

* `netstat -rn`
* `route -n get default`

---

### 🚦Types of Routes

1. **Directly Connected Routes** – For networks the server is part of.
2. **Static Routes** – Manually added by administrators.
3. **Dynamic Routes** – Added by routing protocols like BGP, OSPF, etc.

---

### ✅ Use Cases

* Ensuring internal traffic stays within the LAN.
* Sending specific traffic through a VPN.
* Connecting to different subnets in a cloud environment (AWS/GCP/Azure).
* Isolating networks for security.

---

Would you like an example with `ip route add` or how to modify route tables in cloud platforms like AWS or Azure?


### 🌐 What is a NAT Gateway?

A **NAT Gateway (Network Address Translation Gateway)** is a **network device or service** that allows **instances (like virtual machines)** in a **private subnet** to connect to the internet **without exposing their private IP addresses**.

It acts as an **intermediary**, translating **private IPs → public IPs** when sending traffic out, and then **public IPs → private IPs** when returning traffic comes in.

---

### 📦 Where is NAT Gateway Used?

Mainly used in **cloud environments** (like AWS, Azure, GCP) to:

* Allow **private subnet** instances to access the internet (e.g., to download updates, access APIs).
* **Prevent** the internet from **initiating connections** to private resources.

---

### 🔁 How It Works (Simplified)

| Private Instance | NAT Gateway | Internet                     |
| ---------------- | ----------- | ---------------------------- |
| 10.0.1.10        | ↔           | Public IP (e.g., 54.23.11.5) |

* Outgoing request:
  `10.0.1.10 → www.google.com`
  NAT translates it to: `54.23.11.5 → www.google.com`

* Incoming response:
  `www.google.com → 54.23.11.5`
  NAT translates back to: `→ 10.0.1.10`

---

### 🛡️ Key Features

| Feature               | NAT Gateway                                             |
| --------------------- | ------------------------------------------------------- |
| **Outbound Only**     | No unsolicited inbound traffic allowed                  |
| **Scalable**          | Managed by cloud providers                              |
| **Secure**            | Keeps private instances hidden from the public internet |
| **Automatic Mapping** | Handles port/address translation automatically          |

---

### ☁️ Example (AWS)

* Private EC2 instance in subnet `10.0.1.0/24`
* NAT Gateway created in **public subnet**
* Route table for private subnet:

  ```
  Destination     Target
  0.0.0.0/0       NAT Gateway
  ```

---

### 🆚 NAT Gateway vs Internet Gateway

| Feature   | NAT Gateway            | Internet Gateway            |
| --------- | ---------------------- | --------------------------- |
| Direction | Outbound only          | Both Inbound and Outbound   |
| Use Case  | Private subnets        | Public subnets              |
| IP Type   | Private IP → Public IP | Public IP directly assigned |
| Security  | Hides internal IPs     | Exposes public IPs          |

---

### ✅ When to Use a NAT Gateway

* You have **private EC2s or VMs** that need internet access (e.g., to fetch updates or connect to external APIs).
* You **don’t want those instances to be reachable from the internet**.

---

Let me know if you want a **diagram**, **AWS example**, or **Terraform snippet** for setting one up!



### 🌐 What is an Internet Gateway?

An **Internet Gateway (IGW)** is a **component in a Virtual Private Cloud (VPC)** (like in AWS) that enables communication **between instances in your VPC and the public internet**.

It allows resources (like EC2 instances) that have **public IP addresses** to:

* **Send traffic to the internet**
* **Receive traffic from the internet**

---

### 🔁 How It Works

| Instance Type        | IP Type   | Route             | Internet Gateway Role                       |
| -------------------- | --------- | ----------------- | ------------------------------------------- |
| EC2 in public subnet | Public IP | `0.0.0.0/0 → IGW` | Enables full two-way internet communication |

Example:

* Your EC2 instance has public IP `54.23.12.10`
* You access it via SSH: `ssh ec2-user@54.23.12.10`
* The IGW handles both **inbound and outbound** traffic

---

### 📦 Key Functions of an IGW

| Function                       | Description                                                        |
| ------------------------------ | ------------------------------------------------------------------ |
| **Route Traffic**              | Acts as a target for route table entries (e.g., `0.0.0.0/0`)       |
| **Perform NAT for public IPs** | Maps internal private IPs to external public IPs (1:1)             |
| **Enable internet access**     | Without IGW, no public internet communication is possible in a VPC |

---

### ☁️ AWS Example

#### 1. Public Subnet Setup:

* EC2 instance in subnet `10.0.0.0/24`
* Has a **public IP**
* Subnet is **associated with a route table** that has:

```
Destination     Target
0.0.0.0/0       Internet Gateway (igw-123456)
```

#### 2. Internet Gateway Attached:

* IGW is created and **attached to the VPC**
* The subnet’s route table uses the IGW as the route to the internet

---

### 🆚 Internet Gateway vs NAT Gateway

| Feature               | Internet Gateway              | NAT Gateway           |
| --------------------- | ----------------------------- | --------------------- |
| **Traffic Direction** | Inbound & Outbound            | Outbound only         |
| **IP Type**           | Public IP required            | Private IP            |
| **Used In**           | Public Subnets                | Private Subnets       |
| **Security**          | Can expose instance to public | Keeps instance hidden |

---

### ✅ When to Use an Internet Gateway

* You want your EC2 instance to be **reachable from the internet** (e.g., for hosting a website).
* You assign a **public IPv4 address or Elastic IP** to your instance.
* You **configure the route table** to send `0.0.0.0/0` traffic through the IGW.

---

Would you like a **diagram** or a **Terraform example** showing how to attach and route through an IGW in AWS?



### 📌 Why is Elastic IP Used?

An **Elastic IP (EIP)** is a **static, public IPv4 address** provided by AWS that you can **allocate to your account** and **associate with your EC2 instances or other resources**.

---

### ✅ Main Reasons to Use Elastic IP

| Reason                   | Explanation                                                                                         |
| ------------------------ | --------------------------------------------------------------------------------------------------- |
| **Static Public IP**     | Unlike regular public IPs (which can change on reboot), an EIP stays the same until you release it. |
| **Reliable DNS Mapping** | You can map a domain (like `myapp.com`) to an EIP and know it won't change.                         |
| **Disaster Recovery**    | If an instance fails, you can quickly remap the EIP to another instance, minimizing downtime.       |
| **IP Reusability**       | You can reassign the EIP to another EC2 instance or NAT Gateway without changing IPs.               |

---

### 🛠️ Without vs With Elastic IP

#### 🔄 **Without Elastic IP:**

* AWS assigns a **dynamic public IP** to your instance.
* That IP **can change** if:

  * You stop/start the instance.
  * You terminate and relaunch it.

#### ✅ **With Elastic IP:**

* You get a **static public IP**.
* You can associate and reassociate it to any instance in your VPC.

---

### ☁️ Example Use Case in AWS

1. You have a web server EC2 in a public subnet.
2. You assign it an **Elastic IP** (e.g., `54.22.11.10`).
3. You configure your domain DNS (e.g., `myblog.com`) to point to that IP.
4. Later, if you launch a new instance, you can reassign the same EIP to it — your users see no downtime or IP change.

---

### 💸 Cost Note

* **EIP is free** **only while it's attached to a running EC2 instance**.
* **Charges apply** if:

  * It’s allocated but **not associated** with a resource.
  * You have **multiple** EIPs.

---

### 🔄 Elastic IP vs Public IP

| Feature          | Elastic IP                 | Public IP (default)          |
| ---------------- | -------------------------- | ---------------------------- |
| **Static**       | ✅ Yes                      | ❌ No (changes on stop/start) |
| **Reassignable** | ✅ Yes                      | ❌ No                         |
| **Cost**         | Free if used               | Free                         |
| **Best For**     | Long-running services, DNS | Temporary, testing           |

---

Let me know if you want to try setting one up with AWS CLI, Terraform, or a visual diagram.
