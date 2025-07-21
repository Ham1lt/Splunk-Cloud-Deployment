# Terraform AWS Infrastructure for Splunk Deployment  

This Terraform project automates the creation of AWS infrastructure to deploy a Splunk instance in a VPC with a public subnet and necessary security groups. It uses a custom network setup and fetches the latest Splunk AMI for the EC2 instance.

---

## **Features**

1. **VPC Creation**  
   - Creates a Virtual Private Cloud (VPC) with a custom CIDR block.  

2. **Public Subnet**  
   - A subnet with auto-assigned public IPs in the first availability zone.  

3. **Internet Gateway**  
   - Enables public internet access for resources.  

4. **Route Table**  
   - Routes all outbound traffic (`0.0.0.0/0`) to the Internet Gateway.  

5. **Security Group**  
   - Restricts API (8089) and UI (8000) access to the user's public IP.  

6. **EC2 Instance for Splunk**  
   - Fetches the latest Splunk AMI and launches an EC2 instance in the public subnet.  
   - Assigns a static private IP and tags the instance for identification.  

---

## **Project Structure**

```plaintext
.
├── main.tf           # Combined Terraform configuration
├── variables.tf      # Input variables for the project
├── outputs.tf        # Outputs for Splunk instance IP and credentials
├── terraform.tfvars  # User-defined variable values
├── provider.tf       # AWS provider configuration
├── data.tf           # Data sources (e.g., AMI, Public IP)
├── .gitignore        # Ignore Terraform state and sensitive files
└── README.md         # Project documentation
```

---

## **Requirements**

- **Terraform**: >= 1.0.0  
- **AWS CLI**: Installed and configured with the correct credentials  
- **AWS IAM Permissions**:  
  - Ability to create VPC, subnets, route tables, Internet Gateway, Security Groups, and EC2 instances  

---

## **Usage**

### 1. Clone the Repository  
```bash
git clone <repository_url>
cd <repository_directory>
```

### 2. Initialize Terraform  
```bash
terraform init
```

### 3. Review the Plan  
```bash
terraform plan
```

### 4. Apply the Configuration  
```bash
terraform apply
```

### 5. Outputs  
After the deployment, Terraform will output:  
- **Splunk Public IP**: The public IP address of the EC2 instance  
- **Splunk Default Username**: `admin`  
- **Splunk Default Password**: Auto-generated based on the instance ID  

---

## **Example `terraform.tfvars`**

Create a `terraform.tfvars` file in the root directory with the following content:

```hcl
region        = "us-east-1"
vpc_cidr      = "10.0.0.0/16"
subnet        = "10.0.1.0/24"
private_ip    = "10.0.1.10"
instance_type = "t2.micro"
```

---

## **Outputs**

- **`splunk_public_ip`**: Public IP of the Splunk EC2 instance.  
- **`splunk_default_username`**: Default Splunk username (`admin`).  
- **`splunk_default_password`**: Default password based on the EC2 instance ID.

---

## **Cleanup**

To destroy the infrastructure and release all resources:

```bash
terraform destroy
```

---

## **Notes**

- This code is configured to use your public IP for secure API and UI access to Splunk.  
- Update `terraform.tfvars` to change network configurations.  
- Use a key-pair and IAM role with proper permissions for secure EC2 access.

This lab includes an optional training layer that uses the Ultimate Format to help analysts interpret logs faster and more accurately. Here’s a sample:<details>
<summary><strong>🧠 Ultimate Format – Log Analysis Training (3 Examples)</strong></summary>

---

## ✅ Example 1 – Brute-Force SSH Attack

### ✅ Sample Log Entry
Jul 21 14:01:17 ip-10-0-1-10 sshd[5872]: Failed password for invalid user admin from 203.0.113.5 port 45122 ssh2

### ✅ Correct Response:
**Investigate possible brute-force attack from IP 203.0.113.5 targeting nonexistent accounts.**

### ✅ Keyword Trigger Mapping

| Log Phrase              | Interpreted Meaning              | Mapped Response           |
|------------------------|----------------------------------|---------------------------|
| Failed password        | Unsuccessful login attempt       | Login Failure             |
| invalid user admin     | Targeting non-existent account   | Recon Attempt             |
| from 203.0.113.5       | External IP address              | Suspicious Source         |
| port 45122 ssh2        | SSH protocol activity            | SSH Vector                |

### 🔍 Why This Is Correct:
- `Failed password` = system rejected login attempt.
- `invalid user` = attacker guessing usernames.
- `203.0.113.5` = not internal, likely attacker.
- Matches common brute-force or scanner behavior.

### ❌ Why Other Interpretations Are Wrong:
- Not just a typo – it’s an invalid user.
- Not internal admin – IP is external.
- Not just a failed password – it’s targeting nonexistent accounts.

### 🧠 Mental Shortcut:
> “Invalid user + failed password + public IP = Brute force probe.”

### 🔑 Trigger Shortcuts (Always Maps To):
- `invalid user` → B. Reconnaissance attempt  
- `from <public IP>` → D. External threat vector  
- `sshd + Failed password` → A. Brute-force SSH attack

---

## ✅ Example 2 – Privilege Escalation Attempt

### ✅ Sample Log Entry

Jul 21 14:22:43 ip-10-0-1-10 sudo: tahj : user NOT in sudoers ; TTY=pts/1 ; PWD=/home/tahj ; USER=root ; COMMAND=/bin/bash

### ✅ Correct Response:
**Unauthorized privilege escalation attempt by user `tahj`. Review access controls and audit intent.**

### ✅ Keyword Trigger Mapping

| Log Phrase                  | Interpreted Meaning               | Mapped Response                |
|----------------------------|-----------------------------------|-------------------------------|
| sudo:                      | Privilege command executed        | Privilege Escalation Attempt  |
| user NOT in sudoers        | User lacks permission             | Unauthorized Action           |
| COMMAND=/bin/bash          | Attempt to access privileged shell| Root Shell Attempt            |
| USER=root                  | Target user is root               | High-Risk Escalation Target   |

### 🔍 Why This Is Correct:
- `sudo:` = attempt to gain elevated privileges.
- `NOT in sudoers` = user isn’t authorized.
- `/bin/bash` = intent to launch a root shell.
- Target user is `root` = high sensitivity.

### ❌ Why Other Interpretations Are Wrong:
- Not a normal sudo usage – it’s unauthorized.
- Not a system error – log shows intent.
- Not low risk – targeting root directly.

### 🧠 Mental Shortcut:
> “Not in sudoers + command to root shell = Unauthorized root escalation.”

### 🔑 Trigger Shortcuts (Always Maps To):
- `user NOT in sudoers` → B. Unauthorized privilege escalation  
- `COMMAND=/bin/bash` → D. Attempted root shell access  
- `USER=root` → A. High-privilege target  
- `sudo:` + denial → E. Security policy violation

---

## ✅ Example 3 – Web Recon / Directory Scan

### ✅ Sample Log Entry
Jul 21 15:14:08 ip-10-0-1-10 nginx[1732]: 203.0.113.5 - - [21/Jul/2025:15:14:08 +0000] “GET /admin HTTP/1.1” 404 162 “-” “Mozilla/5.0”

### ✅ Correct Response:
**External IP probing for sensitive endpoint `/admin`. Monitor for follow-up scans or injection attempts.**

### ✅ Keyword Trigger Mapping

| Log Phrase              | Interpreted Meaning                | Mapped Response                     |
|------------------------|------------------------------------|------------------------------------|
| GET /admin             | Sensitive endpoint scan            | Directory Scanning (Recon)         |
| 203.0.113.5            | Public IP                          | External Threat Actor              |
| 404                    | Page not found                     | Directory Discovery Attempt        |
| nginx                  | Web server logs                    | Application Layer Activity         |

### 🔍 Why This Is Correct:
- `/admin` is a known target path during recon.
- `203.0.113.5` is external → not legit internal admin.
- `404` = scan was **probing** for existence.
- Common before credential stuffing or injection attempts.

### ❌ Why Other Interpretations Are Wrong:
- Not a user mistake – it’s a crafted probe.
- Not internal – IP is public.
- Not harmless – checking `/admin` is known recon behavior.

### 🧠 Mental Shortcut:
> “GET /admin + 404 + external IP = Web path reconnaissance.”

### 🔑 Trigger Shortcuts (Always Maps To):
- `GET /admin` → C. Directory scan / reconnaissance  
- `404` with sensitive path → D. Unsuccessful but deliberate scan  
- `nginx` + public IP → B. Application-layer probing  
- `203.0.113.5` → A. Suspicious external source  

---

</details>
 



## **Author**

Tahj Hamilton  


