# Bypassing-CG-NAT-for-Secure-File-Access
This is basically a self-hosted VPN on your home server for acessing files anywhere from any device without the risk of/greatly reduce risk of your private files just being blatantly sent over the internet without encryption

Project Overview
​The goal was to create a secure, high-performance link between my mobile devices(laptops, phones etc.) and my home infrastructure (Debian Server and Windows Workstation).

​The Problem: CG-NAT
​Most modern ISPs use Carrier-Grade NAT (CG-NAT). This meant my router did not have a public-facing IPv4 address, making traditional Port Forwarding and standard VPNs (OpenVPN/WireGuard) impossible to implement.
However, not all hope is lost due to some homes having their own public ip and not under bulk like otheres due to the ipv4 address exhaustion

​🛠️ The Tech Stack
​Host: Debian (Linux)
​Containerization: Docker
​Networking/Zero-Trust: Twingate
​Protocols: SMB v3, SSH, SFTP
​Hardware: Windows 11 PC, Debian Server, Android (File Commander/Termius)

​🏗️ Phase 1: The Gateway (Debian & Docker)
​To bypass the ISP's limitations, I deployed a Twingate Connector. Unlike a VPN, Twingate uses an outbound-only tunnel, which doesn't require a public IP or open ports.

​# Implementation

sudo apt update
sudo apt install docker.io
sudo systemctl enable --now docker

Youre gonna need docker or some sort of containerization tool to be able to manage hardware resources and once you have it installed create your connector within

Deploy Twingate Connector:
Docker container keeps the host OS clean and ensures the connector restarts automatically if the server reboots.

 Note: Tokens generated from Twingate Admin Console
docker run -d --name "twingate-connector" \
--restart=always \
-e TWINGATE_NETWORK="your-network-name" \
-e TWINGATE_ACCESS_TOKEN="your-token" \
-e TWINGATE_REFRESH_TOKEN="your-refresh-token" \
twingate/connector:latest

Phase 2: Integrating Windows Storage (SMB)
​The goal was to map my Windows Desktop folder (work folder) to my phone and work computer over the Twingate tunnel.

​Reasoning for Protocol Selection
​SFTP: Used for the Debian server.
​SMB v3: Selected for the Windows PC because it is the native protocol for Windows file sharing and offers better encryption/performance than v1 or v2.

​# Challenges & Solutions (The "Mistakes" Log)
​1. The Authentication Loop (Windows PIN)
​Error: Credentials were correct, but the connection was rejected.
​Discovery: Windows "Hello" (PIN login) changes how the OS handles passwords. If a PIN is active, the standard password is often "locked" for network requests.
​Solution: Disabled the Windows PIN requirement and reverted to standard password authentication.

​2. The FireWall & Profile Trap
​Error: Connection timed out.
​Mistake: The Windows network profile was set to "Public." Windows automatically blocks Port 445 (SMB) on Public networks, even if sharing is turned on.
​Solution: Changed the Network Profile to Private and manually enabled the "File and Printer Sharing (SMB-In)" rule in Advanced Firewall.

​3. Admin Token Filtering (just a precaution)
​Error: "Access Denied" despite using an Admin account.
​Technical Reason: Windows restricts administrative accounts from accessing shares over a network to prevent lateral movement by attackers.
​Solution: Created a dedicated local user (sharesuser) specifically for the network share. This avoids the security overhead of a Microsoft Account or Admin Profile.

​🚀 Final Configuration Logic
​Twingate Client (Phone/Laptop) connects to the Twingate Cloud.
​The Docker Connector (Debian) maintains a tunnel to the same Cloud.
​The request for <IP Here> (Windows PC) is routed through the Debian Connector.
​The Windows PC accepts the SMB v3 request because it sees the connection as Private and the user as a Local Share User.

​🔧 Useful Commands Used
​Check Network Profile: Get-NetConnectionProfile (PowerShell)
​Check Local IP: ipconfig (Windows) / ip a (Debian)
​Test Connectivity: ping and ssh -v (to debug handshake issues)

# Personal Notes
TwinGate is relatively easy to set up although this isnt a VPN in the sense that you can change your location because that literally requires you to have a physical server there
it achieves the basic requirements of encryption and file transfer without the limits of having speed and reliability stuck behind a paywall
