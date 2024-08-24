---
# layout: posts
# author_profile: true
title: "JS"
permalink: /js/
excerpt: "JS, CSS mad challenge notes"
# last_modified_at: 2016-11-03T11:13:12-04:00
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - template
# toc: true
# toc_sticky: true
# toc_label: "My Table of Contents"
# toc_icon: "cog"

---

- [JavaScript30 challange by WesBos](https://courses.wesbos.com/)
- [My Repo](https://github.com/friendlyantz/JavaScript30)

# Notes

- Array operations from D4 is really useful fundamentals
- alternative to `console.log()` is `console.table(fullNames);`

# Day 3: CSS variables

```css
<style>
      :root {
        --base: #00ff55;
        --spacing: 30px;
        --blur: 10px;
      }

      img {
        padding: var(--spacing);
        background: var(--base);
        filter: blur(var(--blur));
      }

      .hl {
        color: var(--base);
      }
/* etc */
```

basic console log

```js
    <script>
      // returns nodeList
      const inputs = document.querySelectorAll(".controls input");

      function handleUpdate() {
        // console.log(this.value);
        // console.log(this.dataset); // data from 'data-xxx' class element

        const suffix = this.dataset.sizing || "";
        document.documentElement.style.setProperty(
          `--${this.name}`,
          this.value + suffix
        );
      }

      inputs.forEach((input) => input.addEventListener("change", handleUpdate));
      // or mouse move (when dragging while changing)
      inputs.forEach((input) =>
        input.addEventListener("mousemove", handleUpdate)
      );
```

also can override CSS variables

```html
<h2 style="--base:#BADA55">...
```
