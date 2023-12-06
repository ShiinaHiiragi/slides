---
title: STLC
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

<h1 class="sink" style="font-size: 22px; line-height: 2em">
Transformer Models for Type Inference in the Simply Typed Lambda Calculus:
<br>
A Case Study in Deep Learning for Code
</h1>

#### IBM Research

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

type inference for STLC is an excellent few-shot learning benchmark
- type inference in OCaml or Scala is $\text{EXPTIME}$
- more expressive type theories (e.g. those used for theorem proving) becomes undecidable
- type inference in the simply typed lambda calculusn is PTIME-completeness
    $$
    \lambda y: \alpha \to \beta. \lambda z: \alpha. yz: ?
    $$

<img src="/algo.png" class="h-50% ml-25% shadow" />

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

preprocessed sequence representation of the CST of the inputs

$$
[_{\text{term}} [_{\text{param}}y] \ [_{\text{type}} [_{\text{type}}\alpha] \ [_{\text{type}}\beta]] \ [_{\text{term}}[_{\text{param}} z: \alpha] \ [_{\text{term}} [_{\text{term}} y] \ [_{\text{term}} z]]]] ^{\textsf{1}}
$$

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

#### Encoder and Decoder stacks

- 1024 for all embedding dimensions
- 3 for the number of layers for both the encoder layer and decoder layer
- 0.1 dropout rate

<Footnotes separator>
  <Footnote :number=1>
    these pad tokens are masked and never affect the output of the model
  </Footnote>
</Footnotes>

---
transition: fade-out
layout: default
---

# Architecture

#### Training

- compute the loss by computing the cross-entropy loss between
    - the model’s predicted rule sequence
    - the rule sequence of the target type
- study of different behaviors of optimizers:
    - Adam
    - RAdam
    - Adafactor

#### Type Synthesis

- expand the model’s predicted rules in BFS order
- if a pair of rules in the could not have been applied in BFS order, then the algorithm predict the dummy error type

---
transition: fade-out
layout: default
---

# Experiments

#### Is warm-up phase<sup>1</sup> important?

| number warm-up steps | $\text{lr} = 10^{-3}$ | $\text{lr} = 10^{-4}$ | $\text{lr} = 10^{-5}$ |
| :------------------: | :-------------------: | :-------------------: | :-------------------: |
|          0           |         0.31          |         0.95          |         0.99          |
|        2,000         |         0.09          |         0.93          |         0.99          |
|        4,000         |         0.27          |         0.76          |         1.00          |
|        10,000        |         0.31          |         0.97          |         0.99          |

<br/>

optimizer: Adam

- changes of warm-up steps do not change training accuracy
- a good choice of learning rate is more important than having a warm-up phase

<Footnotes separator>
  <Footnote :number=1>
    the learning rate starts at zero and is increased to a target value
  </Footnote>
</Footnotes>

---
transition: fade-out
layout: default
---

# Experiments

#### Does RAdam fix the convergence issues?

| number warm-up steps | $\text{lr} = 10^{-3}$ | $\text{lr} = 10^{-4}$ | $\text{lr} = 10^{-5}$ |
| :------------------: | :-------------------: | :-------------------: | :-------------------: |
|          0           |   training diverged   |         0.91          |         1.00          |
|        2,000         |           -           |         0.96          |         0.99          |
|        4,000         |           -           |         0.80          |         1.00          |
|        10,000        |           -           |         0.96          |         0.99          |

optimizer: RAdam
- RAdam behaves similarly to Adam with or without a warm-up phase
- $\text{lr} = 10^{-3}$: try decaying the learning rate with two different decay schedulers (no help)
- RAdam diverges more than Adam and is not recommended

---
transition: fade-out
layout: default
---

# Experiments

#### Experiments with Adafactor

| embedding method | train accuracy | validation accuracy | iter/sec |  runtime  |
| :--------------: | :------------: | :-----------------: | :------: | :-------: |
|   token (NLP)    |      1.00      |        0.99         |   0.2    | 1.7 hours |
|    Path Embed    |      1.00      |        0.99         |   0.2    | 2.1 hours |
|   Depth Embed    |      1.00      |        1.00         |   0.2    | 2.4 hours |

<br/>

- path embedding method: replaces each node in the CST with the path to that node
    - path: the sequence of symbols visited when traversing to a specific node in CST
    - the path embedding computation is $\mathrm{embed}(x_{\mathrm{path}}) := \mathrm{FDD}[x_1; x_2; \cdots; x_L] \in \mathrm R^{L \times D}$
        - $L$: the maximum length of a path (set to 13 in this experiment)
        - $x_i \in \mathrm R^{D}$: the embedding of the symbol in the path

---
transition: fade-out
layout: default
---

# Experiments

#### Experiments with Adafactor

| embedding method | train accuracy | validation accuracy | iter/sec |  runtime  |
| :--------------: | :------------: | :-----------------: | :------: | :-------: |
|   token (NLP)    |      1.00      |        0.99         |   0.2    | 1.7 hours |
|    Path Embed    |      1.00      |        0.99         |   0.2    | 2.1 hours |
|   Depth Embed    |      1.00      |        1.00         |   0.2    | 2.4 hours |

<br/>

- depth embedding method: $x_{\text{seq}} = [x_1, x_2, \cdots, x_L] + [\mathrm{DE}_1, \mathrm{DE}_2, \cdots, \mathrm{DE}_L]$
    - the path embedding method is expensive
    - $[x_1, x_2, \cdots, x_L] \in \mathrm R^{D \times L}$
    - $[\mathrm{DE}_1, \mathrm{DE}_2, \cdots, \mathrm{DE}_L] \in \mathrm R^{D \times L}$, where $\mathrm{DE}$ is the parent of each token in the CST

---
transition: fade-out
layout: default
---

# Experiments

#### Comparison of Adam’s and Adafactor’s Learning curves

<img src="/joint.svg" class="w-80% ml-10%" />

- Adafactor is able to train in just 2.17 hours, while it took 1 days and 15 hours for Adam
- annealing scheduler rate: $\text{scheduler\_rate} = \min(\text{one\_epoch}, \dfrac{2}{1-\text{beta\_2}})$ (2.5-fold speed up)

---
transition: fade-out
layout: intro
class: text-left
---

<h1 class="sink"> 12/6/2023 </h1>

23110240122
