## **Penetration Test Report: Lord of the Root v1.0**

### **1. Executive Summary**

The objective of this assessment was to identify and exploit vulnerabilities within the **Lord of the Root** target environment. The engagement resulted in a full system compromise (Root access). The primary attack vectors included SQL injection leading to credential harvesting and Local Privilege Escalation (LPE) via a User Defined Function (UDF) exploit within a misconfigured MySQL service.

### **2. Technical Vulnerability Analysis**

#### **Vulnerability A: Time-Based SUID Binary Rotation (Anti-Tamper)**

- **Description:** The system featured three binaries (`/SECRET/door1, 2, 3`) that rotated vulnerability status every few minutes.
    
- **Impact:** Created a moving target for buffer overflow attempts, complicating standard exploitation workflows.
    
- **Mitigation:** Standardize binary permissions and remove SUID bits from non-essential files.
    

#### **Vulnerability B: MySQL Service Misconfiguration (UDF Injection)**

- **Description:** The MySQL service was running with administrative (root) privileges and allowed the loading of external shared libraries.
    
- **Impact:** **Critical.** Allowed a low-privileged user to execute arbitrary system commands as the root user.
    

---

### **3. Exploitation Walkthrough (The "Kill Chain")**

#### **Phase 1: Enumeration & Foothold**

- Identified web service on port **1337**.
    
- Discovered a base64-encoded hidden directory in the 404 page source.
    
- Utilized **SQLMap** to exploit a time-based blind SQL injection in the login form, recovering credentials for the user `smeagol`.
    

#### **Phase 2: Local Privilege Escalation (The Pivot)**

- **Lateral Movement:** Accessed the system via SSH as `smeagol`.
    
- **Service Analysis:** Identified MySQL running as root. Logged in using discovered credentials (`darkshadow`).
    
- **UDF Deployment:** 1. Compiled a custom C-based shared object (`raptor_udf2.so`) on the target system. 2. Injected the library into the MySQL plugin directory via a `DUMPFILE` command. 3. Mapped the `do_system` function to the library, granting the ability to execute bash commands through SQL queries.
    

#### **Phase 3: Post-Exploitation & Root Access**

- Executed the `chpasswd` command through the MySQL `do_system` function to reset the system root password.
    
- Successfully transitioned to the root user via `su -`.
    
- **Proof of Concept:** Retrieved the final flag from `/root/Flag.txt`.
    

---

### **4. Lessons Learned**

- **Adaptability:** When the rotating SUID binaries proved unstable due to ASLR and timing, the attack was successfully pivoted to a service-based vulnerability.
    
- **Tool Proficiency:** Demonstrated the ability to compile exploits manually on-target when pre-built tools were unavailable or corrupted.
    

---