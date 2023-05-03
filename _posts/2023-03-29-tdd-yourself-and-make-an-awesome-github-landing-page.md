---
title: "My talk at Ruby on Rails Oceania in Melbourne slides"
permalink: /roro-melbourne-tdd-and-github-landing-page/
# last_modified_at: 2016-11-03T11:13:12-04:00
collection: learning
excerpt: "RoRo Melbourne presentation on TDD and GitHub landing page"
# TODO not sure if categories and tags are supported outside of `_posts` dir
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
toc: true
# layout: posts
---

## TDD Yourself And Make An Awesome Github Landing Page


##### You don't need anything to start TDD

```bash
if True; then                                                                                      
 echo 'yolo'
fiπ
```

---

```bash
if curl -s -N 'https://github.com/terry-d-4tw/' | grep -q "I’m currently working on HolyC"; then
 echo "it worked! 🎉"; else
 echo "it did not work 😔";
fi
```

---

```
Note: 
Be careful with emojies.
And do not get too emotional with `!!!`
```

---

##### Return meaningful `exitcodes`

```bash
if curl -s -N 'https://github.com/terry-d-4tw/' | grep -q "I’m currently working on HolyC"; then
 echo "it worked! 🎉"; exit 0; else
 echo "it did not work 😔"; return 1
fi
```

```ruby
# machines do not understand your emojies, 
# unless it is an emojicode 🍇
```

---

##### FAIL MESSAGE

```bash
FAIL=$(curl -s https://i.imgur.com/Afil0Do.png | imgcat)
```

> <https://github.com/eddieantonio/imgcat> -
_it's like `cat`, but for images😽_

---

##### SUCCESS MESSAGE

```bash
SUCCESS=$(curl -s https://avatars.steamstatic.com/e924aab1db76fe96f490b51eeed2893571c5d41b_full.jpg | imgcat)
```

---

```bash
if curl -s -N 'https://github.com/terry-d-4tw/' | grep -q "I’m currently working on HolyC"; then
        echo $SUCCESS; return 0; else
        echo $FAIL; return 1
fi
```

 or with caching html

 ```bash
WEBSITE_CACHE=$(curl https://github.com/terry-d-4tw/)

if echo $WEBSITE_CACHE | grep -q "ASSERTION"; then                                                  
 echo $SUCCESS; return 0; else
 echo $FAIL; return 1; 
fi
```

---

##### CI/CD it via GitHub Action Workflow

```bash
if curl -s -N 'https://github.com/terry-d-4tw/' | grep -q "I love Ruby too"; then
 echo "it worked! 🎉"; exit 0; else
 echo "it did not work 😔"; exit 1
fi
```

---
Cool stats for the README.md
<https://github-readme-stats.vercel.app/>

```
![Terry's GitHub stats](https://github-readme-stats.vercel.app/api?username=terry-d-4tw&show_icons=true&theme=synthwave)
```

---

##### Create a GitHub page

Settings -> Pages

Source / Branch: `your_master_branch`

<https://github.com/terry-d-4tw/terry-d-4tw/settings/pages>

---

##### Add JAMStack Framwork

##### Jekyll

create

```
_config.yml
```

with:

```yml
theme: minima
```

---

Thank you for reading
