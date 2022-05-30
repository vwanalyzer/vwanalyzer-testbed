# VWAnalyzer Testbed

This repository provides a way to monitor VoWiFi packets exchanged between UE and ePDG. Also, we provide some scripts to extract keys from UE and decrypt messages with them.

## WiFi AP Setting
**0. Environment**
  - OS: Ubuntu 20.04
  - UE: Samsung Galaxy S6 G920T (Should be rooted)
  - Devices: UE, one laptop (as WiFi AP), and one desktop (host)

**1. Install the WiFi AP application (laptop)**
  - `sudo apt-get update && sudo apt-get install snapd`
  - `sudo snap install wifi-ap`
  - `sudo wifi-ap.config set wifi.interface=<wifi interface name>` (e.g., wlan0)
  - `sudo wifi-ap.config set wifi.ssid=VoWiFi`
  - `sudo wifi-ap.config set wifi.security-passphrase=<password>`
  - `sudo wifi-ap.config set wifi.address=10.0.60.1`
  - `sudo wifi-ap.config set dhcp.range-start=10.0.60.2`
  - `sudo wifi-ap.config set dhcp.range-stop=10.0.60.254`
  - `sudo wifi-ap.config set disabled=false`
  - `sudo wifi-ap.status restart-ap`

**2. Run wireshark to capture VoWiFi-related packets (laptop)**
  - `sudo apt-get update && sudo apt-get install wireshark`
  - `sudo wireshark`
  - Capture packets from <wifi interface> (e.g., wlan0)
  - Filter packets by `isakmp || esp || dns`

## Extracting keys from UE (only works for Samsung Galaxy S6 G920T)
**1. Root UE**

**2. Download Frida-tool (host)**
  - `pip install frida-tools`
  
**3. Install Frida-server at Samsung Galaxy S6 (host to UE)**
  - Download frida-server android arm64 from https://github.com/frida/frida/releases
  - Attach UE to Host via USB
  - Extract the downloaded file by `unxz frida-server.xz` and copy it to the sdcard of UE
  - `adb shell`
  - `su`
  - `cp /sdcard/frida-server /data/local/tmp`
  - `chmod 777 frida-server`
  - `./frida-server &`

**4. Preparation for extracting keys (host)**
  - Run `frida-trace -i aes_v8_set_encrypt_key -i HMAC_Init_ex -U eris`
  - You can find the subdirectory `__handlers__` after quitting the application
  - Copy the two javascript files (aes_v8_set_encrypt_key.js and HMAC_Init_ex.js) in handlers/libcrypto.so to __handlers__/libcrypto.so
  
**5. Capture VoWiFi-related packets**
  - [Host] Run `frida-trace -i aes_v8_set_encrypt_key -i HMAC_Init_ex -U eris > <key file>`
  - [Laptop] Run wireshark at WiFi AP
  - [UE] Turn on the WiFi interface
  
**6. Extract keys**
  - Use the translate.py script in script to get the keys `python3 translate.py -k <key file>`

**7. Decrypt messages**
  - In Wireshark, click [Edit] - [Preferences]
  - Dropdown [Protocols] and find [ISAKMP]
  - Click [Edit...] next to IKEv2 Decryption Table
  - Insert Initiator's SPI and Responder's SPI (You can find them from the captured packets) and extracted keys. (Select AES-CBC-256 for the encryption algorithm and ANY 96-bits of Authentication [No Checking] for the integrity algorithm)
  - You can also decrypt ESP packets similarly
