# OpenFASoC
OpenFASoC: Fully Open-Source Autonomous SoC Synthesis using Customizable Cell-Based Synthesizable Analog Circuits
The FASoC Program is focused on developing a complete system-on-chip (SoC) synthesis tool from user specification to GDSII with fully open-sourced tools. 
See more about FaSoC at https://fasoc.engin.umich.edu/
also at https://openfasoc.readthedocs.io/en/latest/index.html

# Installation
## 1. OpenFASOC:
The command used to install OpenFASOC are 
```
git clone https://github.com/idea-fasoc/openfasoc

cd openfasoc

pip install -r requirements.txt
```
For the complete steps of installing OpenFASOC, refer Manual Installation from [here](https://github.com/idea-fasoc/OpenFASOC/blob/main/docs/source/getting-started.rst). 

## 2. OpenROAD: 
OpenROAD is an integrated chip physical design tool that takes a design from synthesized Verilog to routed layout. OpenROAD uses the OpenDB database and OpenSTA for static timing analysis. Documentation is also available [here](https://openroad.readthedocs.io/en/latest/main/README.html).


The commands to install OpenROAD are,
```
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD.git

cd OpenROAD

./etc/DependencyInstaller.sh

./etc/DependencyInstaller.sh -run

./etc/DependencyInstaller.sh -dev

mkdir build

cd build

cmake ..

make

```

## 3. Klayout
Downlaod the latest version of the Klayout from [here](https://www.klayout.de/build.html). Install the following dependencies: qt5-default, qttools5-dev, libqt5xmlpatterns5-dev, qtmultimedia5-dev, libqt5multimediawidgets5 and libqt5svg5-dev.
```
sudo apt-get install -y libqt5widgets5

sudo dpkg -i klayout_0.27.11-1_amd64.deb

```

## 4. Netgen
To install Netgen, 
```
sudo add-apt-repository ppa:ngsolve/ngsolve

sudo apt-get update

sudo apt-get install ngsolve

```

## 5. Yosys
The software used to run gate level synthesis is Yosys. Yosys is a framework for Verilog RTL synthesis. It currently has extensive Verilog-2005 support and provides a basic set of synthesis algorithms for various application domains. Yosys can be adapted to perform any synthesis job by combining the existing passes (algorithms) using synthesis scripts and adding additional passes as needed by extending the Yosys C++ code base.


To install yosys, install the prerequisites using the following command 
 ```
 sudo apt-get install build-essential clang bison flex \
	libreadline-dev gawk tcl-dev libffi-dev git \
	graphviz xdot pkg-config python3 libboost-system-dev \
	libboost-python-dev libboost-filesystem-dev zlib1g-dev
```
To install latest Version of Yosys, 
```
git clone https://github.com/YosysHQ/yosys.git

make

sudo make install 

make test

```

## 6. Magic
Run following commands one by one to fulfill the system requirement.
#### Prerequisites for magic
```
sudo apt-get install m4

sudo apt-get install tcsh

sudo apt-get install csh

sudo apt-get install libx11-dev

sudo apt-get install tcl-dev tk-dev

sudo apt-get install libcairo2-dev

sudo apt-get install mesa-common-dev libglu1-mesa-dev

sudo apt-get install libncurses-dev
```
#### Install magic
```
git clone https://github.com/RTimothyEdwards/magic

cd magic/

./configure

sudo make

sudo make install
```
type `magic` terminal to check whether it installed succesfully or not. Type `exit` to exit magic.


# Temperature Sensor Generator Circuit
This generator creates a compact mixed-signal temperature sensor based on the topology from this [paper](https://ieeexplore.ieee.org/document/9816083).

It consists of a ring oscillator whose frequency is controlled by the voltage drop over a MOSFET operating in subthreshold regime, where its dependency on temperature is exponential.

![tempsense_ckt](https://user-images.githubusercontent.com/110079631/199317479-67f157c5-6934-470b-8552-5451b1361b9c.png)

  Block diagram of the temperature sensor’s circuit

The physical implementation of the analog blocks in the circuit is done using two manually designed standard cells:
1. HEADER cell, containing the transistors in subthreshold operation;
2. SLC cell, containing the Split-Control Level Converter.

The gds and lef files of HEADER and SLC cells are pre-created before the start of the Generator flow.

# OpenFASOC Flow
![flow](https://user-images.githubusercontent.com/69398841/200112750-d7543e4f-1eaa-42c7-b1bb-14db4e9eaa33.png)

The generator must first parse the user’s requirements into a high-level circuit description or verilog. User input parsing is implemented by reading from a JSON spec file directly in the temp-sense-gen repository. The JSON allows for specifying power, area, maximum error (temperature result accuracy),
an optimization option (to choose which option to prioritize), and an operating temperature range (minimum and maximum operating temperature values).
The operating temperature range and optimization must be specified, but other items can be left blank. 


The generator uses this model file to automatically determine the number of headers and inverters, among other necessary modifications that can be made to meet spec. The generator references the model file in an iterative process until either meeting spec or failing. A verilog description is then
produced by substituting specifics into several template verilog files.

The steps from the RTL-to-GDS flow look like this, usual in a digital flow:

![1tempsense_digflow_diagram](https://user-images.githubusercontent.com/69398841/207781519-f85d14ef-7ee0-4759-a145-c047b6a892b1.png)

![2tempsense_flow_diagram](https://user-images.githubusercontent.com/69398841/207781535-15118d6c-d1df-46ce-b109-81205bad1a60.png)

```   
temp-sense-gen
├── blocks
└── flow
    └── design
        ├── sky130hd
        │   └── tempsense
        │       ├── config.mk             <--
        │       └── constraint.sdc
        └── src
            └── tempsense
                ├── counter.v             <--
                ├── TEMP_ANALOG_hv.nl.v   <--
                ├── TEMP_ANALOG_lv.nl.v   <--
                ├── TEMP_AUTO_def.v       <--
                └── tempsenseInst_error.v <--
```    
The default circuit’s physical design generation can be divided into three parts:

 1. Verilog generation     

 2. RTL-to-GDS flow (OpenROAD)     

 3. Post-layout verification (DRC and LVS)     

### Verilog generation   

Running make sky130hd_temp (temp for “temperature sensor”) executes the temp-sense-gen.py script from temp-sense-gen/tools/. This file takes the input specifications from test.json and outputs Verilog files containing the description of the circuit.

The generator starts from a Verilog template of the temperature sensor circuit, located in temp-sense-gen/src/. The .v template files have lines marked with @@, which are replaced according to the specifications.  

templetes -

1.TEMP_ANALOG_hv.v      
2.TEMP_ANALOG_lv.v     
3.counter_generic.v     

using this generated files-     

1.TEMP_ANALOG_hv.nl.v    
2.TEMP_ANALOG_lv.nl.v    
3.counter.v    

 #### what are hv lv file do-   
 
   they are part of temperature sensor circuits.   
   
   #### HV FILE   

![image](https://user-images.githubusercontent.com/110079729/199910810-4962f9ed-95e8-4857-ae3f-8acf9d9fe634.png)

User Specs    

Temperature sensing range: -20⁰C – 125⁰C    

Frequency range of operation: 100Hz – 10MHz    

* Inputs

1. CLK_REF: System clock taken from input.

2. RESET_COUNTERn: Input signal to reset the module initial state.

3. SEL_CONV_TIME: Four bit input used to select how many times the 1 bit of the output DOUT is fractionated (0-16).

* Outputs

1. DOUT: The output voltage whose frequency is dependent on temperature.

2. DONE: Validity signal for DOUT

Theses are the verilog template files which are used for the creation of netlist verilog files.    

We use an all-digital temperature sensor architecture, that relies on a new subthreshold oscillator (achieved using the auxiliary cell “Header Cell“) for realizing synthesizable thermal sensors. We choose frequency as the temperature dependent variable. So, we use a ring oscillators that is based on inverters only and stacked native IO devices for better line sensitivity.    
Since the subthreshold current has an exponential dependency on the temperature, the frequency generated from the subthreshold ring oscillator is also dependent on temperature. Using this principle, we can sense the temperature by comparing the clock generated from a reference oscillator and the clock frequency from our proposed frequency generator.
