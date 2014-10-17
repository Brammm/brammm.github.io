---
layout: post
title: "Symfony: Doing composer update with symlinked assets"
date: 2013-08-12 08:23
comments: false
categories: [Symfony, Tip]
---

```
$ composer update
...
Installing assets using the hard copy option
```
Ugh fuck. Now I have to make sure I don't forget `app/console assets:install --symlink`, or I'm gonna be cursing next time I'm adjusting CSS and I don't see my changes happening, because I forgot they're no longer symlinked.
<!-- more -->
By fumbling around, I noticed you can set an environment variable, called `SYMFONY_ASSETS_INSTALL` that will define the method used when installing assets after a composer update.

To do this on Mac OS X, add this to your ~/.bash_profile:

```
export SYMFONY_ASSETS_INSTALL=symlink" to ~/.bash_profile
```