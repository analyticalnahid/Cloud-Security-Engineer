# 🔐 Cloud Security Engineer – Complete Roadmap
### Built for: インフラエンジニア (1+ Year Japan Experience) → Cloud Security Engineer

---

## 🔰 What is a Cloud Security Engineer?

In Japan, this role appears as:

- クラウドセキュリティエンジニア
- セキュリティエンジニア（クラウド）
- インフラセキュリティエンジニア
- DevSecOpsエンジニア

**Your advantage:** You already understand how infrastructure works at a physical and operational level. Most security engineers never had this — it makes you significantly harder to replace.

---

## 🗺️ Your Starting Point

You already have:

| Skill | Level |
|---|---|
| Linux server management | ✅ Solid |
| Windows Server / Active Directory | ✅ Solid |
| Networking (CCNA level) | ✅ Solid |
| Virtualization / Docker / K8s basics | ✅ Solid |
| AWS fundamentals | ✅ Solid |
| Monitoring & automation | ✅ Solid |
| Japan IT workplace experience | ✅ 1+ year |

Now you add the **security layer on top of everything you already know.**

---

# 📘 PHASE 1 – Security Mindset & Fundamentals

## 🔹 Attacker vs Defender Thinking

The most important shift from インフラエンジニア to Security Engineer:

```
インフラ mindset:  "How do I build this to work?"
Security mindset: "How would I break this if I were the attacker?"
```

You need both. Infrastructure knowledge tells you how systems work.
Security thinking tells you how they fail.

---

## 🔹 The CIA Triad (Foundation of Everything)

| Principle | Meaning | Example |
|---|---|---|
| Confidentiality | Only authorized access | Encryption, IAM |
| Integrity | Data not tampered with | Hashing, signing |
| Availability | System stays online | DDoS protection, backups |

📌 Every security decision maps back to one of these three.

---

## 🔹 Common Attack Types (Must Know All)

| Attack | How It Works | Defense |
|---|---|---|
| Phishing | Trick user into credentials | MFA, user training |
| Man-in-the-Middle | Intercept traffic | TLS, certificate pinning |
| SQL Injection | Malicious DB queries | Input validation, WAF |
| XSS | Inject malicious scripts | CSP, output encoding |
| Privilege Escalation | Gain higher access | Least privilege, RBAC |
| Lateral Movement | Spread inside network | Network segmentation, Zero Trust |
| Ransomware | Encrypt & ransom data | Backups, EDR, network isolation |
| Supply Chain Attack | Compromise software dependencies | SCA scanning, signed packages |

---

## 🔹 Security Frameworks (Reference Models)

| Framework | Purpose | Where Used |
|---|---|---|
| NIST CSF | Cybersecurity framework | Global |
| ISO 27001 | Security management standard | Global / Europe |
| MITRE ATT&CK | Attacker tactics & techniques | SOC / Red team |
| CIS Controls | Prioritized security actions | Enterprise |
| OWASP Top 10 | Web application risks | AppSec |
| METI Cybersecurity Framework | Japan-specific guidance | Japan enterprises |

📌 Bookmark: https://attack.mitre.org — learn to read ATT&CK matrices.

---

## 🔹 Certification Target This Phase

🎯 **CompTIA Security+**
- Globally recognized entry security cert
- Covers all fundamentals above
- Required by many Japan enterprise security job postings
- Study time: 6–8 weeks

---

# 🐧 PHASE 2 – Linux Security Hardening

## You already know Linux. Now you lock it down.

---

## 🔹 SSH Hardening (First thing you do on any server)

```bash
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
Port 2222               # change default port
MaxAuthTries 3
AllowUsers youruser
```

📌 Always disable root login. Always use key-based auth.

---

## 🔹 Firewall (iptables / ufw / firewalld)

```bash
# UFW example
ufw default deny incoming
ufw default allow outgoing
ufw allow 2222/tcp       # SSH (custom port)
ufw allow 443/tcp        # HTTPS
ufw enable
ufw status verbose
```

```bash
# iptables - drop all INPUT by default
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

---

## 🔹 User & Privilege Management

```bash
# Principle of least privilege
useradd -m -s /bin/bash appuser
passwd appuser

# Never run services as root
# Check who has sudo
cat /etc/sudoers
getent group sudo

# Audit login history
last
lastb              # failed logins
who
w
```

---

## 🔹 File Integrity Monitoring

```bash
# AIDE - detect unauthorized file changes
apt install aide
aide --init
aide --check

# Check for SUID files (privilege escalation risk)
find / -perm -4000 -type f 2>/dev/null

# Check for world-writable files
find / -perm -0002 -type f 2>/dev/null
```

---

## 🔹 Log Analysis (Security-Focused)

```bash
# Auth logs - detect brute force
/var/log/auth.log
/var/log/secure          # RHEL/CentOS

# Find failed SSH attempts
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Detect successful logins from new IPs
grep "Accepted" /var/log/auth.log

# Check cron for persistence
cat /etc/crontab
ls /etc/cron.*
crontab -l
```

---

## 🔹 Fail2ban (Automated Brute Force Protection)

```bash
apt install fail2ban

# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = 2222
maxretry = 3
bantime = 3600
findtime = 600
```

---

## 🔹 OS Hardening Checklist

```
✅ Disable root SSH login
✅ Key-based auth only
✅ Firewall default deny
✅ Remove unused services
✅ Apply all security patches
✅ Enable auditd
✅ Configure fail2ban
✅ File integrity monitoring (AIDE)
✅ Disable IPv6 if not needed
✅ Set kernel hardening params (sysctl)
```

📌 sysctl hardening example:

```bash
# /etc/sysctl.conf
net.ipv4.ip_forward = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.tcp_syncookies = 1
kernel.randomize_va_space = 2
```

---

# 🌐 PHASE 3 – Network Security Deep Dive

## You already know networking. Now you think like a defender — and attacker.

---

## 🔹 Network Security Architecture

```
Internet
    │
  [WAF]              ← Web Application Firewall
    │
  [Firewall]         ← Perimeter defense
    │
  [DMZ]              ← Public-facing servers only
    │
  [IDS/IPS]          ← Detect/block intrusions
    │
  [Internal Network] ← Segmented by VLAN
    │
  [Core Systems]     ← Highest protection
```

---

## 🔹 Network Segmentation

| Zone | What Lives Here | Trust Level |
|---|---|---|
| Internet | Everything public | Zero |
| DMZ | Web servers, APIs | Low |
| Internal | Employee systems | Medium |
| Restricted | Databases, secrets | High |
| Management | Admin access only | Highest |

📌 Rule: Systems should only talk to what they absolutely need to.

---

## 🔹 IDS vs IPS

| Tool | Action | Example |
|---|---|---|
| IDS (Intrusion Detection) | Alerts only | Snort (detect mode) |
| IPS (Intrusion Prevention) | Blocks automatically | Snort (inline mode), Suricata |

```bash
# Suricata - modern IDS/IPS
apt install suricata
suricata -c /etc/suricata/suricata.yaml -i eth0

# Check alerts
tail -f /var/log/suricata/fast.log
```

---

## 🔹 Traffic Analysis Tools

| Tool | Purpose |
|---|---|
| Wireshark | Packet capture & analysis |
| tcpdump | CLI packet capture |
| Zeek | Network security monitoring |
| Suricata | IDS/IPS + logging |
| nmap | Network scanning |

```bash
# Capture traffic on interface
tcpdump -i eth0 -w capture.pcap

# Scan network for open ports
nmap -sV -sC -p- 192.168.1.0/24

# Check open ports locally
ss -tlnp
netstat -tlnp
```

---

## 🔹 VPN Security

| Type | Use Case | Security Notes |
|---|---|---|
| Site-to-Site VPN | Office ↔ Cloud | IPSec, strong encryption |
| Remote Access VPN | User ↔ Corporate | MFA required |
| Zero Trust (replacing VPN) | Per-app access | No network-level trust |

---

## 🔹 DNS Security

```bash
# DNS poisoning protection
# Use DNSSEC
# Check DNS queries for malicious domains

# DNS over HTTPS (DoH)
# DNS over TLS (DoT)

# Block malicious domains via DNS sinkhole
# Tools: Pi-hole, Bind RPZ
```

---

# ☁️ PHASE 4 – AWS Cloud Security (Core Phase)

## This is your biggest differentiator. Deep cloud security knowledge is the #1 shortage in Japan.

---

## 🔹 AWS Shared Responsibility Model

```
AWS Responsible For:          You Responsible For:
─────────────────────         ─────────────────────
Physical datacenters          Your data
Network infrastructure        Your OS configuration
Hypervisor                    Your applications
Managed service security      Identity & access (IAM)
                              Encryption choices
                              Network config (VPC/SG)
```

📌 Misunderstanding this model = #1 cause of cloud breaches.

---

## 🔹 IAM Security (Most Critical AWS Topic)

```json
// Bad policy - never do this
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}

// Good policy - least privilege
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

**IAM Security Rules:**

```
✅ Never use root account for daily work
✅ Enable MFA on all accounts (especially root)
✅ Use IAM roles for EC2, Lambda (not access keys)
✅ Rotate access keys regularly
✅ Least privilege on all policies
✅ Use permission boundaries
✅ Audit with IAM Access Analyzer
```

---

## 🔹 VPC Security Architecture

```
VPC (10.0.0.0/16)
  │
  ├── Public Subnet (10.0.1.0/24)
  │     └── NAT Gateway, ALB, Bastion
  │
  ├── Private Subnet (10.0.2.0/24)
  │     └── EC2, ECS, Lambda
  │
  └── Isolated Subnet (10.0.3.0/24)
        └── RDS, ElastiCache (no internet)
```

**Security Group Rules (Defense in depth):**

```
ALB Security Group:
  Inbound:  443 from 0.0.0.0/0
  Outbound: 8080 to App-SG only

App Security Group:
  Inbound:  8080 from ALB-SG only
  Outbound: 5432 to DB-SG only

DB Security Group:
  Inbound:  5432 from App-SG only
  Outbound: none
```

---

## 🔹 AWS Security Services (Must Know All)

| Service | Purpose |
|---|---|
| AWS IAM | Identity & access management |
| AWS GuardDuty | Threat detection (AI-powered) |
| AWS Security Hub | Centralized security findings |
| AWS Config | Configuration compliance |
| AWS CloudTrail | API audit logging |
| AWS WAF | Web application firewall |
| AWS Shield | DDoS protection |
| AWS Macie | Sensitive data discovery (S3) |
| AWS Inspector | Vulnerability scanning (EC2/ECR) |
| AWS Secrets Manager | Secrets & credential management |
| AWS KMS | Encryption key management |
| AWS Certificate Manager | TLS certificates |

📌 Example: Enable GuardDuty in all regions:

```bash
aws guardduty create-detector --enable --region ap-northeast-1
aws guardduty create-detector --enable --region eu-west-1
```

---

## 🔹 CloudTrail — Audit Everything

```bash
# CloudTrail logs every API call
# Who did what, when, from where

# Query with Athena
SELECT eventTime, userIdentity.arn, eventName, sourceIPAddress
FROM cloudtrail_logs
WHERE eventName = 'DeleteBucket'
ORDER BY eventTime DESC;

# Alert on root account usage
# Alert on IAM changes
# Alert on security group changes
```

---

## 🔹 Encryption on AWS

| Data State | Service | Example |
|---|---|---|
| At rest | KMS + S3 SSE | S3 bucket encryption |
| In transit | ACM + TLS | HTTPS everywhere |
| Secrets | Secrets Manager | DB passwords, API keys |
| Keys | KMS / CloudHSM | Encryption key management |

```bash
# Never hardcode credentials
# Bad:
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"

# Good: Use IAM role attached to EC2/Lambda
aws sts get-caller-identity   # verify role
```

---

## 🔹 S3 Security (Most Breached AWS Service)

```bash
# Check for public buckets
aws s3api get-bucket-acl --bucket my-bucket
aws s3api get-bucket-policy --bucket my-bucket

# Block all public access (do this for ALL buckets)
aws s3api put-public-access-block \
  --bucket my-bucket \
  --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,\
   BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Enable versioning + MFA delete
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration MFADelete=Enabled,Status=Enabled
```

---

## 🔹 Certification Target This Phase

🎯 **AWS Certified Security – Specialty**
- Most valued cloud security cert in Japan
- Covers: IAM, encryption, logging, threat detection, incident response
- Study time: 8–12 weeks after SAA
- Salary jump in Japan: typically ¥1M–¥2M increase

---

# 🔑 PHASE 5 – Identity & Access Management Deep Dive

## IAM is the #1 attack vector in cloud environments. Master it completely.

---

## 🔹 Zero Trust Architecture

```
Old Model (Perimeter Security):
  Inside network = trusted
  Outside network = untrusted

Zero Trust Model:
  Nothing is trusted by default
  Verify every user, device, request
  Assume breach at all times
```

**Zero Trust Pillars:**

| Pillar | What It Means |
|---|---|
| Verify explicitly | Always authenticate & authorize |
| Least privilege | Minimum access needed |
| Assume breach | Limit blast radius, encrypt, monitor |

---

## 🔹 Authentication vs Authorization

```
Authentication (AuthN): Who are you?
  → Username/password, MFA, certificates

Authorization (AuthZ): What can you do?
  → IAM policies, RBAC, ABAC
```

---

## 🔹 MFA Types (Strongest to Weakest)

| Type | Example | Strength |
|---|---|---|
| Hardware key | YubiKey, FIDO2 | ⭐⭐⭐⭐⭐ |
| Authenticator app | Google Auth, Authy | ⭐⭐⭐⭐ |
| Push notification | Duo, Okta Verify | ⭐⭐⭐ |
| SMS OTP | Text message code | ⭐⭐ (phishable) |

📌 SMS MFA is better than nothing but can be SIM-swapped. Always push for hardware keys for privileged accounts.

---

## 🔹 Federation & SSO

```
SAML 2.0:
  User → Identity Provider (IdP) → Service Provider
  Example: Corporate AD → AWS Console

OAuth 2.0:
  Authorization framework
  "Login with Google" uses OAuth

OIDC (OpenID Connect):
  Identity layer on top of OAuth 2.0
  Used by AWS Cognito, Google, Microsoft

JWT (JSON Web Token):
  Signed token for stateless auth
  Header.Payload.Signature
```

---

## 🔹 Privileged Access Management (PAM)

```
Regular User:
  └── Normal daily access only

Privileged User:
  └── Admin rights
  └── Requires PAM solution

PAM Controls:
  ✅ Just-in-time access (request → approve → expire)
  ✅ Session recording for all privileged sessions
  ✅ Password vault (no human knows the password)
  ✅ MFA for all privileged access
  ✅ Audit trail of every action
```

**Tools:**

| Tool | Type |
|---|---|
| CyberArk | Enterprise PAM (most common in Japan) |
| HashiCorp Vault | Secrets + dynamic credentials |
| AWS IAM Identity Center | AWS-native SSO |
| Okta | Identity platform |
| Azure AD / Entra ID | Microsoft identity |

---

## 🔹 RBAC vs ABAC

```
RBAC (Role-Based Access Control):
  User → Role → Permissions
  Example: Developer role → read S3, write to dev bucket

ABAC (Attribute-Based Access Control):
  User attributes + resource attributes + conditions
  Example: Allow IF department=finance AND data-classification=finance
```

---

# 🔍 PHASE 6 – Security Operations (SOC & SIEM)

---

## 🔹 Security Operations Flow

```
Collect logs & events
        ↓
Correlate & analyze (SIEM)
        ↓
Alert on anomalies
        ↓
Investigate (Tier 1 → 2 → 3)
        ↓
Respond & contain
        ↓
Eradicate & recover
        ↓
Lessons learned & improve
```

---

## 🔹 SIEM Tools

| Tool | Strength | Common In |
|---|---|---|
| Microsoft Sentinel | Cloud-native, AI-powered | Japan enterprises (Azure) |
| Splunk | Most powerful, expensive | Large enterprises |
| Elastic SIEM | Open source, flexible | Mid-size companies |
| AWS Security Hub | AWS-native | AWS environments |
| IBM QRadar | Enterprise | Financial sector |

---

## 🔹 Microsoft Sentinel (Hands-On)

```
# KQL (Kusto Query Language) - Sentinel's query language
# Find failed sign-ins
SigninLogs
| where ResultType != 0
| summarize count() by UserPrincipalName, IPAddress
| where count_ > 10
| order by count_ desc

# Detect impossible travel
SigninLogs
| where TimeGenerated > ago(1h)
| project UserPrincipalName, Location, TimeGenerated
| summarize locations = make_set(Location) by UserPrincipalName
| where array_length(locations) > 1

# New admin account creation
AuditLogs
| where OperationName == "Add member to role"
| where TargetResources contains "Global Administrator"
```

---

## 🔹 Log Sources to Collect

```
Infrastructure:
  ✅ Linux auth logs (/var/log/auth.log)
  ✅ Windows Event Logs (4624, 4625, 4720, 4732)
  ✅ Firewall logs
  ✅ DNS query logs
  ✅ DHCP logs

Cloud (AWS):
  ✅ CloudTrail (all API calls)
  ✅ VPC Flow Logs (network traffic)
  ✅ GuardDuty findings
  ✅ WAF logs
  ✅ ALB access logs

Applications:
  ✅ Web server logs (Nginx, Apache)
  ✅ Application error logs
  ✅ Database audit logs
```

---

## 🔹 Key Windows Event IDs (Interview Topic)

| Event ID | Meaning |
|---|---|
| 4624 | Successful login |
| 4625 | Failed login |
| 4720 | User account created |
| 4732 | User added to security group |
| 4776 | Credential validation |
| 7045 | New service installed |
| 4698 | Scheduled task created |

---

## 🔹 Incident Response (IR)

```
PHASE 1 - Preparation
  └── IR plan, playbooks, tooling ready

PHASE 2 - Detection & Analysis
  └── Alert fires → triage → confirm incident

PHASE 3 - Containment
  └── Isolate affected systems
  └── Preserve evidence (snapshots, logs)

PHASE 4 - Eradication
  └── Remove malware, close attack vector

PHASE 5 - Recovery
  └── Restore from clean backup
  └── Verify clean before reconnecting

PHASE 6 - Lessons Learned
  └── Root cause analysis
  └── Update playbooks & controls
```

📌 In Japan: Incident reports (インシデントレポート) are extremely important. Practice writing clear, structured post-mortems.

---

# ⚔️ PHASE 7 – Offensive Security Basics (Think Like an Attacker)

## You don't need to be a pentester. But you must understand what attackers do.

---

## 🔹 Penetration Testing Methodology

```
1. Reconnaissance    → Information gathering
2. Scanning          → Find open ports, services
3. Enumeration       → Extract users, shares, configs
4. Exploitation      → Exploit vulnerabilities
5. Post-exploitation → Lateral movement, persistence
6. Reporting         → Document findings & fixes
```

---

## 🔹 Reconnaissance Tools

```bash
# Passive recon (no direct contact with target)
whois target.com
dig target.com ANY
theHarvester -d target.com -b google

# Active recon
nmap -sV -sC -A target.com
nmap -p- --min-rate 5000 target.com

# Web recon
gobuster dir -u https://target.com -w /usr/share/wordlists/dirb/common.txt
nikto -h https://target.com
```

---

## 🔹 Common Exploitation Tools (Know What They Do)

| Tool | Purpose |
|---|---|
| Metasploit | Exploitation framework |
| Burp Suite | Web app testing (intercept proxy) |
| Nmap | Network scanning |
| Hydra | Brute force attacks |
| John the Ripper | Password cracking |
| Hashcat | GPU-accelerated password cracking |
| Mimikatz | Windows credential extraction |

📌 Use only in authorized environments (your own lab, CTF platforms).

---

## 🔹 Practice Platforms

| Platform | Purpose |
|---|---|
| TryHackMe | Guided learning (start here) |
| HackTheBox | Real-world scenarios |
| PicoCTF | CTF competitions |
| OWASP WebGoat | Intentionally vulnerable web app |
| DVWA | Damn Vulnerable Web App |

📌 Set up a local lab: Kali Linux + VulnHub machines

---

## 🔹 OWASP Top 10 (Web Security Must Know)

| Rank | Vulnerability |
|---|---|
| A01 | Broken Access Control |
| A02 | Cryptographic Failures |
| A03 | Injection (SQL, command) |
| A04 | Insecure Design |
| A05 | Security Misconfiguration |
| A06 | Vulnerable & Outdated Components |
| A07 | Identity & Authentication Failures |
| A08 | Software & Data Integrity Failures |
| A09 | Security Logging & Monitoring Failures |
| A10 | Server-Side Request Forgery (SSRF) |

---

# 🔄 PHASE 8 – DevSecOps

## Security built into the pipeline, not bolted on at the end.

---

## 🔹 DevSecOps vs Traditional Security

```
Traditional:
  Dev → Build → Test → [Security review] → Deploy
  Security = bottleneck, added late, expensive to fix

DevSecOps:
  Dev → [Security scan] → Build → [Security test] → Deploy
  Security = automated, continuous, cheap to fix early
```

---

## 🔹 Secure CI/CD Pipeline

```
Code Commit
    ↓
[SAST] Static Application Security Testing
    → Scan code for vulnerabilities before build
    → Tools: SonarQube, Semgrep, Bandit (Python)
    ↓
Build & Package
    ↓
[SCA] Software Composition Analysis
    → Scan dependencies for known CVEs
    → Tools: Snyk, Dependabot, OWASP Dependency-Check
    ↓
Container Build
    ↓
[Container Scan]
    → Scan Docker images for vulnerabilities
    → Tools: Trivy, AWS Inspector, Snyk
    ↓
Deploy to Staging
    ↓
[DAST] Dynamic Application Security Testing
    → Test running application for vulnerabilities
    → Tools: OWASP ZAP, Burp Suite Enterprise
    ↓
Deploy to Production
    ↓
[Runtime Security]
    → Monitor running containers
    → Tools: Falco, AWS GuardDuty
```

---

## 🔹 Trivy (Container Security Scanning)

```bash
# Install Trivy
apt install trivy

# Scan a Docker image
trivy image nginx:latest

# Scan with severity filter
trivy image --severity HIGH,CRITICAL nginx:latest

# Scan filesystem
trivy fs /path/to/project

# Scan IaC (Terraform, K8s manifests)
trivy config ./terraform/

# Output as JSON for CI/CD
trivy image --format json --output results.json myapp:latest
```

---

## 🔹 Secrets Detection

```bash
# Never commit secrets to Git
# Use pre-commit hooks to detect them

# Gitleaks - scan for secrets in git history
docker run -v ${PWD}:/path zricethezav/gitleaks detect --source /path

# Trufflehog
trufflehog git file://./myrepo

# .gitignore security
*.env
*.pem
*.key
*secret*
*credential*
config/secrets.yml
```

---

## 🔹 Infrastructure as Code Security

```bash
# Checkov - scan Terraform/CloudFormation for misconfigs
pip install checkov
checkov -d ./terraform/

# tfsec - Terraform security scanner
tfsec ./terraform/

# Example finding:
# CRITICAL: Security group allows unrestricted ingress on port 22
# Resource: aws_security_group.example
```

---

## 🔹 Kubernetes Security

```yaml
# Pod Security Context - never run as root
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```

```bash
# Kube-bench - CIS Kubernetes benchmark
docker run --rm aquasec/kube-bench

# Falco - runtime security (detect threats in running containers)
kubectl apply -f https://raw.githubusercontent.com/falcosecurity/deploy-falco/main/falco.yaml

# Check for risky RBAC
kubectl auth can-i --list --as=system:anonymous
```

---

# 📋 PHASE 9 – Compliance & GRC

## Legal accountability. This is what AI can never replace.

---

## 🔹 ISO 27001 (Global Standard)

```
ISO 27001 = Information Security Management System (ISMS)

Structure:
  Annex A Controls (93 controls across 4 domains):
  ├── Organizational controls
  ├── People controls
  ├── Physical controls
  └── Technological controls

Process:
  Plan → Do → Check → Act (PDCA cycle)

Certification steps:
  1. Gap analysis
  2. Risk assessment
  3. Implement controls
  4. Internal audit
  5. External audit (Stage 1 + Stage 2)
  6. Certification issued
  7. Annual surveillance audits
```

---

## 🔹 Japan-Specific: METI & 情報セキュリティ

| Framework | Purpose |
|---|---|
| 情報セキュリティ管理基準 | METI security management baseline |
| サイバーセキュリティ経営ガイドライン | Executive cybersecurity guidance |
| 重要インフラのサイバーセキュリティ | Critical infrastructure protection |
| 個人情報保護法 (PIPA) | Japan's privacy law (like GDPR) |

📌 **登録セキスペ (情報処理安全確保支援士)** — Japan's national security certification. METI is legally doubling the number of holders by 2030. This cert alone puts you in the top 5% of Japan's security market.

---

## 🔹 Europe: NIS2 & DORA (Your Bridge to European Market)

```
NIS2 (Network and Information Security Directive 2):
  → EU law enforced from October 2024
  → Covers: energy, transport, banking, health, digital infrastructure
  → Requires: risk management, incident reporting, supply chain security
  → Penalty: up to €10M or 2% of global turnover

DORA (Digital Operational Resilience Act):
  → EU law for financial sector
  → Covers: banks, insurers, investment firms, crypto
  → Requires: ICT risk management, incident classification, pen testing
  → Effective: January 2025
```

📌 Any company doing business in Europe must comply. Japan companies with EU operations (Toyota, Sony, SoftBank) need people who know both METI and NIS2/DORA frameworks. That is a very rare skill.

---

## 🔹 Risk Assessment Process

```
1. Asset Inventory
   └── What do we have? (servers, data, applications)

2. Threat Identification
   └── What could go wrong? (MITRE ATT&CK)

3. Vulnerability Assessment
   └── What weaknesses exist?

4. Risk Calculation
   └── Risk = Likelihood × Impact

5. Risk Treatment
   ├── Accept (low risk, not worth fixing)
   ├── Mitigate (add controls)
   ├── Transfer (insurance, outsource)
   └── Avoid (stop the risky activity)

6. Monitor & Review
   └── Regular reassessment
```

---

## 🔹 Certification Target This Phase

🎯 **情報処理安全確保支援士 (登録セキスペ)**
- Japan's highest nationally recognized security qualification
- Government-backed, METI-endorsed
- Exam: April and October each year
- Required for: government contracts, major enterprise security roles
- Study time: 3–6 months

🎯 **ISO 27001 Lead Implementer or Lead Auditor**
- Opens Europe market significantly
- Recognized globally
- Required for GRC / compliance track

---

# 🏗️ PHASE 10 – Cloud Security Architecture

## Designing security for entire organizations, not just individual systems.

---

## 🔹 Well-Architected Security Pillar (AWS)

```
5 Design Principles:
  1. Implement a strong identity foundation
  2. Enable traceability (logging everything)
  3. Apply security at all layers
  4. Automate security best practices
  5. Protect data in transit and at rest
  6. Keep people away from data (automate access)
  7. Prepare for security events (IR planning)
```

---

## 🔹 Multi-Account AWS Security Architecture

```
AWS Organizations
  │
  ├── Management Account (billing only)
  │
  ├── Security Account
  │     ├── CloudTrail (all accounts → here)
  │     ├── GuardDuty (delegated admin)
  │     ├── Security Hub (aggregated)
  │     └── SIEM integration
  │
  ├── Production Account
  │     └── Production workloads
  │
  ├── Staging Account
  │     └── Pre-production workloads
  │
  └── Dev Account
        └── Developer sandboxes
```

---

## 🔹 Landing Zone & Security Baseline

```
Every new AWS account gets automatically:
  ✅ CloudTrail enabled (all regions)
  ✅ GuardDuty enabled
  ✅ Security Hub enabled
  ✅ Config rules enforced (compliance)
  ✅ Default VPC deleted
  ✅ S3 public block on
  ✅ Root MFA enforced via SCP
  ✅ Centralized logging to security account
```

---

## 🔹 Service Control Policies (SCPs)

```json
// Deny disabling CloudTrail (nobody can turn off audit logs)
{
  "Effect": "Deny",
  "Action": [
    "cloudtrail:DeleteTrail",
    "cloudtrail:StopLogging",
    "cloudtrail:UpdateTrail"
  ],
  "Resource": "*"
}

// Deny leaving AWS Organizations
{
  "Effect": "Deny",
  "Action": "organizations:LeaveOrganization",
  "Resource": "*"
}

// Restrict to approved regions only
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": ["ap-northeast-1", "ap-northeast-3"]
    }
  }
}
```

---

## 🔹 Security Architecture Documentation

In Japan, architecture documentation is critical. You must be able to produce:

```
✅ Network security diagrams
✅ Data flow diagrams (show where sensitive data travels)
✅ Threat model documents
✅ Security control matrices
✅ Risk register
✅ Incident response playbooks
✅ ISMS documentation (for ISO 27001)
```

📌 Tools: draw.io, Lucidchart, PlantUML (for diagrams-as-code)

---

## 🔹 Certification Target This Phase

🎯 **CISSP (Certified Information Systems Security Professional)**
- Gold standard globally
- Required for Security Architect level positions
- Covers: 8 domains of security knowledge
- Prerequisite: 5 years security experience
- Study time: 3–6 months intensive
- Salary impact in Japan: ¥2M–¥3M increase

---

# 🧪 PHASE 11 – Hands-On Projects (CRITICAL)

## Theory means nothing without proof of skill. Build everything below.

---

## 🔹 Project 1 — AWS Security Hardening Lab

```
Build an insecure AWS environment → audit it → harden it

Steps:
1. Create a VPC with public EC2 (intentionally misconfigured)
2. Run AWS Security Hub → collect all findings
3. Fix every HIGH and CRITICAL finding
4. Document before/after with screenshots
5. Write a security audit report (Japanese + English)

Skills demonstrated:
→ AWS security services
→ IAM policy writing
→ Network security
→ Audit report writing (important for Japan)
```

---

## 🔹 Project 2 — SIEM Setup & Detection Rules

```
Build a working SIEM with real detection rules

Steps:
1. Set up Elastic SIEM or Microsoft Sentinel (free trial)
2. Forward logs from: Linux server, AWS CloudTrail, VPC Flow Logs
3. Write 5 custom detection rules:
   - Brute force SSH detection
   - Root account AWS usage
   - S3 bucket made public
   - New admin user created
   - Impossible travel detection
4. Trigger each alert intentionally → verify it fires
5. Write an incident response playbook for each

Skills demonstrated:
→ SIEM operation
→ KQL / SPL query writing
→ Alert tuning
→ IR documentation
```

---

## 🔹 Project 3 — DevSecOps Pipeline

```
Build a secure CI/CD pipeline from scratch

Steps:
1. Create a simple web app (any language)
2. Set up GitHub Actions pipeline
3. Add security stages:
   - Gitleaks (secrets scanning)
   - Semgrep (SAST)
   - Snyk (dependency scanning)
   - Trivy (container scanning)
   - Checkov (IaC scanning)
4. Intentionally add a vulnerability → confirm pipeline catches it
5. Document the pipeline with security rationale

Skills demonstrated:
→ DevSecOps implementation
→ Multiple security tools
→ CI/CD understanding
→ Security automation
```

---

## 🔹 Project 4 — Zero Trust Lab

```
Implement Zero Trust access model

Steps:
1. Set up AWS IAM Identity Center (SSO)
2. Configure MFA for all users
3. Implement least-privilege roles per team
4. Set up AWS Config rules to detect violations
5. Test: attempt access you shouldn't have → confirm denied
6. Document the Zero Trust architecture

Skills demonstrated:
→ Identity architecture
→ Zero Trust implementation
→ AWS IAM advanced
→ Security architecture documentation
```

---

## 🔹 Project 5 — Threat Modeling

```
Perform a real threat model on one of your previous projects

Method: STRIDE
  S - Spoofing identity
  T - Tampering with data
  R - Repudiation
  I - Information disclosure
  D - Denial of service
  E - Elevation of privilege

Deliverable: Threat model document including:
  → System diagram
  → STRIDE analysis per component
  → Risk rating per threat
  → Mitigation for each threat
  → Residual risk acceptance

Skills demonstrated:
→ Security architecture thinking
→ Risk assessment
→ Documentation quality (critical in Japan)
```

---

## 🔹 Where to Publish

```
✅ GitHub  — all code, IaC, pipelines, detection rules
✅ Zenn.dev — Japanese tech blog (very respected in Japan IT community)
✅ Qiita    — Japanese technical articles (essential for Japan job hunting)
✅ Portfolio PDF (Japanese) — architecture diagrams + project summaries
```

📌 Writing technical articles in Japanese on Zenn or Qiita significantly increases your visibility to Japanese recruiters.

---

# 🇯🇵 PHASE 12 – Japan Job Preparation (Cloud Security)

---

## 🔹 Target Job Titles in Japan

| Japanese | English Equivalent |
|---|---|
| クラウドセキュリティエンジニア | Cloud Security Engineer |
| セキュリティエンジニア | Security Engineer |
| インフラセキュリティエンジニア | Infrastructure Security Engineer |
| SOCエンジニア | SOC Engineer |
| DevSecOpsエンジニア | DevSecOps Engineer |
| 情報セキュリティエンジニア | Information Security Engineer |

---

## 🔹 Japanese Security Technical Vocabulary

| English | Japanese |
|---|---|
| Vulnerability | 脆弱性 (ぜいじゃくせい) |
| Incident | インシデント |
| Threat | 脅威 (きょうい) |
| Risk | リスク |
| Audit | 監査 (かんさ) |
| Compliance | コンプライアンス |
| Penetration test | 侵入テスト / ペネトレーションテスト |
| Firewall | ファイアウォール |
| Encryption | 暗号化 (あんごうか) |
| Authentication | 認証 (にんしょう) |
| Authorization | 認可 (にんか) |
| Incident response | インシデント対応 |
| Threat detection | 脅威検知 |
| Security audit | セキュリティ監査 |

---

## 🔹 Certifications Priority for Japan Market

| Priority | Certification | Impact |
|---|---|---|
| 🥇 Must Have | 情報処理安全確保支援士 (登録セキスペ) | Japan market top 5% |
| 🥇 Must Have | AWS Security Specialty | ¥1-2M salary increase |
| 🥈 High Value | CISSP | Senior/architect roles |
| 🥈 High Value | CompTIA Security+ | Entry validation |
| 🥉 Good To Have | ISO 27001 Lead Implementer | GRC / Europe bridge |
| 🥉 Good To Have | CEH or OSCP | Offensive security validation |

---

## 🔹 Target Companies in Japan

**Foreign/Global (English OK, higher salary):**
- AWS Japan, Microsoft Japan, Google Cloud Japan
- Palo Alto Networks Japan
- CrowdStrike Japan
- Trend Micro (founded in Japan, global)
- Secureworks Japan

**Japanese Enterprise (Japanese required, stable):**
- NTT Communications / NTT Data
- Fujitsu (massive security division)
- NEC Security
- Hitachi Systems Security
- NTTDATA Security

**Financial Sector (highest paid):**
- Nomura, SMBC, Mizuho (in-house security)
- Any megabank's CSIRT team

---

## 🔹 JLPT for Security Roles

| Level | What You Can Do |
|---|---|
| N3 | Read incident tickets, basic meetings |
| N2 | Security review meetings, write reports |
| **N1** | Lead security discussions, CISO track, maximum salary premium |

📌 Bilingual security engineers in Japan command **10–30% salary premium** over monolingual counterparts.

---

# 📅 PHASE 13 – Realistic Timeline

| Time | Phase | Milestone |
|---|---|---|
| Month 1–2 | Phase 1–2 | Security fundamentals + Linux hardening |
| Month 3 | Phase 3 | Network security + CompTIA Security+ exam |
| Month 4–5 | Phase 4 | AWS Cloud Security deep dive |
| Month 6 | Phase 5 | IAM + Zero Trust |
| Month 7 | Phase 6–7 | SOC/SIEM setup + offensive basics |
| Month 8 | Phase 8 | DevSecOps pipeline project |
| Month 9 | Phase 9 | Start 登録セキスペ study + GRC basics |
| Month 10 | Phase 10 | Architecture + AWS Security Specialty exam |
| Month 11–12 | Phase 11 | All 5 hands-on projects |
| Month 13–15 | Phase 12 | 登録セキスペ exam + Japan job hunting |
| Month 18+ | — | CISSP study begins |

---

# 🎯 Complete Career Path

```
WHERE YOU ARE NOW
インフラエンジニア (1+ year Japan)
¥5M–¥7M
        ↓  (This Roadmap — 12-15 months)

PHASE 13 TARGET
Cloud Security Engineer
¥9M–¥13M | Year 3–4
Core certs: Security+, AWS Security Specialty, 登録セキスペ
        ↓  (2–3 more years)

PHASE 14 TARGET
Security Architect / GRC Lead
¥14M–¥18M | Year 6–7
Core certs: CISSP, ISO 27001
        ↓  (Optional, Year 7+)

PHASE 15 TARGET
Japan → Europe Transition
€120,000–€197,000
Countries: Switzerland, Netherlands, Germany, UK
Bridge: NIS2/DORA + ISO 27001 + CISSP + Japan enterprise experience
```

---

## 🔰 Why This Path Is AI-Resistant

| What AI Can Do | What You Will Do |
|---|---|
| Scan for known vulnerabilities | Design security architecture for entire organizations |
| Generate security reports | Sign off legally on compliance — AI cannot |
| Detect patterns in logs | Make judgment calls during live incidents |
| Suggest IAM policies | Own accountability when a breach occurs |
| Run automated pen tests | Hunt novel zero-days with creative thinking |

**The more powerful AI gets at attacking systems, the more organizations need skilled humans to defend them. You are building the skills that sit above the automation layer — permanently.**

---
*Roadmap Version: 2026 | Japan + Europe Market | Built on: 1+ Year インフラエンジニア Experience*
