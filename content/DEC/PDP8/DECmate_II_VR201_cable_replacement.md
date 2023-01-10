# DECmate II VR201 cable (BCC02) replacement
I have a DECmate II sans VR201, an LK201, and an [OSSC](https://videogameperfection.com/products/open-source-scan-converter/) for retrogaming. This should work to allow the use of a standard HDMI monitor with an LK201 (or Paul Koning's [LK201 emulator](https://github.com/pkoning2/lk201emu) on any upscaler/converter with VGA in.

References:
* [NetBSD: LK201 Interface](https://www.netbsd.org/docs/Hardware/Machines/DEC/lk201.html#pinout)
* [DEC Rainbow VR201 DB15 Connector Pinout](http://www.larosse.net/pc100/dec100-vr201-cable.html)

## What you will need
* [DB15-F](https://www.amazon.com/uxcell-Breakout-Connector-Solderless-Terminal/dp/B07MMN55NM) and [RJ11](https://www.amazon.com/dp/B097SPR48M) terminal blocks, or the tools to make something more permanent. The pinouts that follow worked for me with these Amazon products but you will want to verify correct pinning yourself.
* A standard VGA cable and some spare shielded wires
* A wire stripper 
* [VGPerfection OSSC](https://videogameperfection.com/products/open-source-scan-converter/) or similar converter/upscaler. Really, anything that converts VGA in to HDMI/DP/VGA out should work, but an OSSC is what I had lying around. Personally I would choose to use an upscaler unless you have a CRT or LCD natively capable of small resolutions you plan to use. The OSSC provides a larger and more readable image on HDMI displays.

## How to do it
Cut one end off the VGA cable and strip the plastic to expose the wiring. You will only need three: VGA green (should be green), its ground (copper wire typically wrapped around it), and the VGA ground wire (should be black). Verify this yourself using a multimeter to ensure you have the right wires.

Next, strip both ends off your spare shielded wires, we will need four for the RJ11 portion. The RJ11 pinout assumes you have the connector oriented as in the NetBSD document linked above, included here just in case:

```
    |----- data -< -------------------------------------------|
    |    |-------- >- Power -----------------------------|    |
    |    |    |------------- GND -------------------|    |    |
    |    |    |    |------------- <- data -----|    |    |    |
   -------------------                        -------------------
  | "    "    "    "  |                      | "    "    "    "  |
  | L    V    G    D  |                      | D    G    V    L  |
  | K    +    N    E  |                      | E    N    +    K  |
  | ->        D    C  |                      | C    D         -> |
  | D             ->  |                      | ->             D  |
  | E              L  |                      | L              E  |
  | C              K  |                      | K              C  |
   --               --                        --               --
     |             |                            |             |
      --         --                              --         --
        |       |                                  |       |
         -------                                    -------

     Looking into the                       Looking into the DECstation
     LK201 Connector                                 Connector
   (Socket on keyboard)                        (Socket on DECstation)

```

Wire everything up to the DB15-F block using the following pinout:

```
VGA green -> DB15 pin 12 (mono out)
VGA green gnd -> DB15 pin 4 (mono gnd)
VGA gnd -> DB15 pin 5 (gnd)
RJ11 pin 1 -> DB15 pin 15 (kbd rx)
RJ11 pin 2 -> DB15 pin 6 (gnd)
RJ11 pin 3 -> DB15 pin 7 (+12v)
RJ11 pin 4 -> DB15 pin 14 (kbd tx)
```

Take care not to swap pins 2 and 3 on the RJ11, or your LK201 will eat +12v and fall over dead. Verify your pinout with a multimeter before plugging the cable into your LK201. VGA goes to the OSSC, HDMI (or whatever) to your machine, and you're good to go. No more VR201 cataracting!
