---
title: Vi Everywhere
excerpt: visual editor tricks outside of VIM
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - vim
  - emacs
header:
  video:
    id: l6dwu6cCfkM
    provider: youtube
---

Mike's talk. loved it!

> Source:
[https://failure-driven.com/presentations/20180627-vi-everywhere-whats-your-superpower/](https://failure-driven.com/presentations/20180627-vi-everywhere-whats-your-superpower/)

TLDR
```bash
set -o | egrep -e 'emacs|vi '
set -o vi
```
# Further Read and EMACS {#further-read}
BUT, you might still want to invest in learning some basic shortcuts of EMACS, as VI binding are not always available, also look further into [ReadLine](https://readline.kablamo.org/emacs.html)

Shortcuts I use on Mac that I think are useful and does not cause conflicts with other shortcuts(Note: `Alt` is `Option` key):

|Command|Description|
|:---|:---|
|Ctrl-a|Move to the start of the current line.|
Ctrl-e|Move to the end of the line.
Alt-f|Move forward to the end of the next word. Words are alphanumeric.
Alt-b|Move back to the start of the current or previous word. Words are alphanumeric.
Ctrl-l|Clear the screen.
|||
Ctrl-k|Kill (cut) forwards to the end of the line.
Ctrl-u|Kill (cut) backwards to the start of the line.
|
Alt-d|Kill (cut) forwards to the end of the current word.
Ctrl-w|Kill (cut) backwards to the start of the current word.
|
Ctrl-y|Yank (paste) the top of the kill ring.
|
Ctrl-_|undo
|
Ctrl-d|Delete the character under the cursor.
Delete|Delete the character before the cursor.
|
Alt-u|Uppercase till the end of the current word.
|
Alt-t|Transpose(switch order) 2 words.

---

VIM combos:

- replace `:%s/ladida/dududu`
- Ruby linting `:! rubocop -A %`
- `va{V` - select all in {}block and switch to line mode, press `o` to jump between {}
- regex command to bulk replace selected: `:` auto-incerts this`:'<,'>` then type the following`s/\(\w.*\)/your new data = \1;`
- increment sequentially the selection `g<C-a>`
- fix indentation of all paragraph `=ap`
- jump to prev file `<C-^>`


- `gww`  - auto format line width
- `ctrl + w + v / s` open in tab
- `ctrl + 6` last file toggle

---

- [VIM Adventures](https://vim-adventures.com/) - learn VI shortcuts game


---

