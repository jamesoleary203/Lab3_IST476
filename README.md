# Network Reconnaissance and Traffic Analysis

**Course:** IST 476 Applied Information Security, Syracuse University iSchool
**Lab:** Assessing the Network with Common Security Tools (Jones & Bartlett *Ethical Hacking, 4e*)
**Author:** James O'Leary

---

## What this lab covers

A ground-up look at how a host discovers its neighbors and how a scanner behaves at the
packet level. I mapped the local segment with native Windows utilities, watched the
resulting ICMP and ARP traffic in Wireshark, then ran Nmap against the firewall and read
the difference between scan types in the capture rather than trusting the tool's summary.

## Environment

| Host | Role | Student-adapter IP | MAC |
|---|---|---|---|
| `vWorkstation` | Analyst workstation | 172.30.0.2 | 00-0C-29-1D-B6-EF |
| `TargetWindows01` | Scan and ping target | 172.30.0.10 | 00-0C-29-62-58-CE |
| `pfSense` | Firewall, internal interface | 172.30.0.1 | — |

## Method and observations

**Host discovery with native utilities.** `ipconfig /all` on both machines confirmed the
segment layout and tied each IP to its MAC. A `ping` to `TargetWindows01` returned four
clean echo replies, and `arp -a` afterward showed a new dynamic entry for the target's
MAC. That entry is the evidence that Layer 2 resolution happened, not just that the ping
succeeded.

**ICMP in Wireshark.** The `icmp` display filter isolated the echo request and its reply,
matched by sequence number. This is the baseline both scans get compared against.

**ARP in Wireshark.** Filtering on `eth.src == 00:0c:29:1d:b6:ef and arp` surfaced the
broadcast ARP request where the workstation asks who owns `172.30.0.1`. Before any TCP
touches the firewall, the host has to resolve its MAC, and that resolution is visible on
the wire.

**Nmap against the firewall.** An intense scan (`nmap -T4 -A -v 172.30.0.1`) enumerated
three open TCP ports on the pfSense internal interface:

| Port | Service | Version |
|---|---|---|
| 22 | SSH | OpenSSH 7.5 |
| 53 | DNS | — |
| 80 | HTTP | nginx |

## Scan comparison

The useful part of this lab is watching the same target through different scans.

- **Ping scan.** No ICMP to or from the firewall at all. Because Nmap ran with admin
  rights on the same subnet, it reached for an ARP request to decide the host was up,
  not an ICMP echo. Host discovery on a local segment does not have to look like a ping.
- **Regular scan.** Opens the same way with ARP, then moves to TCP against common ports.
  The discovery step is identical to the ping scan. The probing is what is new.
- **Intense scan.** Heavier ICMP and ARP, plus the parts that make it loud: service and
  version detection and the default NSE scripts running against the target.

## Defensive reading

The lab's recon maps onto three controls worth naming, since the whole point of scanning
a firewall is to understand what an attacker would see first.

1. **Edge-device hardening and patching.** Every open service on an internet-facing
   device needs a reason to be there, and known flaws in edge equipment get exploited
   fast. Verizon's 2025 DBIR put exploitation of edge devices and VPNs at a jump from 3%
   to 22% of breaches. The pfSense ports in this lab are the kind of surface that review
   targets.
2. **SASE with zero-trust network access.** Grant remote access on identity and device
   posture instead of handing out a flat VPN tunnel, which narrows what a stolen account
   can reach.
3. **Phishing-resistant MFA on admin logins.** A single stolen admin credential should
   not be enough to take over a router. This protects the management plane, where one
   compromise reaches the whole WAN.

## References

- Australian Signals Directorate. (2025). *Mitigation strategies for edge devices: Executive guidance.*
- Gillis, A. S., & Yasar, K. (2024). *What is secure access service edge (SASE)?* TechTarget.
- Poireault, K. (2025). *Verizon's DBIR reveals 34% jump in vulnerability exploitation.* Infosecurity Magazine.
