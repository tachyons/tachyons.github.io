---
layout: single
title: "Install Apertium Malayalam-english Pair Development Environment in Ubuntu"
modified:
categories:
comments: true
excerpt:
tags: []
image:
  feature:
date: 2014-08-20T07:13:36+05:30
---
Apertium is an open source machine translation toolkit .It is an extensible platform so that new language pairs can be added easily . It is currently in incubator state , we have to do a lot to to make it useful for translation of real life examples . I created ppas for the tools to make the process easier

Platform : ubuntu :14.04

* Add repositories

```bash
wget http://apertium.projectjj.com/apt/install-nightly.sh -O - | sudo bash
sudo add-apt-repository ppa:tinodidriksen/cg3
```

* Install tools

```bash
sudo apt-get update
sudo apt-get -f install locales build-essential automake subversion pkg-config gawk libtool apertium-all-dev

```

* Install English Malayalam pair data

```bash

svn co https://svn.code.sf.net/p/apertium/svn/incubator/apertium-mal-eng/
svn co https://svn.code.sf.net/p/apertium/svn/incubator/apertium-mal
```

* Compile

```bash
cd ~/apertium-mal
./autogen.sh
make
cd ~/apertium-mal-eng
./autogen.sh --with-lang1=../apertium-mal
make
```

* Installation is over , now you can test the system using

```bash
echo " അവന്‍ നല്ല കുട്ടിയാണ്  " | apertium -d . mal-eng
```

This will print
he is nice child
* Prefer Graphical user interface ? , you can try my simple gui for it

```bash
sudo apt-get install qt5-default espeak
git clone https://github.com/tachyons/mltranslator.git
cd mltranslator
qmake
make
sudo make install
```
Stuck ? Don’t worry just comment below . :-)
