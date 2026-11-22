Wireless Security Auditing & Penetration Testing (Lab)
Disclaimer: This repository is for educational purposes only. All testing was performed on networks I personally own or have explicit permission to test. Unauthorized network access is illegal.

🎯 Objective
The goal of this lab was to simulate a wireless network attack to understand the vulnerabilities of WPA2-PSK (Pre-Shared Key) encryption. The focus was on capturing the 4-way handshake and using offline dictionary attacks to crack the password.

🛠️ Tools Used
Operating System: Kali Linux (VirtualBox)

Hardware: Realtek RTL8192CU USB Wi-Fi Adapter (Supports Monitor Mode)

Software: aircrack-ng suite (airmon-ng, airodump-ng, aireplay-ng), hcxdumptool, hcxpcapngtool, hashcat.

📝 Methodology
1. Enabling Monitor Mode
First, I killed interfering network manager processes and placed the USB adapter into monitor mode to capture raw 802.11 frames.

bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
2. Network Reconnaissance
I scanned the airwaves to identify target networks and connected clients (STATIONs).

bash
sudo airodump-ng wlan0mon
Target identified: Target_Network (WPA2-PSK) on Channel X with an active client.

3. Capturing the 4-Way Handshake
To crack the password, I needed the 4-way handshake that occurs when a device connects to the router.
I focused the capture on the target's channel and BSSID:

bash
sudo airodump-ng -c [CHANNEL] --bssid [ROUTER_MAC] -w capture_file wlan0mon
To force the connected client to reconnect and generate a new handshake, I sent a small burst of Deauthentication packets:

bash
sudo aireplay-ng --deauth 3 -a [ROUTER_MAC] -c [CLIENT_MAC] wlan0mon
Result: The airodump-ng screen confirmed WPA handshake: [ROUTER_MAC] in the top right corner.

4. Converting the Capture for Hashcat
Using hcxtools, I converted the .cap file into a hash format that Hashcat can process (.hc22000).

bash
hcxpcapngtool -o handshake.hc22000 capture_file-01.cap
5. Offline Dictionary Attack
Finally, I used Hashcat to perform a dictionary attack using the rockyou.txt wordlist.

bash
hashcat -m 22000 handshake.hc22000 /usr/share/wordlists/rockyou.txt
🛡️ Key Takeaways & Mitigation
WPA2-PSK is vulnerable to offline attacks: Once the handshake is captured, the attacker can guess passwords indefinitely without the router knowing.

Deauth attacks are highly effective: Modern routers need Management Frame Protection (802.11w) enabled to stop attackers from kicking clients off.

Mitigation: Use long, complex passphrases (15+ characters). Upgrade to WPA3 where possible, and hide the SSID (though this is not a strong defense).


