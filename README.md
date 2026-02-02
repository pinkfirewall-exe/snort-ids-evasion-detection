# Snort IDS – Detecting Network Evasion Techniques Through Detection Engineering

## Overview
This project demonstrates how common IDS evasion techniques can bypass a minimally configured Intrusion Detection System, and how creating custom rules can improve its detection accuracy.

Using Snort in IDS mode, I reproduced three real-world evasion techniques and then designed custom rules to detect them reliably:

- Packet fragmentation
- Overlapping fragmentation simulation
- Decoy-based scanning

Each attack was executed against an intentionally "untrained" IDS to highlight detection gaps. Custom rules were then written to transform attacker behaviour into actionable alerts.

---

## Why This Project Matters
Attackers rarely trigger obvious signatures. Instead, they exploit protocol edge cases, ambiguity in packet reassembly, and attribution confusion.

This project demonstrates:
- Practical IDS evasion techniques
- Behaviour-driven detection logic
- Blue-team thinking aligned with MITRE ATT&CK
- How IDS tools must be continuously tuned to remain effective

This mirrors real SOC detection engineering workflows.

---

## Lab Environment
- **IDS:** Ubuntu Linux running Snort (IDS mode)
- **Attacker:** Kali Linux
- **Network:** VirtualBox Internal Network
- **Tools:** Snort, Nmap

All default Snort rules were disabled to observe baseline behaviour.

---

## Attack-Detect-Train Cycle

For each technique, I followed an iterative process:

i) Run the attack with no custom rule.

ii)	Observe Snort’s baseline behaviour, typically meaning no alert appeared.

iii) Create a rule tailored to the attack pattern.

iv)	Re-run the attack to confirm that Snort now detected it.

v)	Evaluate the strengths and weaknesses of the resulting alert.

---

## Evasion Techniques & Detection Strategy

### 1. Packet Fragmentation
**Attack goal:** Evade signature inspection by splitting packets into tiny fragments.

**Command:**
```bash
nmap -sS -f <IDS_IP>
```
The -f flag forces Nmap to break packets into tiny IP fragmenta, usually 8-byte chunks
### Observed behaviour (before rules):

Snort received traffic but produced no alerts 

Fragmented packets were treated as normal traffic

### Detection engineering approach:

Detect non-initial IP fragments using fragment offsets

Treat fragmentation itself as suspicious behaviour

### 2. Overlapping Fragmentation Simulation

**Attack goal:** Abuse packet reassembly ambiguity by forcing extremely small fragments, simulating conditions where IDS and endpoints may reassemble packets differently.

**Command:**
```bash
nmap -sS -f --mtu 16 <IDS_IP>
```

### Observed behaviour (before rules):

Snort silently reassembled fragments

No alerts were generated

### Detection engineering approach:

Alert on packets that are fragmented and abnormally small

Focus on behaviour that is extremely rare in legitimate traffic

### 3. Decoy Scan

**Attack goal:** Obfuscate attacker attribution by injecting multiple spoofed source IP addresses alongside the real scan traffic.

**Command:**
```bash
nmap -sS -D RND:5 <IDS_IP>
```

### Observed behaviour (before rules):

Multiple apparent sources masked the true attacker

No scan detection alerts were triggered

### Detection engineering approach:

Track SYN packet volume by destination rather than source

Detect distributed and decoy-based reconnaissance patterns

---

## Custom Detection Logic

Each evasion technique was addressed using behaviour-based Snort rules rather than payload-specific signatures.

Example: Packet Fragmentation Rule
# Detect non-initial IP fragments used for IDS evasion
# Technique: Packet Fragmentation
# ATT&CK: Defense Evasion
```
alert ip any any -> any any (
    msg:"EVASION: Fragmented packet (non-zero offset)";
    fragoffset:>0;
    sid:5002001;
    rev:1;
)
```
All rules are documented in the rules/ directory with comments explaining:

- the behaviour being detected
- the evasion technique addressed
- the defensive reasoning behind the rule

---

## Results
In all three techniques, the techniques were not detected before the rules, but after the rules, they were	detected

Each detection was validated by re-running the attack and observing real-time alerts on the Snort console.

---

## MITRE ATT&CK Mapping

### Evasion Technique to the 	ATT&CK Technique / Tactic
- In packet fragmentation, the attack technique was defense evasion
- In overlapping Fragmentation,	the attack technique was defense evasion
- In decoy scan,	the attack technique was T1046 – Network Service Scanning

Mapping detections to MITRE ATT&CK helps align IDS coverage with real adversary tradecraft and improves detection strategy visibility.

---

## Key Takeaways

- IDS tools are only as effective as their detection logic
- Fragmentation and attribution abuse remain effective evasion techniques
- Behaviour-based detection is more resilient than static signatures
- Understanding attacker tradecraft is essential for defensive success
- Custom rule engineering significantly improves IDS visibility

---
