---
layout: post
title: "Change local port in Sendmail"
date: 2020-04-28 16:25:06 -0700
comments: true
---

First we must change the port(2525) for the daemon in the following file: `nano /etc/mail/sendmail.mc`
```
FEATURE(`no_default_msa')dnl
DAEMON_OPTIONS(`Port=2525,Addr=127.0.0.1, Name=MTA-v4')dnl
DAEMON_OPTIONS(`Port=587,Addr=127.0.0.1, M=Ea, Name=MSP-v4')dnl
```

To be able to send message to port 2525 with the commands mail and sendmail, we will have to modify the following file and create a SMART_HOST in this: `nano /etc/mail/submit.mc`
```
divert(-1)dnl
divert(0)dnl
define(`_USE_ETC_MAIL_')dnl
include(`/usr/share/sendmail/cf/m4/cf.m4')dnl
VERSIONID(`$Id: submit.mc, v 8.15.2-3 2015-12-10 18:02:49 cowboy Exp $')
OSTYPE(`debian')dnl
DOMAIN(`debian-msp')dnl
define(`RELAY_MAILER_ARGS',`TCP $h 2525')dnl
define(`SMART_HOST',`127.0.0.1')dnl
FEATURE(`msp', `[127.0.0.1]')dnl
```

Apply settings whit the following commands:
```
make -C /etc/mail
sendmailconfig
m4 /etc/mail/submit.mc > /etc/mail/submit.cf
m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf
```
Restart de daemon sendmail: `systemctl restart sendmail`

Check that the daemon is running on the port we want:
```
# lsof -i -P -n | grep sendmail
sendmail- 31492     root    4u  IPv4 10544619      0t0  TCP 127.0.0.1:2525 (LISTEN)
sendmail- 31492     root    5u  IPv4 10544620      0t0  TCP 127.0.0.1:587 (LISTEN)
```

With this we can already send the email with the change local port:
```
echo "Test Email" | mail -s "Subject Here" admin@example.com
```

