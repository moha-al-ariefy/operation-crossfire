# Red Team After-Action Report (AAR)
**Operator:** Mohammed Al Saraify
**Campaign:** Operation Crossfire (Sprint 2)

## 1. Campaign Overview
This campaign was designed to simulate a targeted, multi-stage attack against the Sohail EdTech platform. The objective was to map the attack surface, gain initial access via authentication bypass, attempt evasion, and successfully exfiltrate the backend database. 

## 2. Execution Analysis: What Worked
* **Initial Access:** The standard in-band SQL injection (`admin@juice-sh.op'--`) on the login endpoint was highly effective, immediately granting an administrative JWT. 
* **Data Exfiltration:** The UNION-based SQL injection on the product search API successfully merged the `Users` table with the product catalog, allowing for mass data extraction directly through the frontend HTML rendering.

## 3. Operational Security: What Was Noisy
* **Reconnaissance Phase:** The initial DirBuster scan was incredibly loud. Generating thousands of `404 Not Found` requests in a span of minutes is a massive indicator of compromise (IoC) that any competent SOC will flag immediately. 
* **Payload Signatures:** Executing raw SQL operators (`UNION SELECT`) directly in the GET parameters creates a highly visible footprint in standard HTTP access logs.

## 4. Evasion & Future Improvements
During the evasion phase, I attempted to mask my exfiltration payloads by URL-encoding the SQL operators and spoofing my `User-Agent` as a Googlebot crawler. While this might bypass simple string-matching filters, it remains detectable through behavioral analysis. 

To ensure better stealth in future engagements and align with advanced operational security standards required for certifications like the OSCP or live bug bounty programs like HackerOne, I would implement the following:
1. **Low and Slow Recon:** Replace aggressive automated directory brute-forcing with passive reconnaissance and targeted, throttled manual probing to stay under rate-limit and alerting thresholds.
2. **Advanced Payload Obfuscation:** Utilize SQLmap's tamper scripts (e.g., `space2comment`, `charencode`) to heavily obfuscate injection attempts, making signature-based detection by web application firewalls (WAFs) significantly more difficult.
3. **Session Hijacking over SQLi:** Instead of repeatedly hitting the database with loud queries, I would prioritize stealing active session tokens via DOM-based Cross-Site Scripting (XSS), allowing me to blend in perfectly with legitimate user API traffic.
