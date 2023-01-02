# SNA NJE Tutorial
Courtesy of HackerSmacker.

## Step 1: Setting up VTAM
Common to all of these:
```
NJEAPPLS  VBUILD TYPE=APPL
```

For RSCS on VM:
```
xxxxxxxx  APPL ACBNAME=yyyyyyyy,DLOGMOD=RSCSNJE0,MODETAB=RSCSTAB,      -
               AUTH=(ACQ),AUTHEXIT=YES,                                -
               VPACING=3
```

For JES2 on MVS:
```
xxxxxxxx APPL  EAS=1,ACBNAME=JES2,AUTH=(ACQ,PASS)
```

For POWER on VSE:
```
xxxxxxxx APPL  AUTH=(PASS,ACQ),VPACING=3,MODETAB=VTMLOGTB,DLOGMOD=PNET
```

## Step 2: Starting RSCS on that APPL
```
SMSG RSCS NETWORK START APPLID yyyyyyyy
```

## Step 3: Defining a link on RSCS
```
SMSG RSCS DEFINE nodename TYPE SNANJE LUNAME xxxxxxxx LOGMODE RSCSNJE0
```
Note: `nodename` is the remote machine's RSCS local node name. `LUNAME` is the partner LU, and `LOGMODE` has to be `RSCSNJE0` for RSCS to RSCS connections.
