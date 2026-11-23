# Baseline test 1: firewall config review

## What I tested

This is my first proper test of Fortis after getting it running. I wanted to see how well it finds problems in a config file before I change anything, so I can compare it later.

- **Date:** 23 Nov 2026
- **Model:** Qwen3.5 2B (the Q4_K_M version), running on my laptop's CPU (no NVIDIA GPU, 16 GB RAM)
- **File I uploaded:** `training/sample-lab-docs/firewall-config.txt`
- **Mode:** General
- **What I asked:** "What are the security issues in this firewall config?"
- **How long it took:** about 30–60 seconds. This was the first message, so it also had to load the model. I need to time it properly with a stopwatch next time.
- **Screenshots:** `docs/evidence/15-first-answer-1.png` to `15-first-answer-3.png`

To know what the "right answer" was, I used the list in the lab guide (`training/01-lab-exercises.md`, Lab 2) and also read through the config file myself to find anything else that was wrong.

## Did it find the problems?

| # | Problem in the file | Where in the file | Found? | My notes |
|---|---|---|---|---|
| 1 | Firewall allows all traffic in from the internet | `permit ip any any` on the WAN interface | Partly | It found it but only called it Medium. The lab guide says this is the biggest risk |
| 2 | Weak / default passwords | `admin123`, `cisco123`, `P@ssw0rd123` | Yes | |
| 3 | Telnet turned on (not encrypted) | `transport input telnet` | No | Missed |
| 4 | Web management over plain HTTP | `ip http server` | No | Missed |
| 5 | Weak VPN encryption and a password in plain text | `des`, `md5`, `mysecretpass` | Partly | Found DES/MD5, but missed the plaintext VPN password |
| 6 | SNMP using the default "public" string | `snmp-server community public RO` | Yes | |
| 7 | Old SSH version 1 | `ip ssh version 1` | Yes | |
| 8 | No logs sent to a log server | `no logging host` | Yes | |
| 9 | OSPF routing with no authentication | `router ospf 1` | No | Missed |

**Result:** it found 4, partly found 2 and missed 3. If I count a "partly" as half a point, that's **5 out of 9 (about 56%)**.

## What did it get wrong?

As well as missing things, some of what it said was wrong or made up:

| # | What Fortis said | What's wrong with it |
|---|---|---|
| 1 | Passwords link to NIST control **RA-3** | RA-3 is about risk assessment. Passwords should be **IA-5** (authenticator management) |
| 2 | Called **SC-1** "Security Configuration" | SC-1 is actually "Policy and Procedures" |
| 3 | Mentioned **"CIS-AC-2"** and **"CIS-N-1"** | These don't seem to exist. CIS Controls are just numbered 1 to 18, so it looks like it made them up |
| 4 | Mentioned **GDPR** four times | Doesn't really make sense here, because a firewall config doesn't have personal data in it |
| 5 | Said stolen passwords are a **"DoS"** attack | That's wrong: stolen passwords let someone log in, they don't take the system down |
| 6 | Said the passwords **"cannot be changed"** | They can easily be changed in the config |
| 7 | Said to **store** `admin123` and `cisco123` in a password manager | Weak passwords should be replaced, not saved |
| 8 | Said to "edit firewall-config.txt" | It's treating the device's config like a normal text file |
| 9 | Said the open firewall rule is only Medium | It explained this badly. The rule itself is what lets attackers in |

So there were **9 mistakes in one answer**.

**Still to do:** I need to check points 1–3 myself on the official NIST and CIS websites before I put them in my report.

## What I noticed

- The answer *looks* really professional (colour-coded severity, headings, a remediation plan). That actually makes the mistakes easier to miss.
- It never made up anything that wasn't in the file. All the passwords and settings it quoted were real.
- The worst mistakes were in the framework references (NIST, CIS, GDPR). Those look official, so an analyst might just trust them without checking.

## Ideas for improvements (Stage 3)

- Add a simple rule-based check for well-known bad settings (Telnet, `permit ip any any`, `ip http server`, OSPF without authentication), so Fortis doesn't miss them.
- Check any NIST or CIS control IDs the AI mentions against the real framework data that Fortis already has in `core/frameworks`, and flag the ones that don't exist.
- Then run this exact same test again and compare the before and after results.
