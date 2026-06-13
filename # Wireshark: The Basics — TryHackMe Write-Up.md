# Wireshark: The Basics — TryHackMe Write-Up
<p>
  Platform: TryHackMe  
  Path: Cybersecurity 101 (Networking)  
  Difficulty: Easy  
  Date Completed: 10-06-2026  
  Room Link: https://tryhackme.com/room/wiresharkthebasics  
  Room Type: Guided (Theory with Walkthrough and hands-on questions with attached VM)
</p>  
---
## Room Summary
This room introduces Wireshark — the industry-standard open-source
network packet analyser — and teaches you how to navigate its interface,
load packet captures (PCAPs), dissect packets layer by layer, navigate
through traffic, and apply basic filtering. It is part of the Network
Security & Traffic Analysis module and directly relevant to day-to-day
SOC analyst work.
**Why this matters in the real world:** Every SOC analyst needs to be
comfortable reading packet captures. Whether you're investigating a
phishing campaign, spotting lateral movement, or triaging a potential
data exfiltration event, Wireshark (or a tool built on the same
libraries like NetworkMiner or Zeek) is in the toolkit. This room builds
the foundation everything else sits on.
---
## Learning Objectives
- Understand what Wireshark is and what it can (and cannot) do
- Navigate the Wireshark GUI confidently
- Open and inspect PCAP/PCAPNG files
- Dissect packets across OSI layers
- Navigate to specific packets using search, go-to, and bookmarking
- Apply and use display filters and colour rules
- Export objects (files) embedded within packet captures
---
## Tools Used
| Tool | Version / Notes |
|---|---|
| Wireshark | Provided in TryHackMe VM |
| Exercise.pcapng | Provided capture file (used for questions) |
| http1.pcapng | Provided capture file (used to follow along with
screenshots) |
---
## Key Concepts
### What is Wireshark?
Wireshark is an **open-source, cross-platform packet analyser**. It can:
- Capture live network traffic from an interface
- Open and analyse saved capture files (`.pcap`, `.pcapng`)
- Dissect hundreds of protocols automatically
**Important distinction:** Wireshark is NOT an Intrusion Detection
System (IDS). It captures and presents data — it does NOT alert you. The
analyst's expertise is what turns raw packets into actionable
intelligence.
### The Wireshark Interface
The UI has five main areas:
```
"##################################################$
% Menu Bar (File, Edit, View, Analyse, Statistics) %
&##################################################'
Main Toolbar (shortcuts) %
&##################################################'
Display Filter Bar [e.g. http or ip.src==...] %
&##################################################'
%
| Dest | Protocol | Info) %
&##################################################'
(expandable protocol tree) %
&##################################################'
(hex + ASCII) %
&##################################################'
% Status Bar (profile | filter | packet count) %
(##################################################)
```
% % % % Packet List Pane (No. | Time | Source % Packet Details Pane % Packet Bytes Pane ### Packet Dissection & OSI Layers
When you click a packet, Wireshark breaks it into layers matching the
OSI model:
| Layer | OSI Name | Wireshark Label | What You See |
|---|---|---|---|
| 7 | Application | HTTP, DNS, FTP... | Request/response data |
| 4 | Transport | TCP / UDP | Ports, sequence numbers, flags |
| 3 | Network | IPv4 / IPv6 | IP addresses, TTL |
| 2 | Data Link | Ethernet II | MAC addresses |
| 1 | Physical | Frame | Frame length, timestamps |
This is why knowing the OSI model isn't just theory — it's how you
navigate Wireshark.
### Display Filters vs Capture Filters
| Type | When Applied | Purpose | Example |
|---|---|---|---|
| **Capture Filter** | Before/during capture | Limits what gets recorded
| `port 80` |
| **Display Filter** | After capture, on existing PCAP | Limits what you
see | `http` or `ip.addr==10.0.0.1` |
For most SOC analysis work (working from provided PCAPs), you'll use
**display filters**.
### Colour Coding
Wireshark colour-codes packets by protocol and state to help you spot
things fast:
- **Green** — HTTP traffic
- **Light blue** — UDP
- **Dark blue** — DNS
- **Black with red text** — TCP issues (resets, errors)
- **Yellow** — Windows-specific traffic (SMB, etc.)
You can create custom colour rules to highlight traffic of interest.
---
## Tasks Walkthrough
### Task 1 — Introduction
This task establishes that the room uses two files provided in the VM:
- `http1.pcapng` — used to follow along with the screenshots in the room
text
- `Exercise.pcapng` — used to answer all questions
**Key takeaway:** Always check which file a task expects. Mixing them up
is a common mistake.
---
### Task 2 — Tool Overview
**What it covers:** The GUI layout, the five main panes, how to open
files, and colouring rules.
**Steps I took:**
1. Opened Wireshark from the VM desktop
2. Used `File ! Open` to load `http1.pcapng`
3. Familiarised myself with Packet List, Packet Details, and Packet
Bytes panes
4. Explored `View ! Colouring Rules` to see default colour assignments
**Key action — Capture File Properties:**
`Statistics ! Capture File Properties` gives you a summary of the PCAP
including:
- File hash (SHA256, MD5) — useful for file integrity verification
- Capture duration
- Total packet count
- File comments — *these can contain hidden flags in CTF-style rooms!*
---
### Task 3 — Packet Dissection
**What it covers:** Using the Packet Details pane to explore OSI layers
within a packet.
**Steps I took:**
1. Loaded `Exercise.pcapng`
2. Navigated to packet 38 (used `Go ! Go to Packet...`, typed `38`)
3. In the Packet Details pane, expanded each layer in turn
4. Reached the **Application Layer ! HTTP** section
5. Found the markup language used: **XML (eXtensible Markup Language)**
**Lesson learned:** The OSI model becomes very concrete when you can
expand each layer in Wireshark and see exactly what sits at each level.
Layer 7 had the HTTP request; inside that was the XML payload. Layer 3
had the IP source and destination. Layer 4 had the TCP segment length.
Everything from theory class — now visible.
---
### Task 4 — Packet Navigation
**What it covers:** Finding specific packets efficiently using Find, Go
To, Bookmarks, and Packet Comments.
**Techniques covered:**
| Feature | How to Access | Use Case |
|---|---|---|
| Find Packet | `Edit ! Find Packet` (Ctrl+F) | Search by display
filter, hex, or string |
| Go To Packet | `Go ! Go to Packet` (Ctrl+G) | Jump to a known packet
number |
| Mark Packet | Right-click ! Mark/Unmark | Bookmark packets to revisit
|
| Packet Comments | Right-click ! Packet Comment | View analyst notes
embedded in the PCAP |
**Steps I took for the string search question:**
1. Opened `Edit ! Find Packet`
2. Changed search type to **String**
3. Changed search scope to **Packet Details**
4. Searched for `r4w`
5. Found the packet and expanded the relevant layer to find the artist
name
**Why this matters operationally:** In a real incident, you might
receive a PCAP with thousands of packets. Knowing how to search for a
specific string (e.g., a malicious user-agent, a known C2 domain, a
password submitted over HTTP) is the difference between a 2-minute
investigation and a 2-hour one.
---
### Task 5 — Packet Filtering
**What it covers:** Applying display filters, following streams, and
exporting objects.
**Techniques covered:**
**Display Filter syntax examples:**
```
http # Show only HTTP packets
ip.addr == 10.0.0.1 # Traffic involving this IP
ip.src == 10.0.0.1 # Traffic FROM this IP
tcp.port == 80 # TCP traffic on port 80
http and ip.src == 10.0.0.1 # Combine filters
!(arp) # Exclude ARP
```
**Right-click shortcut:** Right-clicking any field in the Packet Details
pane and choosing **Apply as Filter** will auto-build the display filter
for you. This is the fastest way to build filters when you're not sure
of the syntax.
**Following streams:**
`Right-click a packet ! Follow ! TCP Stream` reconstructs the full TCP
conversation in human-readable form. This is invaluable for reading HTTP
requests/responses, seeing credentials passed in cleartext, or
understanding what a session contained.
**Exporting objects (files from the PCAP):**
`File ! Export Objects ! HTTP` lists all files transferred over HTTP in
the capture. You can filter by extension, then export and inspect them.
This is how analysts recover malware, documents, or images transferred
during an incident.
**Steps for the .txt file extraction question:**
1. `File ! Export Objects ! HTTP`
2. Filtered for `.txt` in the export dialog
3. Saved the file
4. Opened it to find the answer
---
## Key Takeaways
1. **Wireshark is a viewer, not a detector** — it shows you everything;
YOU are the detection engine. The tool is only as good as the analyst
using it.
2. **The OSI model is not just exam theory** — every layer you studied
maps directly to a section in the Wireshark Packet Details pane.
Understanding the model makes navigation intuitive.
3. **Display filters are essential** — a real-world PCAP can have
hundreds of thousands of packets. Filtering to only HTTP, or only
traffic from a suspicious IP, is the first step in any investigation.
4. **Export Objects is a hidden gem** — files transferred in cleartext
over HTTP (and other protocols) can be extracted directly from the PCAP.
This is used in both incident response and malware analysis.
5. **Statistics ! Capture File Properties** should always be your first
stop — it gives you the big picture before you dive into individual
packets.
---
## Where Would I See This in a SOC?
- **Phishing investigation:** Analyst receives a PCAP from an endpoint.
Uses display filters to isolate HTTP/HTTPS, follows TCP streams to see
what the user downloaded, exports objects to recover the suspicious
file.
- **Malware triage:** Uses `Statistics ! Protocol Hierarchy` to identify
unusual protocols or high volumes of DNS traffic (potential DNS
tunnelling).
- **Credential exposure:** Filters for `ftp` or `http.request.method ==
POST` to identify cleartext credentials submitted over unencrypted
protocols.
- **Lateral movement detection:** Filters for SMB or unusual internal
traffic patterns to trace attacker movement inside a network.
---
## Confidence Self-Assessment
| Skill | Before Room | After Room |
|---|---|---|
| Navigating Wireshark UI | 2/5 | 4/5 |
| Packet dissection by OSI layer | 2/5 | 4/5 |
| Writing display filters | 1/5 | 3/5 |
| Exporting objects from PCAPs | 1/5 | 4/5 |
---
## What's Next
- **Wireshark: Packet Operations** — deeper filtering, statistics, and
stream analysis
- **Wireshark: Traffic Analysis** — real-world incident scenarios
- **NetworkMiner** — another PCAP tool with a different workflow worth
knowing
---
## References
- [Wireshark Official Documentation](https://www.wireshark.org/docs/)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/
dfref/)
- [TryHackMe Room](https://tryhackme.com/room/wiresharkthebasics)
- [Wireshark Sample Captures](https://wiki.wireshark.org/SampleCaptures)
— great for additional practice
---
## LinkedIn Post (copy-ready)
```
Room completed: Wireshark: The Basics | TryHackMe Cybersecurity 101
Wireshark is one of those tools that every security analyst needs to
know.
This room was a solid introduction to reading packet captures — the kind
of
skill that shows up constantly in SOC work, incident response, and
network forensics.
Key things I took away:
• Wireshark isn't a detection tool — the analyst is. It shows
everything;
you have to know what to look for.
• The OSI model becomes *tangible* when you can expand each layer in the
Packet Details pane.
• Display filters are what make large PCAPs workable — learning the
syntax
is non-negotiable.
• You can extract files (malware samples, documents) directly from a
PCAP
using File ! Export Objects. That one surprised me.
Full write-up on GitHub ! [YOUR LINK HERE]
#TryHackMe #CyberSecurity101 #Wireshark #NetworkSecurity #SOC #BlueTeam
#PacketAnalysis #LearningInPublic #CyberSecurityStudent
```
---
*Write-up by [Your Name] | TryHackMe: [Your Username] | GitHub: [Your
Profile]*
