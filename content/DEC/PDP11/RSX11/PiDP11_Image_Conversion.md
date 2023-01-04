# Converting Pi11/70 images for use on real Qbus systems
The disk images provided with Oscar Vermuelen's beautiful [PiDP-11/70 kit](https://obsolescence.wixsite.com/obsolescence/pidp-11) (I have one- you should, too!) are extremely useful, packed full of great software and lovingly prepared by fellow PDP enthusiasts. They are also often built for UNIBUS PDP11s, whereas my PDP11 is Qbus. Below is how I have gone about converting these various images for use on a Qbus PDP11. With thanks to [Mark Matlock](http://rsx11m.com/) for his assistance (and patience :>). 

## RSX11M+
RSX11M+ is my personal favorite PDP11 operating system. There is just an inexplicable draw in the way that it behaves, so of course it was my first target. Rebuilding RSX11 requires at minimum a brand new SYSGEN, and since we are also running DECnet and Johnny Bilquist's BQTCP, we will also need fresh NETGEN and IPGENs. Let's tackle the SYSGEN first. Below is my SIMH configuration file:

```
set cpu 11/83, 4m, idle

; disks
set rq0 rd54
att rq0 PiDP11_DU0.dsk
set rq1 rd54
att rq1 PiDP11_DU1.dsk

; peripherals
set xq en
att xq0 bge0
```

From your SIMH PDP11:

```
>@SYSGEN
>;
>; RSX-11M-PLUS  V4.6  BL87  SYSGEN
>;
>; Copyright (c) 1995-1999 by Mentec Inc., U.S.A.
>;
>SET /NONAMED
>SET /UIC=[200,200]
>SET /DPRO=[RWED,RWED,RWE,R]
>;
>; To exit from the SYSGEN procedure at any time, type CTRL/Z.
>;
>; If you are unsure of the answer to a question for which a de-
>; fault answer exists, use the default answer.
>;
>;
>;
>;===================================================
>;  Choosing SYSGEN Options      04-JAN-2022 at 17:03
>;===================================================
>;
>;
>;
>; Every question is preceded by a question number (for example SU010)
>; which you can use to find the explanation of the question in the
>; RSX-11M-PLUS System Generation and Installation Guide.
>;
>; An explanation of every question is also available by pressing
>; the ESC key (or the ALTMODE key) in response to the question.
>;
>; If you are unfamiliar with the SYSGEN procedure, the explanation of
>; each question can be printed automatically before the question.
>;
>* SU010   Do you always want the explanation printed? [Y/N D:N]: N
>;
>; SYSGEN always creates saved answer files containing your responses
>; to the SYSGEN questions:
>;
>;    SYSGENSA1.CMD     Setup questions, Executive options
>;    SYSGENSA2.CMD     Peripheral configuration
>;    SYSGENSA3.CMD     Nonprivileged task builds
>;
>; You should perform a PREPGEN first to create saved answer files, and
>; then perform a SYSGEN, specifying those saved answer files as input
>; to the Executive, peripheral, and nonprivileged task build sections.
>;
>* SU020   Do you want to use a saved answer file as input for
>*         the Executive options? [Y/N D:N]: N
>;
>* SU040   Do you want to use a saved answer file as input for
>*         the peripheral configuration? [Y/N D:N]: N
>;
>* SU060   Do you want to use a saved answer file as input for
>*         the nonprivileged task builds? [Y/N D:N]: N
>;
>* SU080   Do you want to do a PREPGEN? [Y/N D:N]: N
>;
>* SU090   Enter the name of the disk drive containing your
>*         target system disk [ddnn:] [S R:2-5]: DU0
>;
>ASN DU0:=IN:
>ASN DU0:=OU:
>ASN DU0:=LB:
>ASN DU0:=WK:
>ASN DU0:=TK:
>ASN DU0:=BC:
>ASN DU0:=LI:
>ASN DU0:=OB:
>ASN DU0:=EX:
>ASN DU0:=MP:
>;
>; You can:
>;
>;    o  do a complete SYSGEN
>;
>;    o  continue a previous SYSGEN from where you left off
>;
>;    o  do an individual section of SYSGEN
>;
>;
>* SU120   Do you want to do a complete SYSGEN? [Y/N D:Y]:
>;
>INS [3,54]MAC/TASK=MACT0
>INS [3,54]PIP/TASK=PIPT0
>INS [3,54]LBR/TASK=LBRT0
>INS [3,54]TKB/TASK=TKBT0
>INS [3,54]VMR/TASK=VMRT0
[...]
```

SYSGEN in your real devices here. RSX11 under SIMH will boot fine regardless of discrepancies between the devices in SYSGEN and those enabled, at least in my experience. YMMV. If something is truly busted you'll know before you even start looking at the real system.

Once SYSGEN completes, boot as normal. To be safe, I like to make the following changes in `LB:[1,2]STARTUP.CMD`: 

```
[...]
	.SETT $CEX		.; change to SETF
	.SETT $DEC		.; change to SETF
	.SETF $NNS		.; change to SETF
[...]
	@LB:[1,2]INSPROG.CMD	.; change to .;@LB:[1,2]INSPROG.CMD
```

You can ignore the resulting `NCP` errors during boot. This will prevent any potential issues that could arise from DECnet, TCPIP, and their associated services being active during NETGEN/IPGEN. Simply undo these changes later to reenable. It is recommended that you SAVE with the following parameters for a multi-disk system:

```
sav /wb/mou="/acp=unique/lru=14/win=30"
```

If the system comes back up normally, we can continue with NETGEN.

```
>set def lb:[137,10]
>@netgen
>;
>; =====================================================================
>;  NET - RSX-11M-PLUS CEX System Generation Procedure
>;        Started at 17:48:00 on 04-JAN-2022
>; =====================================================================
>;
[...]
```

You will be prompted to make several choices. Answer as follows (this assumes you are not trying to build a router; if something isn't listed, just take the default):

```
>* 08.00 Is this generation to be a dry run [D=N]? [Y/N]: 
>* 09.00 Do you want a standard function network [D=N]? [Y/N]: Y
>* 11.00 Should old files be deleted [D=N]? [Y/N]: Y
>* 06.00 Should tasks link to the Memory Resident FCS library [D=N]? [Y/N]: Y
>* 01.00 Device Driver Process name [<RET>=Done] [S R:0-3]: QNA
>* 04.07 Set the state for QNA-0 ON when loading the network [D=N]? [Y/N]: Y
>* 01.00 What is the target node name [S R:0-6]: YOURNODENAME
>* 02.00 What is the target node address [S R:0.-8.]: YOURDECNETADDR
>* 03.00 Target node ID [D=None] [S R:0.-32.]: 32 character max node description
```

Your NETGEN is now complete, but if you shut down now the system will hard fail. If we inspect the [1,54] UIC, we can see a file version mismatch between the `RSX11M.STB` and `RSX11M.SYS` files:

```
>DIR RSX11M.SYS;* 
Directory DU0:[1,54]
4-JAN-2022 17:55

RSX11M.SYS;8        1026.   C  04-JAN-2022 17:40
RSX11M.SYS;6        1026.   C  14-FEB-2016 23:03
RSX11M.SYS;7        1026.   C  27-DEC-2018 10:11
RSX11M.SYS;9        1026.   C  04-JAN-2022 17:46

Total of 4104./4104. blocks in 4. files

>DIR RSX11M.STB;*


Directory DU0:[1,54]
4-JAN-2022 17:55

RSX11M.STB;4        37.        09-JAN-2016 23:12
RSX11M.STB;5        37.        27-DEC-2018 10:11
RSX11M.STB;6        37.        04-JAN-2022 17:34

Total of 111./111. blocks in 3. files
```

We need to make these files match using PIP. The general syntax is as follows:

```
pip rsx11m.sys;10=rsx11m.sys/nv
```

This will create a new version of the `RSX11M.SYS` file with version 10. We can validate this:

```
>DIR RSX11M.SYS;*


Directory DU0:[1,54]
4-JAN-2022 17:57

RSX11M.SYS;8        1026.   C  04-JAN-2022 17:40
RSX11M.SYS;6        1026.   C  14-FEB-2016 23:03
RSX11M.SYS;7        1026.   C  27-DEC-2018 10:11
RSX11M.SYS;9        1026.   C  04-JAN-2022 17:46
RSX11M.SYS;10       1026.   C  04-JAN-2022 17:46

Total of 5130./5130. blocks in 5. files
```

Do the same for `RSX11M.STB`:

```
>PIP RSX11M.STB;10=RSX11M.STB/NV
>DIR RSX11M.STB;*


Directory DU0:[1,54]
4-JAN-2022 17:58

RSX11M.STB;4        37.        09-JAN-2016 23:12
RSX11M.STB;5        37.        27-DEC-2018 10:11
RSX11M.STB;6        37.        04-JAN-2022 17:34
RSX11M.STB;10       37.        04-JAN-2022 17:34

Total of 148./148. blocks in 4. files
```

You can now shut the system down normally with `RUN $SHUTUP`. Next, it is time for an IPGEN. The version of BQTCP on the Pi11/70 image is quite old, and Johnny Bilquist has recently released version 2.11. Let's grab that:

```
$ wget ftp://mim.stupi.net/bqtcp.dsk
```

If your SIMH-11 is already running you can break with Ctrl-E and attach the disk like so:

```
sim> set rl0 rl02
sim> att rl0 bqtcp.dsk
```

Otherwise, just add the same lines in your SIMH configuration file and boot. Do not writelock the disk, it is used as scratch space during IPGEN. Now, mount the distribution:

```
>MOU DL0:TCPIP
>SET DEF [1,54]
>@DL0:[IP]IPGEN
>;
>; BQTCP/IP V2.11 generation.
>;
>; Started on 04-JAN-2022 18:03:20
>;
>* What is the device where the kit is [S D:"SY:"]: DL0 
>;
>; You have not done any builds from this directory before, so
>; no update can be performed. You need to do a full build.
```

Accept defaults, except the following:

```
>* Do you want to install the DECnet driver? [Y/N D:N]: Y
>* Do you want to install the new HELP files? [Y/N D:N]: Y	.; optional, but recommended
>* Do you want to install the new message files? [Y/N D:N]: Y	.; optional, but recommended
>* Do you want to install RSX patches? [Y/N D:N]: Y		.; optional, but recommended
```

Once it completes, you'll be asked if you want to configure TCPIP. For the sake of simplicity we'll just do it now. The configuration is very simple, with some minor caveats. For convenience I have included my full IPCONFIG session below, but I will outline these separately:

* I have occasionally had issues with getting the proper NTP/DNS server addresses from DHCP, though this may be an issue with my network. I choose to set them manually.
* The default CSR/vector addresses for QNA-0 are fine as is. Similarly, leave the configuration for IF1: untouched, this is your loopback device.

```
>;
>; Check/update IP configuration...
>;
>;
>; No LB:[1,2]IPPARAM.CMD file found.
>;
>* Do you want to configure TCP/IP now? [Y/N D:N T:1M]: Y
>;
>; IP configuration X0.4.
>;
>; Started at 04-JAN-2022 18:04:52.
>;
>; This program will configure BQTCP/IP for you. It is a fairly
>; simple and stupid program. If you answer any question wrong,
>; just stop it, and start over again.
>;
>; Once it has completed, you can alwys run it again, and it will
>; use the answers from previous configurations as defaults.
>;
>; Note that for most questions that asks for an address,
>; either an IP dotted address works, or a host name.
>; For host names, you need the translation stored in the HOSTS
>; file, or defined by logical names for the configuration items
>; that are used before the network is up and running.
>; For hosts used later than that, DNS is also possible, with
>; one exception. The DNS server itself cannot be found through
>; DNS.
>;
>; If a value is presented as a default, but you want to delete
>; that value, enter a single space as the answer to the question.
>; A single space will be interpreted as a value for the parameter
>; should not be set.
>;
>; *** Section 1 - Basic configuration ***
>;
>* Size of IP pool [D R:64.-1024. D:256.]:
>* Hostname [S R:1-20 D:"WOOYOO"]:
>* Get domain name from DHCP? [Y/N D:Y]: N
>* Domain name [S R:0-20 D:"Unknown.Net"]: LAB.RSX11M.IO
>* Get DNS server address from DHCP? [Y/N D:Y]: N
>* Domain name server address [S R:0-20 D:"8.8.8.8"]: 10.5.255.53
>* Use MDNS? [Y/N D:N]:
>* Get NTP server address from DHCP? [Y/N D:Y]: N
>* NTP server address [S R:0-60 D:"pool.ntp.org"]:
>;
>; *** Section 2 - Interface configuration ***
>;
>; Configuring for 2 interfaces.
>;
>; Now configuring IF0:
>;
>; (IF0: is assumed to be an ethernet interface.)
>* ACP for IF0: [S R:0-6 D:"ETHACP"]:
>* Line for IF0: [S R:0-6 D:"QNA-0"]:
>* DHCP enabled for IF0:? [Y/N D:Y]: N
>* IP address for IF0: [S R:0-20 D:"0.0.0.0"]: 10.5.5.83
>* IP mask for IF0: [S R:0-20 D:"0.0.0.0"]: 255.255.255.0
>;
>; Now configuring IF1:
>;
>; (IF1: is assumed to be the loopback interface.)
>* ACP for IF1: [S R:0-6 D:""]:
>* IP address for IF1: [S R:0-20 D:"127.0.0.1"]:
>* IP mask for IF1: [S R:0-20 D:"255.0.0.0"]:
>;
>; *** Section 3 - Routing configuration ***
>;
>* Address of default broadcast interface [S R:0-30 D:"10.5.5.83"]: 10.5.5.255
>* Default router address [S R:0-20 D:""]: 10.5.5.254
>;
>; *** Section 4 - Server configuration ***
>;
>* Log directory [S R:0-20 D:"LB:[IPLOG]"]:
>* Enable telnet server? [Y/N D:Y]:
>* Telnet welcome message [S R:0.-80. D:"Welcome to WOOYOO, an RSX-11M-PLUS syst
em!"]:
>* Telnet terminals [D R:0.-32. D:8.]:
>* Start RWHOD? [Y/N D:N]: 
>* Start SPOOF? [Y/N D:Y]: N
>* Number of HTTPD servers [D R:0.-20. D:5.]: 
>* Number of FTPD servers [D R:0.-20. D:5.]: 
>* Number of MAIL servers [D R:0.-20. D:5.]: 0
>* SMTP relay host [S R:0.-80. D:""]: 
>* SMTP source domain name to use for outgoing mail [S R:0.-80. D:""]: 
>* Number of TCP sink servers [D R:0.-20. D:2.]: 0
>* Number of TCP echo servers [D R:0.-20. D:2.]: 0
>* Number of TCP daytime servers [D R:0.-20. D:2.]: 0
>* Number of TCP quote-of-the-day servers [D R:0.-20. D:2.]: 0
>* Number of TCP IDENT servers [D R:0.-20. D:10.]: 0
>;
>; *** Section 5 - Client configuration ***
>;
>* Install TELNET client? [Y/N D:N]: Y
>* Install IRC client? [Y/N D:N]: Y
>* Install FTP client? [Y/N D:N]: Y
>* Install NTP client? [Y/N D:N]: Y
>;
>;     If left blank, IPINS will not set the time offset.
>;     You want this if you manage the time offset with some other tool.
>* Time offset from UTC (in minutes) [S D:""]: -300
>;
>; Saving...
>;
>;
>; Done
>;
>; Remember to add, change or update the information in LB:[1,2]HOSTS.TXT
>;
>; After DECnet has started (if DECnet is also installed on the machine),
>; invokde [IP]IPINS.CMD to start TCP/IP.
>;
>; Invoke [IP]IPAPPL.CMD at a later point in the
>; startup, when all shared libraries and other requisits have been
>; installed.
>;
>; Edit [IP]IPREM.CMD to customize the shutting down procedure,
>; and remember to invoke this if needed from the standard shutup
>; procedure.
>;
>; Also check [IP]POSTIP.NEW, [IP]PREAPPL.NEW and [IP]POSTAPPL.NEW
>; for details of customization that you might want to do.
>;
>SET /CLI=TI:MCR
>SET /NONAMED
>SET /DEF=[1,54]
>@ <EOF>
```

Once you have IPGENned successfully, you will need to make a few minor changes. First, undo the changes made to `STARTUP.CMD` earlier in the guide. Once you have done that, you will need to change the `@LB:[1,2]INSPROG.CMD` line to the following, or it will not execute properly:

```
.CHAIN LB:[1,2]INSPROG.CMD
```

Once you are done, `RUN $SHUTUP` and your disk image should be ready for use on a real Qbus PDP11 with DE{LQ,QN}A!
