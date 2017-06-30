ASGtools
--------

ASGtools is a tool suite containing all ASGtools.

### Contents ###

The following ASGtools are included in this package:

* ASGbreezegui v1.1
* ASGconfigGen v1.0
* ASGdelaymatch v1.3
* ASGdrivestrength v1.0
* ASGlogic v0.2
* ASGresyn v1.2
* ASGtechMngr v1.0
* DesiJ: A Tool for STG Decomposition v3.1.1

It also includes the following third party tools:

* Balsa Asynchronous Synthesis System v4.0
* Espresso v2.3
* MPSAT v4.41beta
* PCOMP v2.51beta
* Petrify
* PUNF v8.51

### Installation ###

Due to the operating system limitation of some tools, this bundle is only available for UNIX-based systems (on UNIX with 64-bit architecture, 32-bit support is required). You may need the following additional packages:

* The multiprecision arithmetic library libgmp3c2 (32bit version). For Debian-based Linux distributions [this](http://www.ubuntuupdates.org/package/core/precise/universe/base/libgmp3c2) version should work.
* The Scheme interpreter Guile 1.x (2.x won't work). For Debian-based Linux distributions execute `sudo apt-get install guile-1.8`
* Graphviz (http://www.graphviz.org/)
* Ghostview (http://pages.cs.wisc.edu/~ghost/)

Download and unpack the ASGtools package. All external tools needed for operation are included in the package (except for the optional tool petreset, because licensing is unclear). You don't have to install anything or make changes to environment variables (except for the libraries mentioned above). To run it you will need a Java runtime environment (JRE) v1.8 (or later).

### Usage ###

For the following example commands it is assumed that your current working directory is the ASGtools main directory. If you want run the tools from another directory you have to add the path to the ASGtools main directory in front of the following commands (or you could add the `bin/` directory to your `PATH` variable).

For usage instructions for the tools please consult the respective README's.

#### Configuration ####

1. Run `bin/ASGtechmngr_gui` to create or import technologies.
2. Run `bin/ASGconfiggen_gui` to create config files for the tools.

#### Design Flow ####

This will only address the basic design flow. For further configuration and options, please consult the respective README's of the tools.

##### 0) Balsa to Breeze #####

The design flow is intended to start with a breeze file, however the Balsa compiler is also included in the bundle to create a breeze file from a balsa file:

	tools/balsa/balsa-c.sh -o . benchmark.balsa
	
This creates the file `benchmark.breeze` containing the breeze netlist. You can visualise this netlist with

	bin/ASGbreezeGui benchmark.breeze 

##### 1a) Balsa synthesis #####

To synthesise a breeze netlist with the original Balsa implementation tool execute

	tools/balsa/balsa-netlist.sh -X techname -o balsaImpl.v benchmark.breeze
	
This creates the file `balsaImpl.v` containing a verilog netlist implementing the breeze netlist `benchmark.breeze` in the technology `techname` (It creates also some other files, but they are not used in later design steps).

##### 1b) Resynthesis #####

To synthesise a breeze netlist with resynthesis implementing optimisations execute

	bin/ASGresyn -tc D -sout resynImpl.v -stgout benchmark.g -lib tech/techname.xml benchmark.breeze
	 
This creates the file `balsaImpl.v` containing an optimised verilog netlist implementing the breeze netlist `benchmark.breeze` in the technology `techname`. It will also create the file `benchmark.g` which contains the Balsa-STG of the benchmark. You can visualise this STG with

	bin/DesiJ_gui benchmark.g  

ASGresyn will only optimise the control path of the circuit. If you have configured the remote tools in the Configuration step you want to add `-odp` to the options list to optimise the data path. 

##### 2) Transistor sizing #####

The synthesis tools (Balsa netlist and ASGresyn) don't consider the sizing of the implementing gates in their netlists. To take care of this topic execute

	bin/ASGdrivestrength -lib tech/techname_liberty.lib -cellInfoJson tech/techname_addInfo.json -out balsaImplDs.v balsaImpl.v
	
for the Balsa synthesis result or with the options `-out resynImplDs.v resynImpl.v` for resynthesis. It will create the file `balsaImplDs.v` (`resynImplDs.v`) containing the netlists with optimised transistor sizing. If you use data path optimisation during resynthesis, ASGdrivestrength might fail because it can not handle some gates used.

##### 3) Delay matching #####

Because a bundled data implementation is used, proper matched delays have to be ensured. For this step a remote configuration is mandatory. For the Balsa implementation execute

	bin/ASGdelaymatch -lib tech/techname.xml -p config/balsaprofile.xml -out balsaImplDsDm.v balsaImplDs.v
	
For the resynthesis implementation execute

	bin/ASGdelaymatch -lib tech/techname.xml -p config/resynprofile.xml -past benchmark.g -out resynImplDsDm.v resynImplDs.v

These commands will generate `balsaImplDsDm.v` and `resynImplDsDm.v`, respectively. With the option `-sdfOut` you can generate an SDF file for simulation.

### Build instructions ###

To build ASGtools, Apache Maven v3 (or later) and the Java Development Kit (JDK) v1.8 (or later) are required.

1. Build [DesiJ](https://github.com/hpiasg/desij)
2. Build [ASGbreezeGui](https://github.com/hpiasg/asgbreezegui)
3. Build [ASGlogic](https://github.com/hpiasg/asglogic)
4. Build [ASGresyn](https://github.com/hpiasg/asgresyn)
5. Build [ASGdelaymatch](https://github.com/hpiasg/asgdelaymatch)
6. Build [ASGdrivestregth](https://github.com/hpiasg/asgdrivestrength)
7. Build [ASGtechMngr](https://github.com/hpiasg/asgtechmngr)
8. Build [ASGconfigGen](https://github.com/hpiasg/asgconfiggen)
9. Execute `mvn clean install -DskipTests`