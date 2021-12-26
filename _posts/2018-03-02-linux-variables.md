---
layout: post
title: Linux Variables
date:   2018-03-02 10:01:27 +1300
tags: bash
category: linux
---

*What is variable? Simply put `y = ax+b` and `y` is the variable*

<!--more-->


## get or set variable


Variables start with `$`, you can use the `echo` command to access variable's value, use brakets `${variable}` if necessary. use `=` to assign value

```
Frank@cavalry:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

[root@www ~] # echo $myname
<== there is no data here ~ because this variable has not been configured! Is empty!

[root@www ~] # myname=Frank

[root@www ~] # echo $myname
Frank <== appears! Because this variable is already configured!
```


## assign rules

Yeah, there are rules : (

```
[root@www ~]# 12name=Frank
- bash: 12name=VBird: Command not found <== screen displays error! Because you can't start with a number!

[root@www ~]# name= Frank <== error again! Because there is a space!
[root@www ~]# name=Frank <==OK!

[root@www ~]# name=Frank's name
# single and double quotes must be paired, there is only one single quote in the above configuration, press `[CTRL]+C`!

[root@www ~]# name="Frank's name" <==This is OK!

[root@www ~]# name='Frank's name' <== again, failed!

[root@www ~]# name=Frank\'\ name <==OK!

[root@www ~]# PATH=$PATH:/home/dmtsai/bin <==add "/home/dmtsai/bin" 
[root@www ~]# PATH="$PATH":/home/dmtsai/bin <==OK!
[root@www ~]# PATH=${PATH}:/home/dmtsai/bin <==OK!

[root@www ~]# name=$nameyes <==wrong!

[root@www ~]# name="$name"yes <==need to get the value first
[root@www ~]# name=${name}yes <==use this!

[root@www ~]# name=Frank
[root@www ~]# bash <== in a sub-shell
[root@www ~]# echo $name <== no value
[root@www ~]# exit <== Leave this sub-shell

# Command `export` will make a variable become environment variable, you can check with command `env`

[root@www ~]# export name
[root@www ~]# bash
[root@www ~]# echo $name
Frank <== work this time
[root@www ~]# exit 

[root-@www ~]# cd /lib/modules/`uname-r`/kernel
[root-@www ~]# cd /lib/modules/$(uname-r)/kernel

#Single quote and double quote

[root@www ~]# myname="$name is me"
[root@www ~]# echo $myname
Frank is me
[root@www ~]# myname='$name is me'
[root@www ~]# echo $myname
$name is me

Double quotes "" can parse variable but not single quotes
what inside backticks `` is executed and replaced by the output of the command. That is called command substitution because it is substituted with the output of the command.

[root@www ~]# ls -ld `find -name nginx` <==CentOS8

```


## environment variables


Use `env` or `export` to check out current env variables

```
[root@www ~]# env
...
LANG=C.UTF-8
SUDO_GID=1003
USERNAME=root
SUDO_COMMAND=/bin/bash
USER=root
PWD=/lib/modules
HOME=/root
SUDO_USER=gcpvm
XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop
name=Frank
...

```

- $HOME Current user's home directory. 
- $SHELL Tell us which SHELL is currently been used
- HISTSIZE This is related to the "historical command", that is, the commands we have made can be recorded by the system, and the number of strokes recorded is configured by this value.
- MAIL When we use mail to receive a letter, the system will read the mailbox file (mailbox).
- PATH The directory is separated from the directory by a colon (:). Since the file is searched sequentially by the directory in the PATH variable, the order of the directory is also important.
- LANG This is important! For example, when we start some perl programming language files, he will actively analyze the language data files. If he finds some coding language files that he cannot parse, it may cause errors. In general, our Chinese encoding is usually zh_tw.big5 or zh_tw.UTF-8, which cannot be easily interpreted, so sometimes the language data may need to be revised. This part will be introduced in the next section!
- RANDOM This thing is a random number variable. Now most distributions have a random number generator and that is /dev/random file. We can get the RANDOM value through the variable related to the RANDOM number file ($RANDOM). In BASH, the contents of this RANDOM variable are between 0 and 32767, so whenever you echo $RANDOM, the system will actively pull out a value between 0 and 32767. What if I want to use values between 0 and 9? To declare the numeric type, declare the numeric type using the declare method, then do so:

```
[root@www ~]# declare -i number=$RANDOM*10/32768 ; echo $number
```


Use `set` to check out all (environment and custom) variables


Bash is not just about environment variables. There are also variables related to the bash interface and user-defined variables that exist. So how do we look at these variables? At this point, use the `set` command. 



In general, variables related to the interface of our current shell, whether they are environment variables or not, are usually configured with uppercase characters. That is to say, "Basically, by default on Linux, variables configured with {uppercase} are generally required variables within the system." OK! OK! So which of those variables are more important? There are probably a few of them!


**PS1 :(prompt characters)**


This is PS1, this is our command prompt character. Every time we press the [Enter] key to run a command, and the prompt character appears again, we will actively read the variable value. What is shown on the top PS1 is some special symbols and these special symbols can show different information. The default PS1 variable content of each bash may be slightly different, never mind, "get used to your own habits" is good. You can use Man bash to look up the instructions for PS1 to understand some of the symbolic meanings below.


- \d: Date format with "Week/Month/Day" displayed, such as: "Mon Feb 2"

- \H: Full host name. For example, Bird's exercise machine is www.vbird.tsai
- \h: Only take the name of the host before the first decimal point, if the bird's host is omitted after "WWW"
- \t: Display time, "HH:MM:SS" in 24-hour format
- \T: Display time, "HH:MM:SS" in 12-hour format
- \A: Display time, "HH:MM" in 24-hour format
- \@ : Display time, "AM/PM" style in 12-hour format
- \u: The current user's account name, such as "root";
- \v: BASH version information, such as Bird's test motherboard is 3.2.25(1), only take "3.2" to display
- \w: Full working directory name, directory name written from the root directory. But the home directory will be replaced by - ~;
- \W: Gets the working directory name using the basename function, so only the last directory name is listed.
- \#: Which order was given.
- \$: Prompt character, # if root, $~ if not


Ok, let's take a look at CentOS's default PS1 content: "[\u@\h \W]\$". Now you know what those backslashes mean? Be careful! The data after the backslash is a special function of PS1, regardless of bash's variable configuration! Don't get mixed up! Now do you know why your command prompt is: "[root-@www ~]#"? Okay, so let's say I want to have a prompt like this:

```
[root @ WWW/home/dmtsai thus # 12] were #
```

The # stands for the order given at the 12th time. Here's how it works:

```
[root@www ~]# CD /home
[root@www home]# PS1='[\u@\h \w \A #\#]\$'
[root@WWW /home 17:02 #85]#
```

magic :)


**$:(PID)**

try `echo $$` it represents the "thread code of the current Shell"


**? (last command return)**

Run `echo $?` you get 0 that means your last command success