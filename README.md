# Cap 191.2.4 — DoS Attack Investigation & Mitigation

This walkthrough documents the step-by-step process of identifying and mitigating a **Denial of Service (DoS) attack** using Wireshark, Windows Command Prompt, and Windows Defender Firewall.

---

## Table of Contents

- [Step 1 — Download & Initial Inspection](#step-1--download--initial-inspection)
- [Step 2 — TCP Stream Analysis](#step-2--tcp-stream-analysis)
- [Step 3 — Protocol Hierarchy & Process Identification](#step-3--protocol-hierarchy--process-identification)
- [Step 4 — Killing the Malicious Port](#step-4--killing-the-malicious-port)
- [Step 5 — Opening Windows Defender Firewall](#step-5--opening-windows-defender-firewall)
- [Step 6 — Creating a New Inbound Rule](#step-6--creating-a-new-inbound-rule)
- [Step 7 — Configuring the Rule (Port & Protocol)](#step-7--configuring-the-rule-port--protocol)
- [Step 8 — Blocking the Connection](#step-8--blocking-the-connection)
- [Step 9 — Naming & Finalizing the Rule](#step-9--naming--finalizing-the-rule)

---

## Step 1 — Download & Initial Inspection

<img width="742" height="550" alt="Image" src="https://github.com/user-attachments/assets/07006730-f6d4-45f0-8eba-db4c2b00d16b" />

<img width="625" height="560" alt="Image" src="https://github.com/user-attachments/assets/11e7ae45-65c6-4605-aaf6-1c42ef05b89f" />

The `.pcap` file was downloaded and opened in Wireshark. On initial inspection, anomalous behavior was observed between **packets 4 and 6**, with packets 6 onward displaying patterns consistent with a DoS attack. Key details visible in this view included the Source IP address, Destination IP address, and packet timestamps.


---

## Step 2 — TCP Stream Analysis

<img width="516" height="577" alt="Image" src="https://github.com/user-attachments/assets/a2e310c6-6e82-46f0-9f14-32c59e12ff9c" />

<img width="454" height="517" alt="Image" src="https://github.com/user-attachments/assets/6f0402c7-bcbb-4cc5-a3da-0785d6f1ecd5" />

<img width="516" height="577" alt="Image" src="https://github.com/user-attachments/assets/4c575f80-be05-4045-8478-5efad4fe7b1a" />

A **TCP Stream** was executed to better understand the communication between the Source IP and the Destination IP. The stream output was first displayed in **ASCII**, then switched to **YAML** for plain-text readability. The YAML view revealed host information (`34.307.243.93` ↔ `10.2.2.5`), timestamps, packet details, and a **suspicious/unusual port number**, indicating potentially malicious activity.


---

## Step 3 — Protocol Hierarchy & Process Identification

<img width="566" height="395" alt="Image" src="https://github.com/user-attachments/assets/228c646d-1ecb-4f38-aeca-c957302bf760" />

<img width="566" height="382" alt="Image" src="https://github.com/user-attachments/assets/2e7cfc3e-df7f-49ab-8927-18ff54e7040a" />

<img width="721" height="242" alt="Image" src="https://github.com/user-attachments/assets/6e6bf8c2-9471-4de1-a202-4eae6edd0731" />

A **Protocol Hierarchy Statistics** report was generated in Wireshark, revealing the following:

| Protocol | % of Packets | Avg Packet Size |
|----------|-------------|-----------------|
| IPv6 / ICMPv6 | **83.6%** | 18.6 bytes |
| QUIC IETF | 16.0% | 62.2 bytes |

Key findings:
- The network was being **flooded with Router Advertisements** from multiple different MAC addresses within milliseconds — a hallmark of a DoS attack.
- An ARP discovery request was observed: *"Who has 10.2.2.1? Tell 10.2.2.5"*, along with an associated MAC address.

To identify the responsible process, the following command was run:

```cmd
netstat -ano | findstr :47394
```

This narrowed network activity to **port 47394** and returned the associated **Process ID (PID)**.


---

## Step 4 — Killing the Malicious Port

<img width="721" height="325" alt="Image" src="https://github.com/user-attachments/assets/ac67a018-a40c-448e-ba44-0e0df690f80c" />

With the PID identified, the process was terminated to immediately stop the port from being exploited:

```cmd
sudo kill /[PID]
```

This closed the malicious port and halted its use for further attacks.

---

## Step 5 — Opening Windows Defender Firewall

<img width="781" height="514" alt="Image" src="https://github.com/user-attachments/assets/c57b3408-601d-4053-8c2e-0ad61d5859d6" />

<img width="781" height="470" alt="Image" src="https://github.com/user-attachments/assets/9d4a8c2a-7f43-474c-952e-d29b345cea31" />

To create a permanent firewall rule, **Windows Defender Firewall** was opened. From the main screen, **Advanced Settings** was selected to access the advanced firewall configuration panel.

---

## Step 6 — Creating a New Inbound Rule

<img width="760" height="438" alt="Image" src="https://github.com/user-attachments/assets/e2ee35e6-9c51-4b63-83dd-1c64277a946d" />

<img width="707" height="516" alt="Image" src="https://github.com/user-attachments/assets/01c92cd7-8cac-4796-8cd7-cad0da37fd57" />

Inside the Advanced Settings panel, the **Inbound Rules** section was accessed. On the right-hand side, **New Rule** was double-clicked to open the New Inbound Rule Wizard, which allows configuration of custom traffic-blocking parameters.

---

## Step 7 — Configuring the Rule (Port & Protocol)

<img width="692" height="528" alt="Image" src="https://github.com/user-attachments/assets/b506052d-dbf5-4602-86a2-9f7db2ed0f69" />

<img width="807" height="567" alt="Image" src="https://github.com/user-attachments/assets/bbff30ec-07a6-49e3-aed9-bd4fada89c10" />

The rule was configured as follows:

- **Rule Type:** Port — selected because the attack was carried out via a specific port.
- **Protocol:** UDP — confirmed by Protocol Hierarchy Statistics showing the malicious packets used **UDP, not TCP**.
- **Port Specified:** `47394` — the exact port identified during analysis.

---

## Step 8 — Blocking the Connection

<img width="696" height="568" alt="Image" src="https://github.com/user-attachments/assets/ee6f3b26-25b6-400c-8b64-6076c87c0f4d" />

<img width="788" height="643" alt="Image" src="https://github.com/user-attachments/assets/3ef95075-9906-4b88-9acb-f5411c4cfa20" />

The rule action was set to **Block the connection** to prevent any traffic from utilizing this port. The rule was then applied across all network profiles:

- ✅ Domain
- ✅ Private
- ✅ Public

---

## Step 9 — Naming & Finalizing the Rule

<img width="1011" height="703" alt="Image" src="https://github.com/user-attachments/assets/13792c0d-93a5-4933-b100-cb2747b24ef9" />

A descriptive **name** and optional **description** were provided for the new firewall rule before saving. With this final step completed, the inbound rule was active and the DoS attack was successfully mitigated.

---

## Summary

| Step | Action | Tool Used |
|------|--------|-----------|
| 1 | Opened and inspected pcap file | Wireshark |
| 2 | Analyzed TCP stream in YAML format | Wireshark |
| 3 | Ran Protocol Hierarchy Statistics | Wireshark |
| 4 | Identified PID via `netstat` | Command Prompt |
| 5 | Terminated malicious process via `taskkill` | Command Prompt |
| 6 | Opened Windows Defender Firewall | Windows Defender |
| 7 | Created a new Inbound Rule | Windows Defender |
| 8 | Configured port/protocol (UDP/47394) | Windows Defender |
| 9 | Blocked connection on all profiles | Windows Defender |
| 10 | Named and finalized the firewall rule | Windows Defender |

---

## Tools & Technologies

- [Wireshark](https://www.wireshark.org/) — Packet capture and network analysis
- Windows Command Prompt — Process identification and termination
- Windows Defender Firewall (Advanced Security) — Inbound rule configuration


<img width="1011" height="703" alt="Image" src="https://github.com/user-attachments/assets/349204de-c15c-401c-8947-fe354719a427" />

<img width="1858" height="49" alt="Image" src="https://github.com/user-attachments/assets/e2b6e20d-1a9c-494c-bed1-dd7c92da3916" />
