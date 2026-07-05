# Week 6 Reflection — Operation Crossfire (Red Operator Role)

## 1. Weekly Summary
Operation Crossfire was a live Purple Team engagement (Sprint 2) targeting the simulated Sohail EdTech platform via a local OWASP Juice Shop container. My role was the Red Operator, tasked with executing a multi-stage attack campaign against the application while my teammate (Saif) actively monitored the environment as the Blue Lead.

The campaign followed a structured kill chain:
*   **Reconnaissance:** Automated directory enumeration to map API routes.
*   **Initial Access:** In-band SQL injection on the login endpoint (`admin@juice-sh.op'--`) to hijack the administrator account.
*   **Evasion:** Attempting to bypass WAF rules by spoofing a Googlebot User-Agent and URL-encoding the SQL payloads.
*   **Data Exfiltration:** A UNION-based SQL injection on the product search API to dump the backend `Users` table. 
All technical stages landed successfully, granting full platform compromise.

## 2. Technical Growth
Knowing a defender was actively watching fundamentally changed my offensive mindset. I could no longer focus solely on making an exploit work; I had to consider the artifact footprint I was leaving in the web server and database logs. 

My actions varied wildly in "noise" levels. The automated directory brute-forcing (DirBuster) was extremely loud, generating a massive spike of `404 Not Found` errors. Similarly, executing raw `UNION SELECT` statements created highly visible HTTP GET parameters. The evasion attempt taught me that while simple obfuscation (URL encoding and User-Agent spoofing) might bypass static signature-matching, it does not hide the underlying malicious behavior from a human analyst conducting retrospective hunting.

Mapping the campaign to the MITRE ATT&CK framework (utilizing T1595.001 for Scanning, T1190 for Public-Facing App Exploits, and T1020 for Exfiltration) completely changed my planning. It forced me to treat the engagement as a logical progression of tactical goals rather than a random assortment of bugs.

## 3. Red-vs-Blue Dynamic
Reviewing our shared timeline revealed exactly how effective active defense can be against standard payloads. Saif caught my data exfiltration attempt almost instantly (a 2-minute dwell time) by flagging the abnormal HTTP response lengths and the `UNION SELECT` keyword. He also detected the initial recon within 5 minutes due to the sheer volume of traffic.

My longest dwell time was 1 hour and 15 minutes during the evasion phase. Because I disguised the traffic as a legitimate search engine crawler and URL-encoded the payload, it bypassed his initial alerts and was only discovered later during post-incident hunting.

To stay hidden longer next time, I would entirely eliminate automated scanners. I would rely strictly on passive reconnaissance and manual, throttled probing. Furthermore, instead of repeatedly executing noisy database queries, I would prioritize stealing active JSON Web Tokens (JWTs) via DOM-based XSS, allowing me to blend in perfectly with legitimate, authenticated user API traffic.

## 4. Self-Assessment
**Rating: 9/10**
I rate this campaign highly on realism, ATT&CK mapping, and timeline discipline. The payloads represent authentic threats faced by modern Node.js/Express APIs, the MITRE mappings were precise, and the shared timeline was maintained rigorously to accurately measure dwell time.

The weakest part of my Offensive After-Action Report (AAR) was the depth of the evasion strategies. While I identified that automated tools are loud, I did not provide specific, advanced obfuscation examples (such as utilizing specific SQLmap tamper scripts or HTTP desync payloads) that the Blue Team could actually use to write better detection rules. 

## 5. Next-Week Direction
For the next phase of training, I want to focus heavily on **Advanced Evasion & WAF Bypass**. Since Saif proved how easily standard SQLi and XSS payloads are caught in access logs, I need to learn how to manipulate HTTP headers, utilize chunked encoding, and craft heavily obfuscated injection vectors to defeat active perimeter defenses.
