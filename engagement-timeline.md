# Operation Crossfire: Shared Engagement Timeline

**Target:** OWASP Juice Shop (Local Docker - `127.0.0.1:3000`)
**Red Operator:** Mohammed Al Saraify
**Blue Lead:** Saif Alblooshi
**Date:** 3 July 2026

| Timestamp (UTC+4) | Red: Technique | Red: Endpoint | Red: Payload / Action | Red: MITRE ATT&CK ID | Red: Intended Impact | Blue: Detection Timestamp | Blue: Data Source | Blue: Caught/Missed (How) | Blue: Response Action | Blue: Severity |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 2026-07-03 14:00:00 | Active Scanning | `127.0.0.1:3000` | Automated DirBuster/Gobuster directory brute-force | T1595.001 | Map API endpoints and hidden directories. | 2026-07-03 14:05:00 | Web Server Access Logs (Nginx/Express) | **Caught:** Massive spike in `404 Not Found` errors from a single IP. | Monitor IP; flag for potential escalation. | Low |
| 2026-07-03 14:30:00 | Exploit Public-Facing App (SQLi) | `/rest/user/login` | `admin@juice-sh.op'--` | T1190 | Initial Access: Bypass authentication to hijack the administrator account. | 2026-07-03 14:38:00 | Application Auth Logs | **Caught:** Detected SQL comment syntax (`'--`) in the email field resulting in a `200 OK` login success. | Revoke active admin sessions; force password reset. | Critical |
| 2026-07-03 15:15:00 | Obfuscated Files/Info (Evasion) | `/rest/products/search?q=` | User-Agent spoofing (`Googlebot/2.1`) + URL Encoded SQLi: `%27%20OR%201%3D1--` | T1027 / T1036 | Bypass basic WAF string matching and masquerade as legitimate crawler traffic. | 2026-07-03 16:30:00 *(Retrospective)* | WAF / Reverse Proxy Logs | **Missed Initially:** Bypassed standard signature rules due to URL encoding and trusted User-Agent spoofing. Found during post-incident hunting. | Implement strict schema validation and anomaly detection for crawler IPs. | Medium |
| 2026-07-03 16:00:00 | Automated Exfiltration (UNION SQLi) | `/rest/products/search?q=` | `')) UNION SELECT id, email, password, null, null, null, null, null, null FROM Users--` | T1020 | Dump backend user hashes and PII to the frontend UI. | 2026-07-03 16:02:00 | Database Query Logs / HTTP Response Length | **Caught:** Identified `UNION SELECT` keyword in HTTP GET parameters and abnormally large response payload size. | Blacklist attacker IP; isolate database tier; initiate incident response. | Critical |
