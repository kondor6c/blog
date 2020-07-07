---
title: "pyenv" 
date: "2016-03-27"
tags:
  - "linux"
slug: "pyenv"
description: "Version managers for languages"

---


Version Managers
================

I have been enjoying the idea of version managers for interpreted languages like Ruby, Python and even NodeJS. Python likes to advocate using virtualenv, which works very well when interacting and writing the code. However when interacting with automation like Ansible or Chef this becomes difficult since you have to source the shell scripts to install a given version.

Some might argue that there isn't a need to have one of these version managers, and as you might guess I disagree with that, and advocate for them. If you lock in the version that you are testing/developing with this allows for more stability (no surprises is a good thing in production). You can then bring in a new version by putting it through a testing suite. This way you are not constrained to what version that the operating system is doing, it would be nice if the code worked correctly despite a new version, but frequently these are factors out of our control. When your building your project, setup a version manager might require a little bit of work but what if you get a little distracted (personal project focused) or get pulled my from a given project (corporate focused), having the version locked removes a lot of potential pain. Additionally this would allow moving to docker since all you need to do is setup that version manager and you could be on Arch Linux in a docker container. 

We don't know what the future holds, if we make our projects explicit on the versions it needs (even dependencies), then we are increasing the stability.


I frequently try to argue the other side, and two arguments I can see is that it's easier to just do "yum install ruby-devel" or that it increases the likelihood that package requirements/dependencies become stale. The ease of use of many version managers are increasingly more simple, all you need to do is switch to the development user and pipe the install script from curl to bash. Now the former I might have to agree with you on, and generally you could specify in the requirements that a "greater than" version is acceptable and lock it when you feel it is a concern.
To add credibility to my statements I have used `RVM <https://rvm.io/> in production in these examples gitlab, a custom application, and with Canvas LMS. I have code in place for `NVM <https://github.com/creationix/nvm>` (NodeJS) and I am investigating `pyenv <https://github.com/yyuu/pyenv-installer>` for Python.

