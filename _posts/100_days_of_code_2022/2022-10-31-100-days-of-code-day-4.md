---
title: "100 Days of Code: day_4-10"
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
