---
title: "100 Days of Code: day_4-19+"
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - career
  - learning
  - 100DaysOfCode

---
D4: Elixir and Erlang install and intro
D5: Toying with `nmap`, pwnd my first Hack The Box(HTB) puzzle, using nmap and connecting via open port 23, used by unencrypted telnet protocol
D6: learn how to probe open ports to determine service/version & OS info with `nmap -sV 10.129.0.9`
> Mac does not have decent cli tool for accessing ftp(or I couldn't find one quickly). you can connect to FTP via Finder->Go->Connect to Server, but it's just UI. Other option to use `ncftp` that loggs use in as `anonymous` by default

- hacking SMB on p445
> Again Mac does not have decent alternative to Linux `smbclient`, seems like one of the biggies for MacOS [https://discussions.apple.com/thread/6408765](https://discussions.apple.com/thread/6408765) 

D7:
- DNS resolvers -> `scutil --dns`
- classic lsof `lsof -i :9253 -nP -sTCP:LISTEN`
- `nslookup google.com` - this is simple lookup, but if you need custom ports/etc, you need to go into interactive mode by just `nslookup` and setting your parms, refer `man nslookup`
- Mac sys logs located here (i.e. Puma server logs): `tail -f ~/Library/Logs/puma-dev.log`

TCP HTB: pwning Redis
`nmap` timing can be boosted
`-T<0-5>: Set timing template (higher is faster)`
I believe this is similar to
`-T paranoid|sneaky|polite|normal|aggressive|insane`
fastest way was to add couple of more flags, like scan TCP SYN request `-sS`
`nmap -v -sV -sS -p T:1009-9999 -T5 0.0.0.0`
then connect to Redis `redis-cli`
list keys `keys *`
`mget 'yrkey'`

D8:

day8 with setting up my new flipper zero device RogueMaster firmware. Esp32 setup coming soon, need windows to run. Exe scripts. 

D9:

day 9 involved a successful hardware hacking of NFC and sub-GHz spectrum. The feeling of joy is priceless, when the script just works as intended after days of research.

D9+,10-11:
day 9-11 involved hardware hacking and RTFM on RFID - it is an overwhelming rabbit hole, but less overwhelming than couple of days ago...

D12-14:
Days 12-14: NFC reverse engineering keys from reader intro/rtfm.
Aslo SubGhz - main vulnerability - you don’t need to decode, you just need to capture the signal

D15:
Back to Pico W Embedded software plaayground:
got 2 new boards to replace my cooked picow. 
Connected to Dev Board I got from Alim which is great and shiny, but I suspect they screwed up wiring for buttons, so the input is 0, and no-input is 1, which we can dance around.
Also managed to fire-up oled.
Still struggling with CO2 sensor, connecting MicroPython with Adafruit hardware that originally designed for their CircuitPy is not straight forward. 

Day 18 n 19 bash scripting
translating your solution is bash syntax is not easy. Learned basic control flow, bit or regex and args handling.  I think i might switch to something more useful

Day 20, 21
Ruby Exercism - love the mentoring and focusing on perfecting simle problems
