+++
title = "bash tips 1"
date = "2016-03-29"
category = "devops"
tags = [ "linux" ]
slug = "bash_tips1"
description = "after reviewing some bash tips I post a reflection"

+++

BASH tips
=========

So I read a comment on Hacker News about ctrl+o and and the comment (and subsequent comments) made it sound as though it was this ultra helpful trick. Well I had a bit of difficulty finding what and how to use, perhaps its because I'm a bit stupid. So here is how it works, you type a command (I found it works best when you use ctrl+r then press ctrl+o) press ctrl+o instead of enter. It will run that command, then autocomplete the next command from the last previously typed entry that you ran when you pressed ctrl+o. Might be a little confusing. Take a look at this fake, yet focused on the concept session below
```
[kondor6c@workhorse ~]# <CTRL+R> service tomcat sto<CTRL+O>
Redirecting to /bin/systemctl stop  tomcat.service
[kondor6c@workhorse ~]# rsync -va current-version.war server:/app/tomcat/webapps/ #this will automatically appear! Just press to get the next line to automatically appear<CTRL+O>
kondor6c@server.mooo.com's password:                                                                
sending incremental file list
current-version.war                                                                          
sent 43,611,957 bytes  received 824 bytes  3,426.29 bytes/sec 
total size is 43,611,838  speedup is 0.99 

[kondor6c@workhorse ~]# service tomcat start <enter>
Redirecting to /bin/systemctl start  tomcat.service
```

As you can see ctrl+o would require explicit knowledge of what was ran last, otherwise you wasted time. Additionally I would want to group any commands that I would type after one another into a one liner or make it a script-able action. In short I'm glad I learned about it, I probably won't instruct others about it, as I think it will be harder to grasp, and I don't see the use case being that strong.

I have explored many "Bash tips" I think I know the vast majority of them. If you know of one I encourage you to share it. Here are just a couple that I have very recently discussed with coworkers/intern:

  * "ctrl + \" : sends a SIGQUIT as in a signal 3, which tries to dump the core, if not then it will generally quit with a higher priority than SIGINT, not really a BASH specific tip, but certainly useful.
  * "~." : When ssh'ing and you have a hung session, press "~." on the SSH session, this should interrupt it and bring you out. I have found that when my sessions are hung they are not always at a newline 

```
kondor6c@computer:~$ ~?
Supported escape sequences:
 ~.   - terminate connection (and any multiplexed sessions)
 ~B   - send a BREAK to the remote system
 ~C   - open a command line
 ~R   - request rekey
 ~V/v - decrease/increase verbosity (LogLevel)
 ~^Z  - suspend ssh
 ~#   - list forwarded connections
 ~&   - background ssh (when waiting for connections to terminate)
 ~?   - this message
 ~~   - send the escape character by typing it twice
(Note that escapes are only recognized immediately after newline.)
kondor6c@computer:~$ #<enter>
kondor6c@computer:~$ #<enter> then press ~.
kondor6c@computer:~$ Connection to computer closed.
```

