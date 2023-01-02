# RSX11M UIC notes
From [VCFED](https://forum.vcfed.org/index.php?threads/discovering-rsx11.80035/#post-974051):

```
[1,1] system libraries
[1,2] help files and system wide command files
[1,3] directory where lost files are placed
[1,4] system log directory
[1,6] print and batch queue directory
[1,54] system image, drivers and tasks dependent on current system
[2,54] baseline system
[3,54] system tasks and tools
[6,54] standalone brusys
[11,*] kernel, drivers
[12,*] MCR and related tools
[13,*] MOU/DMO and some more file system control stuff
[14,*] RMD
[15,*] Executive utilities
[16,*] Multi-user tools
[17,*]
[20,*] File related tools
[21,*]
[22,*]
[23,*] DCL
[24,*] TDX
[25,*] QMG
[26,*] BPR
[27,*] CON,HRC
.
.
.

[*,10] source files for parts of the system
[*,24] object files
[*,34] list files

[200,200] SYSGEN
```

>Group 5 is usually used for DECnet related things. Group 4, 7 and 10 are usually not used by anything. I usually create privileged users in group 10, and use directory [4,54] to add local system tools so I don't have to pollute [3,54] with that. And then add unprivileged users in group 201 and onwards. But of course, now we're really talking about local conventions...
