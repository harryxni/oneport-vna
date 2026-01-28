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

GOAL: Develop a system-level simulation using idealized components to return $\Gamma$ and $S_{11}$. 

[...]

#### Step 2: Moving towards imperfect directional couplers

GOAL: Introduce realistic imnperfection into the model, such as finite directivity and source match, using a realistic coupler (ie a Return loss bridge) to model effects of error in measurement. 

[...]

#### Step 3: Schematic Integration
GOAL: Integrate the ADL5960, ADC, RF source and clocking (could possibly be off-board), and supporting circuitry into a complete schematic.

[...]

#### Step 4: Layout
GOAL: Route and place the PCB. 
[...]

#### Step 5: Firmware
GOAL: Implement ADC capture, vector extraction and calibration routines. Generate results. 

