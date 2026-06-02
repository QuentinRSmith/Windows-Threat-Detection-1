# Windows-Threat-Detection-1
Follow me through the process of going through the Windows Threat Detection 1 TryHackMe room!

# Introduction

In this room, we will:

- Explore how threat actors access and breach Windows machines
- Learn common Initial Access techniques via real-world examples
- Practice detecting every technique using Windows event logs


# Initial Access

Initial access is the stage in a cyberattack where a threat actor gains their first foothold in a system or network—often through phishing, exploiting vulnerabilities, or using stolen credentials—establishing the entry point for all later malicious activity.

Exposing a Windows server directly to the Internet is common—websites need open HTTP ports, mail servers need SMTP, and admins often rely on RDP for remote management. But every exposed service increases risk. Automated bots scan new hosts within minutes, probing for open ports, weak credentials, and unpatched vulnerabilities. If anything is misconfigured, attackers will exploit it, as reflected in MITRE techniques like T1133 (External Remote Services) for abusing exposed RDP/SSH and T1190 (Exploit Public‑Facing Applications) for targeting vulnerable or misconfigured web services.

A laptop doesn’t need to be Internet‑exposed to get infected—users often infect themselves. Clicking on malicious links, opening phishing attachments, using pirated software, or plugging in unknown USB devices all create easy entry points for attackers. Since human error remains the weakest link, user‑driven Windows initial‑access alerts are common. Key MITRE techniques include T1566 (Phishing), where users are tricked into running malware, and T1091 (Removable Media), where infected USB devices spread malware across systems.

# Initial Access via RDP

Now, in our VM scenario, the IT admin exposed RDP on a production server so that it could be accessed from home on weekends. The credentials were set to Administrator:Summer2025. Let's reconstruct what happened next, just in a few hours, and try to detect it in logs by using Event Viewer (C:\Users\Desktop\Administrator\Practice\RDP Case\RDP-Security.evtx file):

<img width="485" height="433" alt="image" src="https://github.com/user-attachments/assets/0c057483-285f-4f1b-a3a8-7444000ea600" />



We can see that the ADMINISTRATOR user was actively brute-forced by botnets

<img width="480" height="359" alt="image" src="https://github.com/user-attachments/assets/45b1af14-5bd7-4c99-879d-07f0a960878f" />


The IP address 203.205.34.107 managed to breach the host via RDP

<img width="476" height="326" alt="image" src="https://github.com/user-attachments/assets/bb349c35-80e4-4a4b-86ae-7a8cc5e7bb69" />


The Workstation Name (hostname) of the threat actor was DESKTOP-QNBC4UU

<img width="485" height="353" alt="image" src="https://github.com/user-attachments/assets/16131b65-a9a0-4530-bb3d-75539ed157c3" />


# Initial Access via Phishing

Phishing remains a major threat because it can’t be mitigated as easily as blocking remote access. If users can reach the Internet, they will eventually encounter—and sometimes execute—malware that bypasses perimeter defenses. Phishing attacks have surged dramatically in recent years, and their success rate remains high. This task focuses on two Windows breach vectors: malicious binaries and LNK attachments, both tied to MITRE techniques like T1566 (Phishing).

Windows supports many executable file types, and users often recognize risky .exe files but overlook others like .com, .scr, or .cpl, all of which can carry malware. Attackers exploit this by giving malicious files harmless‑looking names—especially since Windows hides file extensions by default. A file like invoice.pdf.exe may appear simply as invoice, and with a matching icon, users are far more likely to open it.

To evade antivirus detection, attackers often use PowerShell, VBScript, or BAT files instead of binaries. A common trick is hiding these scripts behind LNK shortcuts, which look like normal Windows shortcuts but execute malicious commands when opened.

In this task, we will investigate three phishing attachment examples stored in:
C:\Users\Administrator\Desktop\Practice\Phishing Case 1-3

Playing the role of the untrained user and mindlessly opening the COM file. Running the www.skype.com file from the Phishing Case 1 folder, in which we get the flag: THM{misleading_extension}

<img width="490" height="137" alt="image" src="https://github.com/user-attachments/assets/16db026d-0d9a-4515-975e-f0ace5e51d28" />



Next, with the second attachment from the Phishing Case 2 folder. The malicious LNK downloads from the URL: http://wp16.hqywlqpa.thm:8000/cgi-bin/f


<img width="310" height="434" alt="image" src="https://github.com/user-attachments/assets/a76afc36-f761-4452-9978-8227aa42205e" />



Finally, moving on to the Phishing Case 3 folder and reviewing its content. The name of the double-extension file we see is: best-cat.jpg.exe

<img width="486" height="168" alt="image" src="https://github.com/user-attachments/assets/b2a10df4-9eb1-4b45-801b-225eb00d5c3c" />


# Phishing Continued

Hunting malicious downloads is straightforward once you understand how users encounter them. A phishing email or application typically leads to a direct malware download—or more commonly, a ZIP/RAR archive containing it. Sysmon is especially useful here, as it can reveal each stage of the attack chain, from opening the attachment to extracting and executing the payload.

LNK files have a unique advantage for attackers: they leave very little execution trace. When a user opens a malicious shortcut disguised as a legitimate app, Windows Explorer simply follows the LNK’s target path, making it appear as though a trusted process launched the payload. The reliable way to confirm LNK‑based phishing is to look for the file‑creation events that preceded execution—malicious shortcuts must first appear in locations like the Downloads folder before they are run.

In this task, we will investigate the third phishing case by checking the attached Sysmon logs:
C:\Users\Administrator\Desktop\Practice\Phishing Case 3\Phishing-Sysmon.evtx

<img width="486" height="428" alt="image" src="https://github.com/user-attachments/assets/7ee75660-cafe-4395-894d-e90242940b4a" />



We see that the file that the user downloads via the web browser is: C:\Users\Administrator\Downloads\top-cats.zip

<img width="481" height="336" alt="image" src="https://github.com/user-attachments/assets/a46a0e42-ecdb-4fc7-8235-922f944857e2" />



The user unarchived the suspicious file in: C:\Users\Administrator\Pictures

<img width="487" height="371" alt="image" src="https://github.com/user-attachments/assets/55fc3868-88a0-48cb-9309-b9584ec14a65" />



The process ID of the launched phishing malware is: 5484

<img width="484" height="330" alt="image" src="https://github.com/user-attachments/assets/990b337b-e8b4-494e-a88e-ded27c0bf5b7" />



Finally, the malicious domain the malware tries to connect to is rjj.store

<img width="475" height="331" alt="image" src="https://github.com/user-attachments/assets/449041d2-fe53-4b65-9faa-4d30888da56c" />



# Initial Access via USB

Initial Access via an infected USB drive bypasses firewalls, much like phishing, and can start the attack chain even without Internet access, continuing to spread without user interaction.

While USB malware can use advanced techniques to auto‑execute, most infections still happen because users manually run malicious files. Common tricks include hiding legitimate USB contents and replacing them with a fake RECOVERY.lnk, adding binaries like Photos.exe disguised as folders, or creating double‑extension files such as photo_2024_1_12.jpg.exe. Detecting USB‑based initial access often resembles phishing cases, since both rely on users launching malware through Explorer. In some investigations, you may find clear evidence of execution directly from an external drive path like E:\malware.exe.

For this task, we will investigate a typical attack chain via USB using the attached Sysmon logs:
C:\Users\Administrator\Desktop\Practice\USB Case\USB-Sysmon.evtx

<img width="484" height="434" alt="image" src="https://github.com/user-attachments/assets/b1e084db-a57f-4198-a38c-1051a65be086" />



We see the USB file was launched by the user is: E:\Open Sandisk 4GB USB.exe

<img width="434" height="304" alt="image" src="https://github.com/user-attachments/assets/836db67d-52d1-4c5a-bc53-e57a8dfc74f6" />


The suspicious file that the malware dropped to the disk is: C:\Users\Public\Documents\winupdate.exe 

<img width="482" height="371" alt="image" src="https://github.com/user-attachments/assets/28fb2e2e-15f4-469b-a41b-ccda2bde8357" />


Lastly, the other USB that the malware propagates was: F:\Open Data Traveler 32 GB USB.exe

<img width="430" height="295" alt="image" src="https://github.com/user-attachments/assets/cd4fae4e-000f-425f-ad8c-b74be1025d76" />


# Conclusion

Understanding Windows Initial Access comes down to recognizing the patterns behind how attackers first enter an environment.
Some Key Takeaways:  
• The most common Windows Initial Access vectors are exposed services and user‑driven attacks.
• RDP‑based access is easy to spot using default authentication logs (4624/4625).
• User‑driven attacks are best detected through process‑execution telemetry, especially Sysmon.
• Each Initial Access method—such as LNK‑based attacks—has distinct indicators you learn to recognize through practice.
