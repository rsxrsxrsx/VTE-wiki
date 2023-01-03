# TCP/IP on MVS/ESA 5.2.2
Work in progress.
Courtesy of [Matt "racingmars" Wilson](https://github.com/racingmars).

* Copy `SYS1.SEZAINST(TCPIPROC)` to `SYS1.PROCLIB(TCPIP)`
    * Change `STEPLIB` to `SYS1.SEZATCP`
    * Change `PROFILE` to `SYS1.TCPIP(PROFILE)`
    * Change `SYSTCPD` to `SYS1.TCPIP(TCPDATA)`

* Allocate a small `SYS1.TCPIP` PDS.
* Copy `SYS1.SEZAINST(SAMPPROF)` to `SYS1.TCPIP(PROFILE)`
* Copy `SYS1.SEZAINST(TCPDATA)` to `SYS1.TCPIP(TCPDATA)`

* Edit `SYS1.TCPIP(PROFILE)` to your liking.
    * Set `DATASETPREFIX` to `SYS1`
    * Delete all the `DEVICE` and `LINK`s, add your own.
    * Comment out/remove everything in the `AUTOLOG` section. We haven't created corresponding startup procedures in `PROCLIB` yet for any of those, so they'll just fail.
    * Set `HOME` address for your new device; remove others.
    * Set `PRIMARYINTERFACE` to your new device.
    * Configure your routes appropriately.
    * At the bottom, delete all the `START` statements and add one for your device.
    * See below for more detailed network setup information.

* Edit `SYS1.TCPIP(TCPDATA)` to your liking. Most important is to set `DATASETPREFIX` to `SYS1`

Now we need to integrate TCPIP libraries with MVS. In `SYS1.PARMLIB`, edit...

**LNKLST00**:
* Add `SYS1.SEZALINK`
* Add `SYS1.SEZALNK2`

**LPALST00**:
* Add `SYS1.SEZALPA`

**IEAAPF00**:
* Add `SYS1.SEZATCP`
* Add `SYS1.SEZADSIL`
* Add `SYS1.SEZALINK`
* Add `SYS1.SEZALNK2`
* Add `SYS1.SEZALPA`
(all on volume MVSV5R)

**IEFSSN00**:
* Add `TNF,MVPTSSI`
* Add `VMCF,MVPXSSI,P390`

(P390 is the node name that needs to match your system)

**SCHED00**:
* Add `PPT PGMNAME(EZAINMAN) KEY(6) NOCANCEL PRIV NOSWAP SYST`

**IKJTSO00**:
* Add `"MVPXDISP TCPIP"` to the `AUTHCMD NAMES` list (note continuation character in column 80 of those lines)

I do all my Hercules networks with little point-to-point /30s between the Hercules guest and the host, and then add the routes to my network router to route through the Hercules host.

In `hercules.cnf`:
```
0E20.2 CTCI /dev/net/tun 1500 192.168.59.222 192.168.59.221 255.255.255.252
```

And the relevant TCPIP profile entries:
```
; Hardware definitions:
;

DEVICE CTCA1 CTC E20
LINK CTC1 CTC 1 CTCA1

HOME
   192.168.59.222 CTC1

GATEWAY
 192.168.59.221 = CTC1 1492 HOST
 defaultnet 192.168.59.221 CTC1 1492 0

START CTCA1
```

Re-IPL, CLPA, and see if `S TCPIP` gets you going after everything else has started. You should be able to ping the system and it should be able to ping IP addresses from TSO. But not much else yet.

