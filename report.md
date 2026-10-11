# Lab 1: Network Traffic Analysis & Investigation

**Author:** Djerou Fouzia  
**Environment:** Isolated VirtualBox Lab  
**Target:** Metasploitable 2

## 1. Objective

The main objective of this lab was to gain practical experience in network traffic analysis and investigation, identify open ports and running services, examine TCP communication patterns, and produce a professional technical investigation report based on collected evidence.

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Attacking machine | Kali Linux |
| Target machine | Metasploitable 2 |
| Virtualization | VirtualBox |
| Network configuration | Host-only (`HostNetwork`) |
| Kali IP address | `192.168.56.7` |
| Target IP address | `192.168.56.8` |
| Scanning tool | Nmap |
| Traffic analysis tool | Wireshark |

The two virtual machines communicated through a host-only network configured for the lab. This setup provided an isolated environment for authorized testing.

## 3. Methodology

### 3.1. IP Address Identification

The IP addresses of the virtual machines were identified using `ip addr` on Kali Linux and `ifconfig` on Metasploitable 2.

### 3.2. Connectivity Verification

The `ping` command was used to verify network connectivity between the two virtual machines. Successful replies confirmed that the target was reachable during the test.

### 3.3. Baseline Traffic Capture

A baseline packet capture was collected before the scan to observe background network traffic. The capture was saved for comparison with the scan-related traffic.

### 3.4. Nmap Port Scan

The following command was executed from Kali Linux:

```bash
nmap -sV --top-ports 100 192.168.56.8
```

The command scanned the 100 most common TCP ports and attempted to identify the versions of detected services.

### 3.5. Wireshark Traffic Analysis

Wireshark was used to inspect captured packets, including TCP SYN, SYN-ACK, and RST responses. Display filters were applied to examine traffic between Kali Linux and Metasploitable 2.

**Filter for initial TCP SYN packets:**

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

**Filter for SYN-ACK responses from the target:**

```text
ip.src == 192.168.56.8 && ip.dst == 192.168.56.7 && tcp.flags.syn == 1 && tcp.flags.ack == 1
```

**Filter for TCP reset packets from the target:**

```text
ip.src == 192.168.56.8 && ip.dst == 192.168.56.7 && tcp.flags.reset == 1
```

TCP stream information was also examined to understand whether conversations contained data, ended with a reset, or appeared incomplete.

## 4. Findings and Evidence

### Finding 1: TCP Port Scanning Activity

**Evidence**

Multiple TCP SYN packets were observed from Kali Linux (`192.168.56.7`) to Metasploitable 2 (`192.168.56.8`), targeting different destination ports within a short time interval.

**Analysis**

This pattern is consistent with TCP port scanning, which is used to identify accessible ports on a target system. The packet pattern, when correlated with the Nmap command and scan results, supports the conclusion that a port scan was performed.

**Conclusion**

TCP port scanning activity was observed during the lab investigation.

**Limitation**

The packet pattern alone does not identify the generating application with certainty. Attribution to Nmap should be supported by correlating packet timestamps with the scan execution.

### Finding 2: Open Port and Service Identification

**Evidence**

Nmap reported 18 open ports and 82 closed ports among the 100 TCP ports scanned.

Examples of identified services included:

| Port | Service | Reported version or identification |
|---|---|---|
| 21 | FTP | vsftpd 2.3.4 |
| 22 | SSH | OpenSSH 4.7p1 |
| 23 | Telnet | Linux telnetd |
| 25 | SMTP | Postfix smtpd |
| 53 | DNS | ISC BIND 9.4.2 |
| 80 | HTTP | Apache 2.2.8 |
| 139/445 | SMB | Samba smbd 3.X–4.X |
| 3306 | MySQL | MySQL 5.0.51a |

**Analysis**

The scan identified multiple network services listening on the target. Some reported versions are old and warrant further security assessment.

**Conclusion**

Multiple services were accessible on the target during the scan. The presence of open ports does not, by itself, confirm that the target is vulnerable.

**Limitation**

The results cover only the 100 TCP ports scanned. Version detection does not independently confirm that a service is exploitable.

### Finding 3: FTP Service Version Disclosure

**Evidence**

A response from the target's FTP service included the following banner:

```text
220 (vsFTPd 2.3.4)
```

The observed response came from `192.168.56.8:21` to Kali Linux.

**Analysis**

The banner disclosed the FTP server software and its reported version. This information can help an investigator determine whether the installed software is affected by known vulnerabilities.

**Conclusion**

The FTP service responded and disclosed its reported version. A vulnerability was not confirmed by the banner alone.

**Recommendation**

Verify the installed version and build against reliable vulnerability references, then update to a supported version if necessary. Use SFTP or FTPS when encrypted file transfer is required.

**Limitation**

The banner alone does not establish exploitability or prove that the target was successfully compromised.

## 5. Severity and Risk Assessment

The observed port scan is evidence of reconnaissance activity, not proof of a successful attack. The existence of open ports is not sufficient to establish a vulnerability.

The FTP service version warrants further verification because version information may correspond to known security issues. A final severity rating should be assigned only after verifying the applicable vulnerability, its impact, exploit
