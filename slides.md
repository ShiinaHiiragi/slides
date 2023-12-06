---
title: Title
theme: academic
highlighter: shiki
lineNumbers: true
themeConfig:
  paginationX: r
  paginationY: t
  paginationPagesDisabled: [1]

transition: fade-out
layout: cover
class: text-center
coverDate: ""
---

<link rel="stylesheet" href="global.css">

<h1 class="sink"> True Title </h1>

Grey Title

### Subtitle

---
transition: fade-out
layout: default
---

# Code & LaTeX

Use code snippets and get the highlighting directly.

```ts {all|1-6|8-12|all}
interface User {
  id: number
  firstName: string
  lastName: string
  role: string
}

function updateUser(id: number, update: User) {
  const user = getUser(id)
  const newUser = { ...user, ...update }
  saveUser(id, newUser)
}
```

LaTeX is supported out-of-box powered by [KaTeX](https://katex.org/).

$$
\begin{aligned}
X \to_c 1 & = X \\
X \to_c 1 \to_c a & = X \\
X \to_c (a + 1) \to_c (b + 1) & = X \to_c (X \to_c a \to_c (b + 1)) \to_c b
\end{aligned}
$$

---
transition: fade-out
layout: figure
figureCaption: Curated cover image for Slidev
figureFootnoteNumber: 1
figureUrl: https://source.unsplash.com/collection/94734566/1920x1080
---

# Figure-Center

<Footnotes separator>
  <Footnote :number=1>
    <a
      href="https://unsplash.com/collections/94734566/slidev"
      rel="noreferrer"
      target="_blank"
    >
      Anthony Fu
    </a>
  </Footnote>
</Footnotes>

---
transition: fade-out
layout: figure-side
figureCaption: Curated cover image for Slidev
figureFootnoteNumber: 1
figureUrl: https://source.unsplash.com/collection/94734566/1024x768
---

# Figure-Side

- Ensures figures are displayed nicely out of the box
- Allows placing the figure on the left or right
- Features an optional figure caption

<Footnotes separator>
  <Footnote :number=1>
    <a
      href="https://unsplash.com/collections/94734566/slidev"
      rel="noreferrer"
      target="_blank"
    >
      Anthony Fu
    </a>
  </Footnote>
</Footnotes>

---
transition: fade-out
layout: center
class: text-center
---

# Footnote & Quote

<span class="font-extralight">
  <q> Give credit where credit is due </q>
  <sup> 1 </sup>
  <sup> 2 </sup>
</span>

<Footnotes separator>
  <Footnote :number=1>
    Smart person
  </Footnote>
  <Footnote :number=2>
    Another smart person
  </Footnote>
</Footnotes>

---
transition: fade-out
layout: intro
class: text-left
---

<h1 class="sink"> End </h1>

Grey Title

### Subtitle
