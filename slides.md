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

<link rel="stylesheet" href="/global.css">

<h1 class="sink" style="font-size: 22px; line-height: 2em">
Transformer Models for Type Inference in the Simply Typed Lambda Calculus:
<br>
A Case Study in Deep Learning for Code
</h1>

### IBM Research

---
transition: fade-out
layout: default
---

# Introduction

- little exploration of transformer for reasoning about typed lambda calculi
    - typed lambda calculi are a lingua franc of programming languages
    - transformer are relatively unexplored in programming language applications
- contributions
    - the learning rate but not the warm-up steps, is the most important hyperparameter
    - extensive hyperparameter search was needed to achieve acceptable performance
    - Adafactor optimizer consistently achieves zero train error and zero validation error without any hyperparameters tuning

---
transition: fade-out
layout: figure
figureCaption: Transformers Model for STLC Type Inference
figureUrl: /model.svg
---

# Transformers

---
transition: fade-out
layout: default
---

# Simple Typed Lambda Calculus

1. let infinite set of type variables be $\mathrm V = \{\alpha, \beta, \gamma, \cdots\}$, then the set of simple types $\mathrm T$ is defined by
    - type variable: if $\alpha \in \mathrm V$, then $\alpha \in \mathrm T$
    - arrow type: if $\sigma, \tau \in \mathrm T$, then $(\sigma \to \tau) \in \mathrm T$
2. derivation rules for $\lambda_{\to}$:
    - variables: if $x: \sigma \in \Gamma$, then $\Gamma \vdash x: \sigma$

    - application: $\dfrac{\Gamma \vdash M: \sigma \to \tau \quad \quad \Gamma \to N: \sigma}{\Gamma \vdash MN: \tau}$

    - abstraction: $\dfrac{\Gamma, x: \sigma \vdash M: \tau}{\Gamma \vdash \lambda x:\sigma.M: \sigma \to \tau}$
3. context: a list of declarations with different subjects
    - statement: be of the form $M: \sigma$, where $M \in \Lambda_{\mathrm T}$ and $\sigma \in \mathrm T$
    - declaration: a statement with a variable as subject

---
transition: fade-out
layout: default
---

# Type Inference

- type inference in OCaml or Scala is $\text{EXPTIME}$
- more expressive type theories (e.g. those used for theorem proving) becomes undecidable
- Type inference in the simply typed lambda calculusn is PTIME-completeness

$$
\lambda y: \alpha \to \beta. \lambda z: \alpha. yz: ?
$$

type inference for STLC is an excellent few-shot learning benchmark

<!-- TODO -->

---
transition: fade-out
layout: default
---

# Dataset for Type Inference

1. initial context with a single base variable $x$ with base type $T$ such that $\Gamma = \{x: T\}$
2. generate a random type $\tau$ and then proceed to generate a term $e$ with that type
    - trivial in STLC by inspecting AST of the term
    - undecidable in general

**Random type generation:**
1. input
    - a list of valid base types from global context $\Gamma$
    - the maximum allowed depth for any type
    - probability distribution for when to (1) generate a base type; (2)
to branch
2. output: a list of types

intersection of repeated types between the train, validation and test sets is not allowed

---
transition: fade-out
layout: default
---

# Dataset for Type Inference

1. initial context with a single base variable $x$ with base type $T$ such that $\Gamma = \{x: T\}$
2. generate a random type $\tau$ and then proceed to generate a term $e$ with that type
    - trivial in STLC by inspecting AST of the term
    - undecidable in general

**Random term generation**
1. input
    - the desired type $\tau$
    - the **current dynamic context** $\gamma'$ (starts off as the global context $\Gamma$)
    - the probability distribution for branching
    - the maximum allowed depth for any term
2. output: a single term $e$ of type $\tau$

context is always available: generating abstraction $\to$ adding bounded variables to context

---
transition: fade-out
layout: default
---

# Dataset for Type Inference

when generating a term, there are two main cases to consider:
1. if the given type $\tau$ is a base type: two options, uniform probability and the first for max depth
    - choosing a variable from the current dynamic context $\Gamma'$
    - generate an application of the form $e_1 e_2: \tau$
        - generate a new type $\tau' \to \tau$
        - generate a term $e_1: \tau' \to \tau$
        - generate a term $e_2$ with type $\tau'$
2. if $\tau$ is an arrow type $\tau' \to \tau''$: three options, uniform probability
    - choosing a variable from the current dynamic context $\Gamma'$
    - create a new abstraction that directly has that type $\tau' \to \tau''$
    - create an application that ultimately returns  
    the current depth of the term has reached the limit: return a variable from the dynamic context or an abstraction of depth 1

---
transition: fade-out
layout: default
---

# Architecture

#### Overview

1. preprocessed sequence representation of the CST of the inputs
    $$
    [_{\text{term}} [_{\text{param}}y] \ [_{\text{type}} [_{\text{type}}\alpha] \ [_{\text{type}}\beta]] \ [_{\text{term}}[_{\text{param}} z: \alpha] \ [_{\text{term}} [_{\text{term}} y] \ [_{\text{term}} z]]]] ^{\textsf{1}}
    $$
2. embedding dimension of size 1024
3. three layers for both the decoder and encoder

<br/>

#### Preprocessing of names

In program synthesis, there does not exist a closed vocabulary as in natural
language
1. only use 32 different bound variable names to keep closed vocabulary
assumption true
2. all bound variables have been preprocessed such that they are renamed according to their Breadth-First Search order

<Footnotes separator>
  <Footnote :number=1>
    a possible speculation of CST
  </Footnote>
</Footnotes>

---
transition: fade-out
layout: default
---

# Architecture

#### The CST input to Encoder and Decoder

- input to encoder: BFS traversal of the CST
    - append the special tokens "sos", "eos" at the start & end of the
sequence for each term
    - add the positional embedding from the original transformer to the input sequence
    - append a pad token to the shorter term sequences up to the longest term sequence <sup> 1 </sup>
- input to decoder: follow the same input format

bound variables of the lambda abstractions are not hidden from the model

<Footnotes separator>
  <Footnote :number=1>
    these pad tokens are masked and never affect the output of the model
  </Footnote>
</Footnotes>

---
transition: fade-out
layout: intro
class: text-left
---

<h1 class="sink"> 12/6/2023 </h1>

23110240122
