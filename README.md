# **One Port VNA**

 My attempt at designing a one port VNA and to learn some RF fundamentals and technqiues. The goal here is to build a simple working S11 (reflection) VNA with minimal RF complexity.

## Architecture

The design will be based around the ADL 5960, a VNA frontend that should simplify many design considerations by black-boxing a few components of typical VNAs. Besides the ADL 5960, we will require an ADC stage, digital processing, and an RF source and a clock source.

## Repo Structure

- kicad/ Schematic and PCB design files

- docs/ Notes on design and theory, block diagrams, data sheets and user guides

- firmware/ ADC, FPGA code etc.

- simulations/ SPICE, RF simulations

- fabrication/ Fabrication outputs by kicad

- scripts/ Data processing and plotting

## Results

Results will be posted here as I progress. 

#### Step 1: Simulating Idealized Components

GOAL: Develop a system-level simulation using idealized components to return $\Gamma$ and $S_{11}$. It will consist of a voltage source (V_1), a source impedance ($R_{\text{src}}$) and a DUT ($R_{\text{load}}$).

![](simulation/oneport-vna_part1.png) 

The schematic and ngspice directive is shown above. 

A few notes on this...KiCAD has some quirks when running spice simulations (coming from xschem). Specifically, running plotting commands within .control does not work since KiCAD uses its own plotting software. This creates an issue since defining vectors and variables within the KiCAD simulator and plotter is very clunky. Here, I remedy this by running a single point in the .ac analysis and just printing $\Gamma$, since I don't expect any change at different frequencies. In the future, I'll have to use wrdata to export data and use python to plot. Also, .step does not work in KiCAD, so I had to use a foreach loop which edits the resistance of R_DUT, rather than setting a param and stepping through.

Results are below with some of the spice output cleaned up. They are as expected...

```
Resistance: 0 ohms | Gamma: -1
Resistance: 50 ohms | Gamma: 0
Resistance: 100 ohms | Gamma: 0.333333
```



#### Step 2: A more realistic simulation

GOAL: Introduce realistic imperfection into the model, such as finite directivity and source match, using a realistic coupler (ie a Return loss bridge) to model effects of error in measurement. 

[...]

#### Step 3: Schematic Integration

GOAL: Integrate the ADL5960, ADC, RF source and clocking (could possibly be off-board), and supporting circuitry into a complete schematic.

[...]

#### Step 4: Layout

GOAL: Route and place the PCB. 
[...]

#### Step 5: Firmware

GOAL: Implement ADC capture, vector extraction and calibration routines. Generate results. 
