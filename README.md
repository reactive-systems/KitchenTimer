# TSL Kitchen Timer

| <img src="https://github.com/reactive-systems/KitchenTimer/blob/master/yampa/application.png" alt="Desktop Version" style="max-height:150px;"> | <img src="https://github.com/reactive-systems/KitchenTimer/blob/master/threepennygui/application.png" alt="Web Version" style="max-height=150px;"> | <img src="https://github.com/reactive-systems/KitchenTimer/blob/master/clash/application.png" alt="Hardware Version" style="max-height: 150px;"> |
|:-:|:-:|:-:|
| Desktop version </br> ([Yampa](https://wiki.haskell.org/Yampa)) | Web version </br> ([Threepenny-GUI](https://wiki.haskell.org/Threepenny-gui)) | Hardware version </br> ([CλaSH](https://clash-lang.org/)) |

We use [Temproal Stream Logic
(TSL)](https://www.react.uni-saarland.de/publications/FKPS19a.html)
for the specification of a real-world kitchen timer application.
First, a Control Flow Model (CFM) is synthesized from the
specification, which then is turned to a desktop application using the
Arrowized FRP library [Yampa](https://wiki.haskell.org/Yampa), a web
application using the Monadic
[Threepenny-GUI](https://wiki.haskell.org/Threepenny-gui) library, and
to hardware using the Applicative hardware description language
[CλaSH](https://clash-lang.org/).

More details on how the application works and how the respective code
is generated from the CFM can be found
[here](https://www.react.uni-saarland.de/publications/FKPS19b.html).

# The Kitchen Timer Specification

The specification uses
[TSL](https://www.react.uni-saarland.de/publications/FKPS19a.html) to
realize the following requirements:

1. Whenever the `MIN` and `SEC` buttons are pressed simultaneously,
  the timer is reset, meaning the time is set to zero and the system
  stays idle until the next button gets pressed.
2. If only the `MIN` button is pressed and the timer is not currently
  counting up or down, then the currently set time is increased by one
  minute.
3. If only the `SEC` button is pressed and the timer is not currently
  counting up or down, then the currently set time is increased by one
  second.
4. As long as no time greater than zero has been set and the system is
  idle: if the `START/STOP` button is pressed and the timer is not
  already counting up or down, then the timer starts counting up until
  it is stopped by any button pressed.
5. If a time has been set and the `START/STOP` button is pressed while
  the timer is not currently counting up or down, then the timer
  starts counting down until it is stopped by any button pressed.
6. The timer can only be started by pressing start.
7. The timer can always be stopped by pressing any button while
  counting up or down.
8. It is possible to start the timer and to set some time
  simultaneously.
9. The buzzer beeps on any button press and after the counter
  reaches zero while counting down.
10. The display always shows the time that is currently set.

# Installation

First, create a copy of `build.cfg_sample` and safe it to `build.cfg`:

`cp build.cfg_sample build.cfg`

The control flow model is synthesized by using `make` in the top-level
directory. The different applications are build similarly using `make`
in the respective sub-directories. The hardware binary can be uploaded
to the FPGA using `make upload`. For running the synthesis and
building steps, the following tools must be installed and the
respective paths must be set in `build.cfg`.

## Synthesis

You need [TslTools](https://github.com/reactive-systems/tsltools),
which provides `tsl2tlsf` and `cfm2code`, and an LTL synthesizer that
adheres to the rules of the LTL track of the [Reactive Synthesis
Competition (SYNTCOMP)](http://www.syntcomp.org/). We used [Strix
(v18.07)](https://strix.model.in.tum.de/), since the tool is able to
synthesize a CFM from the TSL specification within a few
seconds. However, other alternatives also are also possible, as for
example the tools [BoSy](https://github.com/reactive-systems/bosy), or
[PARTY](https://github.com/5nizza/party-elli).

## Desktop + Web

We recommend using the [Haskell Tool Stack](http://haskellstack.org/)
for building both applications. The tool automatically pulls the
required version of the Glasgow Haskell Compiler (GHC) and all
required dependencies. Using `stack`, their installation will not
interfer with any system installation.

## Hardware

The following tools are required for creating the final binary to be
upload to the hardware:

* [CλaSH Compiler](https://github.com/clash-lang/clash-compiler) (commit fff4606)
* [Yosys](https://github.com/cliffordwolf/yosys)
* [nextpnr](https://github.com/YosysHQ/nextpnr)
* [icepack](https://github.com/cliffordwolf/icestorm/tree/master/icepack)
* [iceprog](https://github.com/cliffordwolf/icestorm/tree/master/iceprog)
* [iCEDude](https://github.com/reactive-systems/icedude)

Before that, however, the respective hardware parts must be purchased
and the custom kitchen timer PCB shield needs to be created. More
detailed information on this process can be found in the [assembly
tutorial](https://github.com/reactive-systems/KitchenTimer/blob/master/clash/assembly.md).
To this end, note that you can either use an active buzzer, shipped
with a builtin wave generator that only produces a single tone, or a
passive buzzer that requires the respective output frequency to be
produced manually.  By default, we assume that an active buzzer is
used. However, if you want to use a passive buzzer instead, you also
need to add to following line to your `build.cfg`:

`CPP_FLAGS=-DPASSIVEBUZZER`
