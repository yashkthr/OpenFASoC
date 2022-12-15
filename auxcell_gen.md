# Phase Lock Loop (PLL) Generator

Phase Lock Loop (PLL) is a system that consists of three major parts; Phase Frequency Detector (PFD), Charge Pump and Loop Filter, and Voltage Controlled Oscillator (VCO). A PLL is highly preferred because it is a feedback system that compares the output frequency from the input frequency and can survive in a single chip. A PLL is normally used in well-timed clock generator, recovery of signal from noisy communication channel and high performance wireless with additional application in PLL’s parts

![pll](https://user-images.githubusercontent.com/69398841/207780408-b0fe4793-8616-4c49-a0b3-bf3dd9f5cb5a.png)

## Charge Pump

The charge pump circuit is connected with loop filter and located within PFD and VCO. Charge pump is functioning as a converter for the logic states of the PFD into an analog signal in order to control the VCO. The frequency of the VCO is controlled by the output signal of the charge pump circuit. The output voltage of the charge pump circuit must be held at a constant voltage, when PLL locks in some frequency. The charge pump consists of two switched current source that pump charge in or out of the
loop filter according to two logical inputs.

![CP](https://user-images.githubusercontent.com/110079763/206902465-51f3c27a-219a-4a0c-b768-72321d4f9ed4.png)

## Phase frequency detector
![PFD](https://user-images.githubusercontent.com/110079763/206902472-fdc42b56-e77f-4ea3-8196-719e90638ee0.png)

## Frequency Divider
![FDD](https://user-images.githubusercontent.com/110079763/206902481-f9512a87-907c-4027-8454-f614a1c85504.png)

## VCO
![VCO](https://user-images.githubusercontent.com/110079763/206902490-9021933d-ac0f-4eaf-bcc7-464abacefee9.png)


# MY TASK: Generating AUX CELLS for OpenFASoC Flow
Aux Cells are part of big analog design which cannot be implemented with exisiting standerd cell in library. For different designs we have different aux cells. We have to manually create these.

# AUX CELL GENERATION FOR - OpenFASoC(Fully Open-Source Autonomous SoC)
## GENERATING .lef, .gds for Aux cells
**Discription** : In Open FASoC Flow to generate a automated Analog design, few auxilaury cells(.lef,.gds) are required to be created which cannot be implemented with existing library cells (like Header and SLC in temp_sence_gen). To generate these .lef and .gds files of AUX cells we use ALIGN.

### Reduired inputs from previous step of flow:
 - SCHEMATIC and SPECIFICATION of AUX cell to be generated.
(usually AUX cell contains 6 to 12 transistors)

### First Step 

- Depending upon given SCHEMATIC and SPECIFICATION of AUX cell, a SPICE Netlist will be created with .sp file extension.

### Using ALIGN: Analog Layout, Intelligently Generated from Netlists:

**About:**

ALIGN is an open source automatic layout generator for analog circuits jointly developed under the DARPA IDEA program by the University of Minnesota, Texas A&M University, and Intel Corporation.

The goal of ALIGN (Analog Layout, Intelligently Generated from Netlists) is to automatically translate an unannotated (or partially annotated) SPICE netlist of an analog circuit to a GDSII layout. The repository also releases a set of analog circuit designs.

The ALIGN flow includes the following steps:

Circuit annotation creates a multilevel hierarchical representation of the input netlist. This representation is used to implement the circuit layout in using a hierarchical manner.
Design rule abstraction creates a compact JSON-format represetation of the design rules in a PDK. This repository provides a mock PDK based on a FinFET technology (where the parameters are based on published data). These design rules are used to guide the layout and ensure DRC-correctness.
Primitive cell generation works with primitives, i.e., blocks at the lowest level of design hierarchy, and generates their layouts. Primitives typically contain a small number of transistor structures (each of which may be implemented using multiple fins and/or fingers). A parameterized instance of a primitive is automatically translated to a GDSII layout in this step.
Placement and routing performs block assembly of the hierarchical blocks in the netlist and routes connections between these blocks, while obeying a set of analog layout constraints. At the end of this step, the translation of the input SPICE netlist to a GDSII layout is complete.

#### Installing ALIGN:
**Prerequisites**

- gcc >= 6.1.0 (For C++14 support)
- python >= 3.7 

Use the following commands to install ALIGN tool.

```
export CC=/usr/bin/gcc
export CXX=/usr/bin/g++
git clone https://github.com/ALIGN-analoglayout/ALIGN-public
cd ALIGN-public

#Create a Python virtualenv
python -m venv general
source general/bin/activate
python -m pip install pip --upgrade

# Install ALIGN as a USER
pip install -v .

# Install ALIGN as a DEVELOPER
pip install -e .

pip install setuptools wheel pybind11 scikit-build cmake ninja
pip install -v -e .[test] --no-build-isolation
pip install -v --no-build-isolation -e . --no-deps --install-option='-DBUILD_TESTING=ON'
```

## Making ALIGN Portable to Sky130 tehnology

Clone the following Repository inside ALIGN-public directory

```
git clone https://github.com/ALIGN-analoglayout/ALIGN-pdk-sky130
```

move `SKY130_PDK` folder to `/home/yash/Desktop/ALIGN-public`

## Running ALIGN TOOL

Everytime we start running tool in new terminal run following commands.

```
python -m venv general
source general/bin/activate
```
Commands to run ALIGN (goto ALIGN-public directory)


```
mkdir work
cd work
```
General syntax to give inputs
```
schematic2layout.py <NETLIST_DIR> -p <PDK_DIR>/
```

## FLOW    

![image](https://user-images.githubusercontent.com/69398841/207779636-2ee480a4-7caa-4d81-be49-8ecbddbd7cce.png)

## Generated .lef and .gds
### CP GDS 

![Screenshot from 2022-12-14 23-31-07](https://user-images.githubusercontent.com/69398841/207777789-1a96cbfa-04a7-43fc-a5d9-dfa31fade1e2.png)

![Screenshot from 2022-12-15 10-35-11](https://user-images.githubusercontent.com/69398841/207777832-e5728334-9298-43a1-afcb-abef67781fdf.png)


### CP LEF

![image](https://user-images.githubusercontent.com/69398841/207779153-b4774c2a-c861-41bc-ae7e-1fe8df562a35.png)

## Acknowledgement
  
  * Kunal Ghosh, Director, VSD Corp. Pvt. Ltd

## Contact Information

  * Yash Kothari, Post Graduate student, IIIT Bangalore, Yash.Kothari@iiitb.ac.in
  * Kunal Ghosh, Director, VSD Corp. Pvt. Ltd. kunalghosh@gmail.com

