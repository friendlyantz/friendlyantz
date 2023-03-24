---
title: "100 Days of Code"
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - career
  - learning
  - 100DaysOfCode
  - android
  - kali
  - hacking

gallery:
- url: assets/images/100_days_of_code_2021/Screenshot_20221031-055158_Termux.png
  image_path: assets/images/100_days_of_code_2021/Screenshot_20221031-055158_Termux.png
  alt: "Kali Nethunter install via Termux"
  title: "Kali Nethunter install via Termux"
- url: assets/images/100_days_of_code_2021/Screenshot_20221031-061524_Termux_NH.png
  image_path: assets/images/100_days_of_code_2021/Screenshot_20221031-061524_Termux_NH.png
  alt: "NetHunter success"
  title: "NetHunter success"
- url: assets/images/100_days_of_code_2021/Screenshot_20221031-063654_NetHunter KeX.png
  image_path: assets/images/100_days_of_code_2021/Screenshot_20221031-063654_NetHunter KeX.png
  alt: "Kali NetHunter run"
  title: "Kali NetHunter run"
---
* Table of contents (do not remove this line)
{:toc}

Joining @saramic with his [100 Days of Code challange](https://saramic.github.io/100-days-of-code/code/2022/10/29/day-1.html)

Fasting, exercising and coding seems like a universal fomula for success and happiness as everythings seems to follow through if you do these 3 simple steps ;D

## Day1: PicoW, ESP32, Elixir, Ruby, Flipper Zero, CO2 sensor, Advent of Code, just, mruby

Don't know where to start with coding - so many exciting ideas and projects, here what I think I will do:
1. prepere my 1st talk at Ruby /RoRo Melbourne meetup - will start simple - a GitHub user profile repo, but I might overcomplicate things with BDD and emojilanng or ...?
1. try a functional language - already started peaking into Elixir, a sister language to Ruby, which I love
1. hack my own wifi Network and open my garage door(which key was lost) with a [Flippper Zero](https://flipperzero.one/) + ESP32 dev board which I can't wait to receive very soon. [RougeMaster](https://github.com/RogueMaster/flipperzero-firmware-wPlugins) firmware for flipper + [Marauder](https://github.com/justcallmekoko/ESP32Marauder/wiki/Flipper-Zero) for wifi biz via ESP32 4tw 👾
1. build CO2 sensor /alarm with Raspberry Pico W
1. Contribute to a Scientist / Refactorly project
1. try `mruby` maybe and flash it on PicoW or esp32?
1. try [just](https://github.com/casey/just) tool - alternative to the glorios `make`. It seems to be more friendly for beginners in some edge /challanging cases, and why not to try something new? It does not come by default in unix, sorry Mike.
1.  Advent of code - 2021: Continue and hopefully finish; 2022: coming in December, holy gang of musketeers help me

## Day 2: Emojicode 🐽

Played for the first time with `emojicode` - a fun example of an [esoteric language](https://en.wikipedia.org/wiki/Esoteric_programming_language):
> _"...a programming language designed to test the boundaries of computer programming language design, as a proof of concept, as software art, as a hacking interface to another language (particularly functional programming or procedural programming languages), or as a joke."_

Last Ruby Meetup reminded and inspired me to play with it, also 'hacking' and it's error handling sounds like fun🎉🙀

Other languages I was considering were `Piet`, [Shakespeare](https://en.wikipedia.org/wiki/Shakespeare_programming_language) and `Chef` - because I love art, literature and cooking 😂

I recon if you are a true and curious engineer - you definetly should play with some esoteric lang.

Emojicode sandbox repo ➡ [https://github.com/friendlyantz/emojicode-sandbox](https://github.com/friendlyantz/emojicode-sandbox)

## Day3: Kali Nethunter on Android
4hrs of sleep later... it is a day_3 of 100DaysOfCode challange when I am (very)happily back to the Android Termux setting up Kali Nethunter, thanks for inspiration and amazing guide [@davidbombal](https://twitter.com/davidbombal)

[David's instructions for rootless version install https://davidbombal.wiki/nhandroid](https://davidbombal.wiki/nhandroid)

{%
  include gallery 
  id="gallery"
%}

## Day 4: Elixir and Erlang install and intro

## Day 5: nmap, telnet

Toying with `nmap`, pwnd my first Hack The Box(HTB) puzzle, using nmap and connecting via open port 23, used by unencrypted telnet protocol
## Day 6: nmap, ftp
D6: learn how to probe open ports to determine service/version & OS info with `nmap -sV 10.129.0.9`
> Mac does not have decent cli tool for accessing ftp(or I couldn't find one quickly). you can connect to FTP via Finder->Go->Connect to Server, but it's just UI. Other option to use `ncftp` that loggs use in as `anonymous` by default

- hacking SMB on p445
> Again Mac does not have decent alternative to Linux `smbclient`, seems like one of the biggies for MacOS [https://discussions.apple.com/thread/6408765](https://discussions.apple.com/thread/6408765) 

## Day 7: nmap, dns, tcp
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

## Day 8: hacking with flipper

day8 with setting up my new flipper zero device RogueMaster firmware. Esp32 setup coming soon, need windows to run. Exe scripts. 

## Day 9: sub GHz hacking flipper 

day 9 involved a successful hardware hacking of NFC and sub-GHz spectrum. The feeling of joy is priceless, when the script just works as intended after days of research.

## Day 9-11: hacking RFID
day 9-11 involved hardware hacking and RTFM on RFID - it is an overwhelming rabbit hole, but less overwhelming than couple of days ago...

## Day 12-14: NFC reverse engineering keys from reader intro/rtfm.
Days 12-14: NFC reverse engineering keys from reader intro/rtfm.
Aslo SubGhz - main vulnerability - you don’t need to decode, you just need to capture the signal

## Day 15: Hardware with PicoW
Back to Pico W Embedded software plaayground:
got 2 new boards to replace my cooked picow. 
Connected to Dev Board I got from Alim which is great and shiny, but I suspect they screwed up wiring for buttons, so the input is 0, and no-input is 1, which we can dance around.
Also managed to fire-up oled.
Still struggling with CO2 sensor, connecting MicroPython with Adafruit hardware that originally designed for their CircuitPy is not straight forward. 

## Day 18 - 19 bash scripting
translating your solution is bash syntax is not easy. Learned basic control flow, bit or regex and args handling.  I think i might switch to something more useful

## Day 20 - 24 exercism
Ruby Exercism - love the mentoring and focusing on perfecting simle problems

## Day 25 PostreSQL
```sql
CREATE DATABASE test
`\l`
```
### Connect opt 1
```sh
psql -h localhost -U friendlyantz -p 5432 test
```
### Connect via SQL / `psql`
```sql
\c test
```

### Delete
```sql
DROP DATABASE test;
```
This is very dangerous

### add table
```sql
CREATE TABLE test (
  id INT,
  name VARCHAR(30),
  note TEXT,
  dob DATE );
```
