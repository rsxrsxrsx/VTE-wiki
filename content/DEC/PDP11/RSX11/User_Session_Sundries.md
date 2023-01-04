# How to improve your time in an RSX11 system
This is a collection of useful commands and procedures for working on and administering RSX11 systems. Assumes at least some working knowledge of the environment.

## Configuring your session at login
If you have commands you want to run at login, they can be placed in `login.cmd` in your UFD:

```
>type du1:[hush]login.cmd
set term/vt100
show users
show time
```

## Unix-style line editing
This is supported by the [CLEACD](http://mim.stupi.net/rpmpkg?PKG=CLEACD) software available in Johnny Bilquist's [RSX Package Manager](http://mim.stupi.net/rpm). I highly recommend setting up RPM as it makes managing software trivial.

ACD supports EDT or EMACS style line editing. To configure it, you can place either of the following in your `login.cmd`:

```
acd link ti: to number cle$edt
acd link ti: to number cle$emacs
```

You should now be able to navigate MCR/DCL sessions using arrow keys and other such niceties. Check out `help acd` for more information.

## Increasing secondary pool space
On busier systems with plenty of memory (my 11/83 has a full 4MB of PMI RAM), you may wish to allocate additional secondary pool space. The reasoning behind doing so is outside the scope of this document; you are highly encouraged to read "Chapter 8: Memory Management" in the [RSX-11M-PLUS and Micro/RSX System Management Guide (pp.304)](http://bitsavers.org/pdf/dec/pdp11/rsx11m_plus/RSX11Mplus_V4.x/7/AA-JS14A-TC_RSX-11M_PLUS_V4.0_RSX-11M-PLUS_and_MicroRSX_System_Management_Guide_Sep87.pdf) manual. 

Expanding the secondary pool temporarily can be done in a few different ways, but doing so permanently requires a sysgen and VMR. First, temporarily; `LOAD /EXP=SEC /SIZE=n` can be used to increase the size of a given pool (in this example, the secondary pool as specified by `/EXP=SEC`) by `n` words:

```
>LOAD /EXP=SEC /SIZE=100
```

You can also expand the size of the secondary pool at boot in the file `LB:[1,2]SYSPARAM.DAT`, using the `SECONDARY_POOL=n` statement. This will increase the size of the pool by `n` 32-word blocks.

Finally, the recommended method of increasing secondary pool is via sysgen and VMR. First, you will need to generate a new system image:

```
>set def [200,200]
>@sysgen
```

Do not boot the new image yet. Create a new one with PIP, and make sure the `/BL:` flag does not exceed your amount of available system memory or things will break in strange and terrifying ways:

```
>set def [1,54]
>PIP RSX11M.SYS/CO/NV/BL:1026.=RSX11M.TSK
```

Next, edit `SYSVMR.CMD` and find the `SET /PAR=SECPOL` line- this is where your system's secondary pool is defined. Increase it by a few hundred, re-VMR, and boot:

```
> vmr @sysvmr
[...]
POOL=1200:13640.:13640.:2004
>boo [1,54]
XDT: 87  

XDT>g
RSX-11M-PLUS V4.6   BL87  


>
 sav /wb/mou="/acp=unique/lru=14/win=30"
[...]
>set /secpol
SECPOL=811.:1024.:79%
```
