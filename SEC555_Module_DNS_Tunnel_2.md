# SEC555 — DNS tunnel lab (module notes)

## Lab summary

Traffic was generated from a **Windows server** toward a **Slingshot** (or lab C2) host using **DNS encapsulation**. **Sysmon** and **Wireshark** were used to observe how tunneling appears at the **endpoint** versus **on the wire**.

## Commands / tooling (conceptual)

| Tool | Role |
|------|------|
| **Sysmon** (Event ID 22 / DNS client, depending on config) | DNS query telemetry on the Windows host |
| **Wireshark** | Filter `dns` to inspect QNAME length, entropy, subdomains, and response patterns |
| **Zeek / Suricata** (if used in class) | Aggregate DNS logs for volume and rare-query analytics |

## What to document

- Baseline **benign** DNS for the host, then **delta** during tunneling.
- **Query rate**, **TXT** / **NULL** record abuse, and **CNAME** chains common in encoders.
- Any **detection rules** drafted during the exercise (thresholds, allowlists, response actions).

## QC

Replace this stub with your **class-specific** PCAP paths, time windows, and Slingshot listener configuration when archiving notes.
