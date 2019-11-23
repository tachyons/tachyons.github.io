---
title: "Apertium Malayalam-english Pair Tools Usage and Directory Structure"
modified:
categories: 
comments: true
excerpt:
layout: single
tags: []
tags: 
image:
  feature:
date: 2014-08-20T07:33:25+05:30
---
Apertium malayalam english pair contain two directories

1. apertium-mal -> conatins malayalam specific rules and resources
2. apertium-mal-eng ->contains translation rules

* Apertium-mal

1. apertium-mal.mal.lexc  -> contain mono lingual dictionary in lexec format , it is contain morphotactics rules
2. apertium-mal.mal.twol -> contain morphophonology rules (changes when morphemes are joined together)
3. apertium-mal.mal.tsx ->tag set to train the tagger
4. apertium-mal.mal.rlx  -> constarint grammar rules

* Apertium-mal-eng

1. apertium-mal-eng.eng.dix -> mono lingual dictionary for english in lttoolbox format
2. apertium-mal-eng.post-eng.dix -> English post generator
3. apertium-mal-eng.eng-mal.t1x – > english to malayalam chunker
4. apertium-mal-eng.eng-mal.t2x -> english to malayalam interchunk
5. apertium-mal-eng.eng-mal.t3x ->english to malayalam post chunk

Translation
---------------------

```bash
cd apertium-mal-eng
echo “മലയാളം വാക്യം ” | apertium -d . mal-eng
```
where

-d . -> directory=current directory

mal-eng -> malayalam english mode

Morphological analyser
---------------------

```bash
echo “മലയാളം “| lt-proc mal-eng.automorf.bin
```

GUI for translator
---------------------
![Qt GUI]({{ site.url }}/images/mltranslator.png)

Step by step process view (Developer ->Modes viewer)
---------------------

![Modes viewer]({{ site.url }}/images/modes.png)

