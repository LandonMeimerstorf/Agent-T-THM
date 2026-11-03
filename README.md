<div align="center">
🕵️ Agent T
TryHackMe · Author: John Hammond · Difficulty: Easy

Exploiting the backdoored PHP 8.1.0-dev build for remote code execution.

</div>
About the Box

Agent T is a beginner-level Linux challenge built around a single, well-known flaw: the target is running PHP 8.1.0-dev, a development build that shipped with a hidden backdoor back in March 2021. The web page looks completely innocent. The giveaway is in how the server responds, not in anything it shows you. Spot the version, abuse the backdoor, grab the flag. One flag, no privilege escalation needed.

Recon

Deploy the box, give it a minute or two to boot, then run an initial scan:
<img width="956" height="883" alt="image" src="https://github.com/user-attachments/assets/9763f4b9-64f7-4715-96e3-0c8e57159bd6" />

bash
nmap -sC -sV -oN agentt.txt <TARGET_IP>

<img width="670" height="172" alt="image" src="https://github.com/user-attachments/assets/aba21e6c-5876-4ff8-8d88-ed3830bbee37" />

Port 80 is open running Apache. The site itself is a plain default-looking page with nothing obviously interesting on it.

The Tell

The clue lives in the HTTP response headers. Pull them with a HEAD request:

curl -I http://<TARGET_IP>

<img width="507" height="152" alt="image" src="https://github.com/user-attachments/assets/c7bd5b26-e93e-475e-b2a5-5271f5d2f7f6" />

Look at the X-Powered-By header:

X-Powered-By: PHP/8.1.0-dev

[!IMPORTANT] No production server runs a -dev build of PHP. And this exact version, 8.1.0-dev, is the one that was published with a backdoor. The server is basically announcing the vulnerability in its own response headers. That's the "something seems off" the room is hinting at.

The Vulnerability

Two malicious commits were slipped into the PHP source repo on March 28, 2021, disguised as minor typo fixes and signed off under the names of trusted maintainers. The planted code checks every request for a header called User-Agentt (note the double t). If that header's value starts with the string zerodium, everything after it is passed to zend_eval_string() and executed as PHP on the server.

Translation: send the right header, run any command you want.

Exploitation

Python exploit (Exploit-DB 49933)

flast101's script turns the backdoor into an interactive pseudo-shell:

bash
searchsploit -m 49933
python3 49933.py

<img width="633" height="214" alt="image" src="https://github.com/user-attachments/assets/23828b25-98ac-414c-9a63-b18dc2e91fd3" />

It prompts for the host URL, then hands you a shell:

<img width="503" height="147" alt="image" src="https://github.com/user-attachments/assets/7b87e13e-b8af-46a4-9d5f-f14cfb604ad3" />

$ id
$ ls /
$ cat /<flag_file>

[!TIP] There's no official Metasploit module for this one, so don't burn time hunting for it. The curl one-liner or the Python script is the intended path.

Flag
<details> <summary>🚩 Click to reveal</summary> <br>
flag{4127d0530abf16d6d23973e3df8dbecb}
</details>
Takeaways
Always check response headers, not just the rendered page. Server and X-Powered-By fingerprints give away the whole game here.
A -dev or otherwise non-standard version string is a red flag worth researching immediately.
Supply-chain attacks are real. A backdoor in an upstream build compromises every server that runs it.
<div align="center">

Walkthrough by <a href="https://github.com/LandonMeimerstorf">LandonMeimerstorf</a>

</div>
