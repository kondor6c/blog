---
title: Golang with no Googling
date: 2019-03-17
category: programming
tags:
 - golang
 - learning
 - linux
slug: golang_with_no_googling
description: As an exercise, I tried writing Go without Googling

---

I spent a good amount of time learning Golang (Go, but since using two letters in any form of a search usually is an exercise in futility). My goal was fairly simple, I wanted to display some system statistics via HTTP by reading them directly from /proc. I saw that Golang had a module in the standard library call "syscall" and I decided that it might be fun to give that a try. Since I have seen a fair amount of syscalls by using strace and could always use trusty man pages, I thought this might be interesting and I might learn something in the process. I also was feeling fairly confident with the langauge after ingesting data from the web with Bleve (Golang Elasticsearch). It seems many of my results have been from Medium or StackOverflow, I really don't care for Medium, I get the feeling that it is tracking me; additionally the annoying notification that I need to sign up reminds me of Quora.
I knew from experience that generally I would need to stat the file, then read and write to the location I wanted. A fairly easy operation, having considered all things that could be involved. I could probably even skip the stat since I know it is there and that I can open it. To check my thought process and keeping with my no internet searching exercise, I did an strace on cat just to check my thought process.

::

  execve("/bin/cat", ["cat", "/proc/interrupts"], 0x7ffd84c42188 /* 82 vars */) = 0
  brk(NULL)                               = 0x560adc986000
  access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
  openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
  fstat(3LL, 1886056, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f6688407000
  mprotect(0x7f6688429000, 1708032, PROT_NONE) = 0
  mmap(0x7f6688429000, 1404928, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x22000) = 0x7f6688429000
  mmap(0x7f6688580000, 299008, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x179000) = 0x7f6688580000
  mmap(0x7f66885ca000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1c2000) = 0x7f66885ca000
  mmap(0x7f66885d0000, 14184, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f66885d0000
  close(3)                                = 0
  arch_prctl(ARCH_SET_FS, 0x7f66885d5540) = 0
  mprotect(0x7f66885ca000, 16384, PROT_READ) = 0
  mprotect(0x560adba2a000, 4096, PROT_READ) = 0
  mprotect(0x7f6688637000, 4096, PROT_READ) = 0
  munmap(0x7f66885d6000, 236364)          = 0
  brk(NULL)                               = 0x560adc986000
  brk(0x560adc9a7000)                     = 0x560adc9a7000
  openat(AT_FDCWD, "/usr/lib64/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
  fstat(3, {st_mode=S_IFREG|0644, st_size=4554608, ...}) = 0
  mmap(NULL, 4554608, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f6687faf000
  close(3)                                = 0
  fstat(1, {st_mode=S_IFCHR|0600, st_rdev=makedev(0x88, 0x2), ...}) = 0
  openat(AT_FDCWD, "/proc/interrupts", O_RDONLY) = 3
  fstat(3, {st_mode=S_IFREG|0444, st_size=0, ...}) = 0
  fadvise64(3, 0, 0, POSIX_FADV_SEQUENTIAL) = 0

Pretty trivial, as I thought, but I looked at the AT_FDCWD. This is due to the fact the file being opened might be relative. I then tried to use Golang to grab the current working directory. Using Godoc this was my simple result.

.. code-block:: go

  package main

  // an exercise where I am not going to "google" anything, use only reference docs
  
  import (
  	"fmt"
  	"syscall"
  )
  
  func main() {
  	var cwd_fd int
  	var err error
  	cwd_fd, err := syscall.Getcwd([]byte("/"))
  	if err != nil {
  		panic(err)
  	}
  	fmt.Println(cwd_fd)
  }

Upon running this I got a panic, err was hit and it said that

| panic: numerical result out of range

I double checked Godoc and it does return and int, I tried an int64 and that was indeed the wrong type, I also tried to automatically assign the type using good ole ":=" and got the same result. I still plan on continuing this exercise, maybe even have this little project/program be specifically dedicated to not searching. Like a search free clean room implementation. I still don't know if others have encountered this, but the syscall page doesn't have much for documentation (probably because many of these are already standard and documented elsewhere). The overview of the docs does say that this module is deprecated, but I wonder why it won't work at all, I also wonder if the GCC Go might have a different result. Another fun excercise might be to compare the benchmarks of both to see which might perform better. I would think that the syscall might since there isn't any guessing or fringe attempts that would be going on. This attempt was neat to do and I actually recommend it! Have fun.
