# Network-Traffic-Triage-Lab-Wireshark-Capture-CSV-Export-and-Explainable-Alert-Investigation-
An offline, browser based tool that turns a Wireshark CSV export into an explainable traffic triage report. It flags activity worth investigating, explains why, and never claims anything is malicious. Built for learning SOC analyst triage, with all analysis done locally in the browser.
# Network Traffic Triage Lab (Wireshark Capture, CSV Export and Explainable Alert Investigation in an Offline Browser Tool)

`Wireshark` · `Traffic Triage` · `HTML/CSS/JavaScript` · `SOC Analyst Workflow` · `Packet Analysis` · `AI Assisted Development`

## Overview
This project was about learning how a SOC analyst or network defender looks at an unfamiliar packet capture. I captured live traffic on my own Windows laptop with Wireshark, exported the packet list as a CSV, and ran it through a **Network Traffic Triage Tool** that I designed and had built as a fully offline, browser based page. The tool reads the CSV, applies seven simple detection rules, and produces a report that explains what stood out, why it stood out, and what to check next.

The rule I set for the whole project was that **an anomaly is a starting point for investigation, not proof of compromise.** Nothing in the tool says unusual traffic is malicious. Every alert lists innocent explanations next to the reasons it deserves a look, and the investigation score is just a sum of fixed rule weights that is shown line by line, not a "97% malicious" style number.

I ran the tool against my own real capture (679 packets over about 32 seconds), got two alerts, and then spent time working out whether those alerts actually meant anything. That part taught me more than building the tool did.

## Objective
Practice the triage mindset: observe something unusual, identify the source and destination, work out the protocol, decide whether the activity is expected, and decide whether it needs more investigation. Along the way, build a tool that makes that process visible to a beginner, keeps all packet data on the local machine, and is honest about what it cannot tell you.

## Environment
- **Capture machine:** my own Windows laptop on home WiFi, capturing on the WiFi interface
- **Capture tool:** Wireshark 4.6.x, 679 packets captured, 0 dropped
- **Browser:** Microsoft Edge, opening the tool straight from disk (`file://`), no server needed
- **Build approach:** a plain English prompt written in Notepad, handed to an AI coding agent (Cline) running in my editor
- **Project folder:** `network-tool` in my Documents folder
- **Data used:** one real capture exported as `test.csv` (94.5 KB), plus three fictional demo datasets built into the tool

## Tools I Used

| Tool | What It Does | Why I Used It |
|------|--------------|----------------|
| **Wireshark** | Packet capture and analysis | Captured live traffic and exported the packet list with File, Export Packet Dissections, As CSV |
| **Network Traffic Triage Tool** | My offline HTML, CSS and vanilla JavaScript analyser | Parsed the CSV in the browser and produced the dataset summary, overview dashboard and alerts |
| **AI coding agent (Cline)** | Writes and tests code from instructions | Built the tool from my written requirements, since I am not a programmer |
| **Command Prompt (`ping`)** | Basic connectivity test | Sent a few test pings to confirm Wireshark was ready and the network was responding |
| **Microsoft Edge** | Web browser | Ran the tool locally and let me check that nothing was being sent anywhere |
| **Notepad** | Text editor | Wrote and refined the project prompt before handing it over |

## What I Did

### Writing the Requirements First
1. Before any code existed, I wrote a long plain English prompt (about 17,000 characters) describing exactly what I wanted. The most important instructions were these:
   - The tool must never claim that unusual traffic automatically means malicious activity.
   - All analysis must happen locally in the browser, with no uploads, no AI provider, no analytics, no cookies and no telemetry.
   - Use only HTML, CSS and vanilla JavaScript, with no frameworks, no backend, no databases and no API keys.
   - Version 1 should only support Wireshark CSV exports, not raw `.pcap` or `.pcapng` files.
   - Tolerate different column names, so `Source`, `Source IP`, `src` and `ip.src` all count as the source column.
   - If a required column is missing, show a clear explanation instead of crashing.
   - Show this message in the app: "Your packet capture data is analysed locally in your browser and is not uploaded anywhere."
2. I made the decisions about what the tool should and should not do. The agent wrote the code. I want to be clear about that, because I did not type the JavaScript myself. What I can speak to is the design, the detection logic, and what the output means.

### How the Tool Works
The tool has seven rules, and every threshold is adjustable on the page:

| Rule | Default threshold |
|------|-------------------|
| High Packet Volume | more than 50 packets from one source |
| Possible Host Scanning | more than 10 unique destinations from one source |
| Possible Port Scanning | more than 10 unique destination ports on one destination |
| High ICMP Activity | more than 30 ICMP packets from one source |
| High DNS Query Activity | more than 30 DNS packets from one source |
| Repeated Source to Destination Communication | more than 40 packets between one pair of hosts |
| TCP SYN Heavy Activity | more than 20 pure SYN packets from one source |

Each alert shows the observed value, the threshold, why it could matter, possible benign explanations, and suggested investigation steps. The score is made of fixed weights (for example, High Packet Volume adds 20 points), and the report lists each contribution so nothing is hidden.

### Checking the Tool Before Trusting It
1. The agent ran an automated test page with 50 checks covering parser edge cases, column name variations, each detection rule, threshold behaviour, score maths, and hostile CSV cells such as script tags. The result was **50 out of 50 passing**.
2. It also confirmed the real page loads in a headless browser without JavaScript errors, and that the demo buttons and the privacy message appear.
3. The project includes a second, independent PowerShell script that checks the fictional demo datasets behave as labelled, without relying on the JavaScript at all.
4. I opened the page myself and confirmed the privacy banner, the upload area, the demo buttons and the adjustable thresholds were all there.

### Confirming Wireshark Was Ready
1. I opened Command Prompt and tried to ping `192.168.56.1`, but typed `ping192.168.56.1` with no space, and Windows correctly replied that it was not a recognised command. The same thing happened in the Nmap lab, and the fix is the same: read the error, spot the typo, retype.
2. The corrected `ping 192.168.56.1` returned replies with a TTL of 128, which is what I would expect from a Windows host.

### Capturing and Exporting Traffic
1. Started a capture on the WiFi interface, let it run for a short period, then stopped it. Wireshark showed **679 packets with 0 dropped**.
2. The capture was mostly TLS 1.2 application data, TCP acknowledgements and connection teardowns, some DNS queries and responses (including a "no such name" answer for `wpad.Home`), and a few broadcast frames.
3. Exported the packet list with **File, Export Packet Dissections, As CSV** and saved it as `test.csv` in the tool folder.

### Uploading and Parsing
1. Chose `test.csv` with the file picker. The tool reported: **679 packets loaded from 680 parsed rows, analysis complete with 2 alerts.**
2. The dataset summary showed the file name, size (94.5 KB), UTF-8 encoding, comma separator, 7 columns detected, and a recognised header row. Parse status was "Successful, with notes."
3. The 680 parsed rows against 679 packets lines up with Wireshark's own packet count if the header row is counted, though I did not verify that line by line.

### Reading the Triage Report
1. **Investigation score: 20 out of 100, labelled "Low activity."** Only one rule fired (High Packet Volume), and it flagged 2 source hosts.
2. **Network overview:** 679 total packets, 29 unique source hosts, 30 unique destination hosts, TCP as the most common protocol (257 packets), average packet length of 518.9 bytes, and a capture window of 32.1 seconds.
3. **Alert 1, an IPv6 address (`2a02:c7c:...:fabd`):** 163 packets, 24% of the capture, 10 unique destinations. Top protocols were TCP (65), Other (56) and HTTPS/TLS (42).
4. **Alert 2, `192.168.0.84`:** 133 packets, 19.6% of the capture, 10 unique destinations. Top protocols were TCP (78), HTTPS/TLS (50) and Other (3). The most contacted destinations were `192.168.0.2` (33 packets), `192.168.0.8` (32 packets) and `20.89.1.13` (20 packets).
5. Both alerts were rated Medium and both listed the same benign explanations: backups and sync, system updates, streaming or large downloads, monitoring agents, scheduled jobs, and misconfiguration causing a retry loop.

### Working Out Whether the Alerts Meant Anything
1. The top source in the overview was the same IPv6 address that was also the most contacted destination (188 packets received, from 10 sources). A host that sends and receives the most traffic in a capture taken on its own machine is very likely the capturing laptop itself.
2. `192.168.0.84` was the client address visible throughout my Wireshark packet list, and the IPv6 address ends in the same interface identifier pattern as the local IPv6 addresses seen in the DNS traffic. My read is that **the two alerts are probably the same laptop, seen over IPv6 and IPv4.** Together they account for 296 of 679 packets, which is about 43.6% of the capture.
3. I have not confirmed this against my router's device list, so I am treating it as a strong guess and not a fact. The suggested investigation steps in the alert (identify the device behind the address, check whether the volume is normal, review the destinations) are exactly how I would settle it.
4. The conclusion for this capture: a laptop generating over 50 packets in 32 seconds with normal apps running is not surprising. The default threshold is a teaching example, and my own machine was the thing being flagged. This is the lesson of the project in miniature.

### Noticing Rough Edges in My Own Tool
While reading the output carefully, I found a few things that are not right yet:
1. **The "Demo Data" badge appears on my real upload.** The dataset summary for `test.csv` still shows the Demo Data label, which is wrong. A real capture should not be labelled as fictional.
2. **The severity text reads "Rated undefined."** The alert sentence is meant to explain why the severity was chosen, but a variable is not being filled in.
3. **IPv4 and IPv6 traffic from one device is counted as two hosts.** This is a known limitation the project already documents, but seeing it happen on my own data made it real.
I am listing these here instead of hiding them, because noticing flaws in a tool's output is part of the job.

## What's in This Repo

```
network-tool/
├── index.html                   # The application page
├── styles.css                   # Dark theme styling
├── core.js                      # Analysis engine: CSV parsing, 7 rules and scoring
├── script.js                    # Interface: upload, settings, filters and charts
├── samples.js                   # Embedded fictional demo datasets
├── tests.html + tests.js        # Offline self test page (50 checks)
├── README.md                    # This file
├── samples/
│   ├── normal-traffic.csv       # Expected result: 0 alerts, score 0
│   ├── high-volume-traffic.csv  # Expected result: 1 alert, score 20
│   └── mixed-soc-lab.csv        # Expected result: 9 alerts, score 90
├── tools/
│   ├── build-samples.ps1        # Generates the demo CSVs
│   └── verify-samples.ps1       # Independent PowerShell check of the datasets
└── screenshots/
    ├── 01-project-prompt.png
    ├── 02-tool-privacy-banner-and-upload.png
    ├── 03-detection-thresholds.png
    ├── 04-tests-passing.png
    ├── 05-wireshark-capture.png
    ├── 06-export-as-csv.png
    ├── 07-dataset-summary.png
    ├── 08-triage-report-score.png
    ├── 09-high-packet-volume-alerts.png
    └── 10-top-destinations.png
```

## Skills I Picked Up
- **Capturing and exporting traffic properly,** using Wireshark on an interface I own and exporting the packet list to CSV with a header row the tool can read.
- **Reading a triage report critically,** understanding that an alert, a severity and a score are all signals about where to look, not conclusions about what happened.
- **Spotting when a host is probably the capture machine,** by noticing that the top source and top destination were the same address.
- **Understanding why thresholds are not universal,** because a default of 50 packets flagged ordinary laptop traffic in a 32 second window.
- **Writing clear requirements for software I cannot write myself,** including privacy rules, scope limits and "do not claim X" rules that shaped the whole tool.
- **Checking work instead of assuming it is right,** from the automated tests down to noticing the wrong Demo Data badge and the "undefined" wording in my own output.

## How This Applies in the Real World
Triage is the daily reality of SOC work. Analysts do not get a clean answer from a tool, they get an alert, and the job is to decide quickly whether it needs attention. The seven rules here are simplified versions of things real detections look for: unusually chatty hosts, sweeps across many destinations, many ports on one target, heavy ICMP or DNS, repeated conversations and floods of SYN packets.

The most realistic part of this project is that my own alerts turned out to be explainable. A large share of real alerts are ordinary activity that crossed a threshold, which is why every alert here carries benign explanations and a next step, and why the central line of the project is that detection tells you where to look and investigation tells you what happened.

The privacy design also reflects real habits. Packet captures can contain sensitive information, so the tool processes everything in the browser, makes no network requests, and sets no cookies.

## Where I'm Coming From
I'm making the jump into cybersecurity from a background in **healthcare**. The mindset carries over more than I expected. In healthcare you learn that a single reading is not a diagnosis, you look at the whole picture, rule out the harmless explanations, and document what you find. Triage in a SOC is the same idea applied to networks. I'm currently studying for **CompTIA Security+** and building projects like this one to get real practice in.

I'll be upfront about two things. I am not a programmer, and the code in this project was written by an AI coding agent from requirements I wrote. And the tool has flaws I found only after running it on a real capture, which I have left in this write up instead of cleaning away.

## What I Want to Learn Next
- Fixing the three rough edges above, starting with the Demo Data badge and the "undefined" severity wording
- Merging IPv4 and IPv6 activity from the same device so one machine is not counted as two
- Rerunning my capture with different thresholds to see exactly which alerts appear and disappear
- Capturing a deliberate test of scanning behaviour on a lab network I own, so I can see the host scanning and port scanning rules fire on real traffic
- Learning to read `.pcapng` files directly in Wireshark, including following TCP streams, before thinking about parsing them in a tool
- Correlating a network alert with other telemetry, such as endpoint logs or a SIEM, to practise the last step of triage

## Limitations & What I'd Do Differently in Production
- **A CSV is a flattened view of a capture.** Payloads, TCP sequence analysis, retransmissions and full flow reconstruction are gone. Real analysis would go back to the original `.pcapng`.
- **Encrypted traffic limits what can be seen.** Most of my capture was TLS application data, which is unreadable and is not meant to be read here.
- **The thresholds are teaching examples.** A real environment needs thresholds tuned to its own baseline, and a 32 second capture cannot establish a baseline at all.
- **NAT and IPv6 can distort host counts.** I saw one version of this on my own data.
- **Rule based detection only.** It cannot say whether a host is compromised, and it does not replace Wireshark, an IDS or IPS, a SIEM, or a trained analyst.
- **I cannot independently audit the code.** I relied on the automated tests, the independent PowerShell checks and watching the browser, and a production tool would need review by someone who can read the source.
- **Only one real capture was analysed.** A proper assessment would compare several captures across different times and devices.
- **Real captures contain real addresses.** I would not publish a raw capture CSV or unredacted screenshots from my home network in a public repository.

## References
- [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [Wireshark: Export Packet Dissections](https://www.wireshark.org/docs/wsug_html_chunked/ChIOExportSection.html)
- [CompTIA Security+ (SY0-701) Exam Objectives](https://www.comptia.org/certifications/security)
- Network Traffic Triage Tool v1.0, the project in this repository
