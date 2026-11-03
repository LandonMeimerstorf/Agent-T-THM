# 🕵️ Agent T — TryHackMe Walkthrough
 
*TryHackMe · Author: John Hammond · Difficulty: `Easy`*
 
> **Room Summary:** Agent T is a beginner-level Linux challenge on [TryHackMe](https://tryhackme.com/room/agentt). The target is running a backdoored build of PHP (8.1.0-dev), and the whole room comes down to spotting it and abusing it for remote code execution. One flag, no privilege escalation needed.
 
---
 
## 🕵️ Reconnaissance
 
Deploy the box, give it a minute or two to boot, then run an initial scan:
 
```bash
nmap -sC -sV -oN agentt.txt <TARGET_IP>
```
 
<img width="670" height="172" alt="nmap scan results showing port 80 running PHP 8.1.0-dev" src="https://github.com/user-attachments/assets/aba21e6c-5876-4ff8-8d88-ed3830bbee37" />
Port 80 is open running a PHP development server. The nmap service line literally reads `PHP cli server 5.5 or later (PHP 8.1.0-dev)`, with an HTTP title of *Admin Dashboard*. In a browser it's a plain, harmless-looking page with nothing obviously interesting on it:
 
<img width="956" height="883" alt="the Admin Dashboard web page, looking innocent" src="https://github.com/user-attachments/assets/9763f4b9-64f7-4715-96e3-0c8e57159bd6" />
---
 
## 🧠 The Tell
 
The clue lives in the HTTP response headers, not the page itself. Pull them with a HEAD request:
 
```bash
curl -I http://<TARGET_IP>
```
 
<img width="507" height="152" alt="curl -I response showing X-Powered-By: PHP/8.1.0-dev" src="https://github.com/user-attachments/assets/c7bd5b26-e93e-475e-b2a5-5271f5d2f7f6" />
Look at the `X-Powered-By` header:
 
```http
X-Powered-By: PHP/8.1.0-dev
```
 
> 🧠 No production server runs a `-dev` build of PHP. And this exact version, **8.1.0-dev**, is the one that shipped with a backdoor. The server is basically announcing the vulnerability in its own response headers. That's the "something seems off" the room is hinting at.
 
---
 
## ⚙️ The Vulnerability
 
Two malicious commits were slipped into the PHP source repo on March 28, 2021, disguised as minor typo fixes and signed off under the names of trusted maintainers. The planted code checks every request for a header called **`User-Agentt`** (note the double `t`). If that header's value starts with the string `zerodium`, everything after it is passed to `zend_eval_string()` and executed as PHP on the server.
 
> 💡 Translation: send the right header, run any command you want.
 
---
 
## 💥 Exploitation
 
### Python exploit (Exploit-DB 49933)
 
flast101's script turns the backdoor into an interactive pseudo-shell:
 
```bash
searchsploit -m 49933
python3 49933.py
```
 
<img width="633" height="214" alt="running the python exploit script" src="https://github.com/user-attachments/assets/23828b25-98ac-414c-9a63-b18dc2e91fd3" />
It prompts for the host URL, then hands you a shell:
 
<img width="503" height="147" alt="pseudo-shell returned by the exploit" src="https://github.com/user-attachments/assets/7b87e13e-b8af-46a4-9d5f-f14cfb604ad3" />
```bash
$ id
$ ls /
$ cat /<flag_file>
```
 
> 🧩 There's no official Metasploit module for this one, so don't burn time hunting for it. The Python script (or a one-line `curl` carrying the `User-Agentt: zerodium...` header) is the intended path.
 
---
 
## 🏁 Flag
 
<details>
<summary>🚩 Click to reveal</summary>
<br>
```
flag{4127d0530abf16d6d23973e3df8dbecb}
```
 
</details>
---
 
## ✅ Takeaways
 
By completing this room, you practiced:
 
- Reading HTTP response headers to fingerprint a server. The `Server` and `X-Powered-By` values give the whole game away here.
- Recognizing a non-standard version string (a `-dev` build) as a red flag worth researching right away.
- Exploiting a real supply-chain backdoor, the PHP 8.1.0-dev `User-Agentt` RCE from March 2021.
---
 
*Walkthrough by [LandonMeimerstorf](https://github.com/LandonMeimerstorf)*
