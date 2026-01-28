## **VNA Math**

The VNA calculates the ratio of the reflected wave to the incident wave

### **Step 1: Voltage to Complex Vector**

For both the **Reference** ($V\_{ref}$) and **Reflected** ($V\_{rl}$) channels, we calculate:

$V = I + jQ$

In the ALD5960, this is done using an I/Q measurement via an Intermediate Frequency (IF). The input is:

$V(t) = A\cos{(2\pi f_rt+\phi)}$

and is mixed with a separate frequency $f_{lo}$ to form:

$I = V(t) \times cos(2\pi f_{lo}t)$

$Q=V(t) \times \sin(2\pi f_{lo}t)$

Using some trig identities gets us:

$I=A/2\cos(2\pi f_{IF}t+\phi)$

$Q = A/2 \sin(2\pi f_{IF} + \phi)$

where $f_{IF} = f_r -f_{LO}$, known as the intermediate frequency. The

### **Step 2: Raw Reflection Coefficient**

The raw measurement ($\Gamma_{raw}$) is the complex division of these two vectors:

$\Gamma_{raw} = \frac{V_{rl}}{V_{ref}}$  

## Step 3: **The 3-Term Error Model (Calibration)**

We use **OSL (Open, Short, Load)** calibration to correct for the following three systematic errors:

1. **Directivity ($E_D$):** Leakage inside the coupler where the forward signal "bleeds" into the reflection port.  
2. **Source Match ($E_S$):** Reflections from the VNA's own port bouncing back toward the DUT.  
3. **Reflection Tracking ($E_{RT}$):** The frequency response (gain/loss/tilt) of the cables and internal traces.

### **Calibration Formula**

After measuring OSL standards, the true reflection coefficient ($\Gamma_{actual}$) is recovered from the raw measurement using:

$\Gamma_{actual} = \frac{\Gamma_{raw} - E_D}{E_{RT} + E_S(\Gamma_{raw} - E_D)}$ 

## **Derived Measurements**

Once $\Gamma_{actual}$ is calculated, we can derive standard RF metrics:

* **Return Loss (dB):** $-20 \log\_{10}(|\Gamma|)$  
* **VSWR:** $\frac{1 + |\Gamma|}{1 - |\Gamma|}$  
* **Impedance ($Z$):** $50 \cdot \frac{1 + \Gamma}{1 - \Gamma} \rightarrow (R + jX)$


