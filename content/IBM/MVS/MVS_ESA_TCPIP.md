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

### Sample TCPIP PROFILE entry
```
;
; PROFILE.TCPIP
; =============
;

ACBPOOLSIZE                 1000
ADDRESSTRANSLATIONPOOLSIZE  1500
CCBPOOLSIZE                 150
DATABUFFERPOOLSIZE          160  16384
ENVELOPEPOOLSIZE            750
IPROUTEPOOLSIZE             300
LARGEENVELOPEPOOLSIZE       50    8192
RCBPOOLSIZE                 50
SCBPOOLSIZE                 256
SKCBPOOLSIZE                256
SMALLDATABUFFERPOOLSIZE     1200
TCBPOOLSIZE                 256
TINYDATABUFFERPOOLSIZE      500
UCBPOOLSIZE                 100

;
; -----------------------------------------------------------------------
;
; Inform the following users of serious errors.
;

INFORM
    OPERATOR P390
ENDINFORM

;
; Obey the following users for restricted commands.
;

OBEY
    OPERATOR TCPMAINT SNMPD SNMPQE ROUTED NCPROUT
ENDOBEY

;
; -----------------------------------------------------------------------
;
; Flush the ARP tables every 5 minutes.
;

ARPAGE 5

;
; -----------------------------------------------------------------------
;
; The SYSCONTACT and SYSLOCATION statements are used for SNMP.
;
; SYSCONTACT is the contact person for this managed node and how to
; contact this person.  Used for MVS agent MIB variable "sysContact".
;

SYSCONTACT
   MAIN OPERATOR (555-1234)
ENDSYSCONTACT

;
; SYSLOCATION is the physical location of this node.  Used for MVS
; agent MIB variable "sysLocation".
;

SYSLOCATION
   FIRST FLOOR COMPUTER ROOM
ENDSYSLOCATION

;
; You can specify DATASETPREFIX in the PROFILE.TCPIP and
; TCPIP.DATA data sets.  The character string specified as a
; parameter on DATASETPREFIX takes precedence over both the distributed
; or modified data set prefix name as changed by the EZAPPRFX
; installation job.  If this statement is used in a profile or
; configuration data set that is allocated to a client or a server, then
; that client or server dynamically allocates additional required data
; sets using the value specified for DATASETPREFIX as the data set name
; prefix.  The DATASETPREFIX parameter can be up to 26 characters long,
; and the parameter must NOT end with a period.
;
; For more information please see "Understanding TCP/IP Data Set
; Names" in the Customization and Administration Guide.
;
DATASETPREFIX SYS1
;
; -----------------------------------------------------------------------
;
; Set Telnet time-out to 10 minutes.
;

INTERNALCLIENTPARMS TIMEMARK 600 ENDINTERNALCLIENTPARMS

;
; -----------------------------------------------------------------------
;
; AUTOLOG the following servers.
;

AUTOLOG
 ;  FTPSERVE    ; FTP Server
 ;  LPSERVE     ; LPD Server
 ;  NAMESRV     ; Domain Name Server
 ;  NCPROUT     ; NCPROUTE Server
 ;  PORTMAP     ; Portmap Server
 ;  ROUTED      ; RouteD Server
 ;  RXSERVE     ; Remote Execution Server
 ;  SMTP        ; SMTP Server
 ;  SNMPD       ; SNMP Agent Server
 ;  SNMPQE      ; SNMP Client
 ;  TCPIPX25    ; X25 Server
 ;  MVSNFS      ; Network File System Server
ENDAUTOLOG

;
; -----------------------------------------------------------------------
;
; Reserve ports for the following servers.
;
; NOTES:
;
;    A port that is not reserved in this list can be used by any user.
;    If you have TCP/IP hosts in your network that reserve ports
;    in the range 1-1023 for privileged applications, you should
;    reserve them here to prevent users from using them.
;
;    The port values below are from RFC 1060, "Assigned Numbers."
;

PORT
     7 UDP MISCSERV            ; Miscellaneous Server
     7 TCP MISCSERV
     9 UDP MISCSERV
     9 TCP MISCSERV
    19 UDP MISCSERV
    19 TCP MISCSERV
    20 TCP FTPSERVE  NOAUTOLOG ; FTP Server
    21 TCP FTPSERVE            ; FTP Server
    23 TCP INTCLIEN            ; Telnet Server
    25 TCP SMTP                ; SMTP Server
    53 TCP NAMESRV             ; Domain Name Server
    53 UDP NAMESRV             ; Domain Name Server
   111 TCP PORTMAP             ; Portmap Server
   111 UDP PORTMAP             ; Portmap Server
   135 UDP LLBD                ; NCS Location Broker
   161 UDP SNMPD               ; SNMP Agent
   162 UDP SNMPQE              ; SNMP Query Engine
   512 TCP RXSERVE             ; Remote Execution Server
   514 TCP RXSERVE             ; Remote Execution Server
   515 TCP LPSERVE             ; LPD Server
   520 UDP ROUTED              ; RouteD Server
   580 UDP NCPROUT             ; NCPROUTE Server
   750 TCP MVSKERB             ; Kerberos
   750 UDP MVSKERB             ; Kerberos
   751 TCP ADM@SRV             ; Kerberos Admin Server
   751 UDP ADM@SRV             ; Kerberos Admin Server
  2049 UDP MVSNFS              ; NFS Server
  3000 TCP CICSTCP             ; CICS Socket

;
; -----------------------------------------------------------------------
;
; Hardware definitions:
;

DEVICE CTCA1 CTC E20
LINK CTC1 CTC 1 CTCA1

;
; -----------------------------------------------------------------------
;
; HOME internet (IP) addresses of each link in the host.
;
; NOTE:
;
;    The IP addresses for the links of an Offload box are specified in
;    the LINK statements themselves, and should not be in the HOME list.
;

HOME
   192.168.59.222 CTC1

;
; ---------------------------------------------------------------------
;
; The new PRIMARYINTERFACE statement is used to specify which interface
; is the primary interface.  This is required for specifying an Offload
; box as being the primary interface, since the Offload box's links
; cannot appear in the HOME statement.
;
; A link of any type, not just an Offload box, can be specified in the
; PRIMARYINTERFACE statement.  If PRIMARYINTERFACE is not specified,
; then the first link in the HOME statement is the primary interface,
; as usual.
;

PRIMARYINTERFACE CTC1

;
; -----------------------------------------------------------------------
;
; IP routing information for the host.  All static IP routes should
; be added here.
;

GATEWAY
 192.168.59.221 = CTC1 1492 HOST
 defaultnet 192.168.59.221 CTC1 1492 0


;
; -----------------------------------------------------------------------
;
; Use TRANSLATE to specify the hardware address of a specific IP
; address.  See the Customization and Administration Guide for more
; information.
;

TRANSLATE

;
; -----------------------------------------------------------------------
;
; Turn off all tracing.  If tracing is to be used, comment out the
; NOTRACE command and insert the TRACE statements here.

NOTRACE
; TRACE     <trace_parameter>
; MORETRACE <trace_parameter>
SCREEN
;
; -----------------------------------------------------------------------
; Use ASSORTEDPARMS NOFWD to prevent the forwarding of IP packets
; between different networks.  If NOFWD is not specified IP packets
; will be forwarded between networks.

ASSORTEDPARMS
  NOFWD
ENDASSORTEDPARMS

;
; -----------------------------------------------------------------------
;
; Define the VTAM parameters required for the Telnet server.
;

BEGINVTAM
    ; Define logon mode tables to be the defaults shipped with the latest
    ; level of VTAM
  3278-3-E NSX32703 ; 32 line screen - default of NSX32702 is 24 line screen
  3279-3-E NSX32703 ; 32 line screen - default of NSX32702 is 24 line screen
  3278-4-E NSX32704 ; 48 line screen - default of NSX32702 is 24 line screen
  3279-4-E NSX32704 ; 48 line screen - default of NSX32702 is 24 line screen
  3278-5-E NSX32705 ; 132 column screen - default of NSX32702 is 80 columns
  3279-5-E NSX32705 ; 132 column screen - default of NSX32702 is 80 columns
    ; Define the LUs to be used for general users.
  DEFAULTLUS
      TCP00001 TCP00002 TCP00003 TCP00004 TCP00005
      TCP00006 TCP00007 TCP00008 TCP00009 TCP00010
      TCP00011 TCP00012 TCP00013 TCP00014 TCP00015
      TCP00016 TCP00017 TCP00018 TCP00019 TCP00020
      TCP00021 TCP00022 TCP00023 TCP00024 TCP00025
      TCP00026 TCP00027 TCP00028 TCP00029 TCP00030
  ENDDEFAULTLUS
; LUSESSIONPEND    ; On termination of a Telnet server connection,
                   ; the user will revert to the DEFAULTAPPL
; DEFAULTAPPL SAMON ; Set the default application for all Telnet sessions.
  LINEMODEAPPL TSO ; Send all line-mode terminals directly to TSO.
  ALLOWAPPL TSO* DISCONNECTABLE ; Allow all users access to TSO applications
              ; TSO is multiple applications all beginning with TSO, so use
              ; the * to get them all.  If a session is closed, disconnect
              ; the user rather than log off the user.
; RESTRICTAPPL IMS ; Only 3 users can use IMS.
;   USER USER1     ; Allow user1 access.
;     LU TCPIMS01  ; Assign USER1 LU TCPIMS01.
;   USER USER2     ; Allow user2 access from the default LU pool.
;   USER USER3     ; Allow user3 access from 3 Telnet sessions,
;                  ; each with a different reserved LU.
;     LU TCPIMS31 LU TCPIMS32 LU TCPIMS33
  ALLOWAPPL *      ; Allow all applications that have not been
                   ; previously specified to be accessed.

;   Map Telnet sessions from this node to display USSAPC screen.
;   USSTAB USSAPC 130.50.10.1
;
;   Map Telnet sessions from this link to display USSCBA screen.
;   USSTAB USSCBA SNA1

ENDVTAM

;
; -----------------------------------------------------------------------
;
; Start all the defined devices.
;

START CTCA1
```
