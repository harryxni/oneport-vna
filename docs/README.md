# **VNA Math**

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

# **Filter Bank, IF selection**

## Stimulus and Harmonic Content

We can generally assume an RF source wave that's roughly square, which is primarily constructed of odd-order harmonics:

$x(t) \propto \sin(\omega t) + \frac{1}{3} \sin(3\omega t) + \frac{1}{5}\sin(5 \omega t) + \dots$ 

To create an ideal sine wave source, we can use a low-pass filter to filter out these harmonics. However, because a VNA operates over a wide range of frequencies, we must select different filters to operate over different bands within the full frequency range, known as the **Source Filter Bank**.

## Frequency Planning

An additional part of the filter selection criteria is consideration of the Local Oscillator (LO) frequency and the Intermediate Frequency (IF), related by:

$f_{IF} = |f_{RF} - f_{LO\_eff}|$

Generally, we pick $f_{IF}$ to be constant (e.g., $100\text{ kHz}$ or $1\text{ MHz}$) and have $f_{LO}$ track the stimulus $f_{RF}$. We typically use **High-Side Injection** ($f_{LO} > f_{RF}$) to ensure the phase polarity remains consistent throughout the sweep.

## Harmonic Mixing and Aliasing

The issue with square-ish waves is **Harmonic Mixing**. Even if the fundamental produces the desired $f_{IF}$, the harmonics in the stimulus and LO interact:

$f_{IF\_harmonic} = |m \cdot f_{RF} - n \cdot f_{LO\_eff}|$

For example, the 3rd harmonics produce a "ghost" signal at $f_{IF}^3 = |3 f_{RF} - 3f_{LO}|$, which is exactly $3 \times f_{IF}$. While the LPF on the IF output can easily remove this $3\text{ MHz}$ signal, the real danger is **Aliasing**.

In real hardware, duty cycle imperfections (signals not being exactly 50/50) create even-order harmonics ($2f, 4f$). If these are present in the LO, a harmonic like $2f_{LO}$ can mix with a stimulus harmonic like $2f_{RF}$ or $3f_{RF}$ to create products that land exactly near$f_{IF}$.

One method used to combat this duty cycle asymmetry is using the LO divider, which is offered on the ALD5960. I'm not entirely clear on why this is, but diving the LO input suppresses the even harmonics. 

### Example:

Let's work through a VNA system to illustrate this. We want a VNA that operates from 100 MHz to 10 GHz for example. We select an IF of $f_{IF}= 1 $ MHz arbitrarily but can be easily digitized by ADCs and processed by an FPGA. 

Let's take a look at the low end 100 MHz measurement point. We will need to set a target $f_{LO}=101$ MHz. We can achieve this easily using the internal frequency divider of $N=4$, which means our $f_{LO}^{\text{Injected}} = 404$ MHz.

 We can also filter out the stimulus frequency harmonics. Our primary concern is the 3rd harmonic of 300 MHz, which we can cut off with a LPF. 

The choice of LPF cutoff frequency $f_c$ is dependent on the order of filter (which defines the degree of attenuation at a given frequency) and this decision then directly decides the band range $f_{max}$, where the band goes from $100 \text{ MHz}-f_{max}$.

The Gain of a filter:

$G = \frac{1}{1+(f/f_c)^{2n}}$

where $n$ is the order of the filter. Say we choose a 5th order LPF and we target an   ~ ~-20db (1% power) reduction at the harmonic, then a possible cutoff frequency $f_c= 200$ MHz. That means $f_c\approx (3/1.5) f_{RF}=2f_{RF}$. 

Then we choose a max stimulus frequency. We should target an $f_{max}$ that retains ~90%/-0.5 dB of the original signal. This means a $f_{max}\approx 0.8 f_c$. 

The full filter bank is then:

| Band | RF Source Range | LO Divider | Injected $f_{LO}$ | Source LPF cutoff |
| ---- | --------------- | ---------- | ----------------- | ----------------- |
| 1    | 100 - 160 MHz   | 4          | 404 - 644 MHz     | 200 MHz           |
| 2    | 160  - 250 MHz  | 4          | < 1GHz            | 320 MHz           |
| 3    | 250 - 400 MHz   | 4          | < 2 GHz           | 510 MHz           |
| 4    | 400 - 650 MHz   | 2          | < 2 GHz           | 815 MHz           |
| 5    | 650 - 1000 MHz  | 2          | <2 GHz            | 1.3 GHz           |
| 6    | 1 - 1.6 GHz     | 2          | $f_{RF}$          | 2 GHz             |
| 7    | 1.6  - 2.5 GHz  | 1          | $f_{RF}$          | 3.2 GHz           |
| 8    | 2.5 - 4 GHz     | 1          | $f_{RF}$          | 5.1 GHz           |
| 9    | 4 - 6.5 GHz     | 1          | $f_{RF}$          | 8.15 GHz          |
| 10   | 6.5 - 10 GHz    | 1          | $f_{RF}$          | 10.3 GHz          |

Okay, so these 10 bands are a bit crazy... And I don't believe its easily achievable in engineering. That is, there will be tradeoffs and we can use things like higher-order filters, lightening some of the "rules" I set and also relying on post-processing of the IF signal to alleviate some of these bands. 

Ideas:

* At lower frequncies ( higher LO divider numbers), we can count on having 0 even harmonics, which can allow us to increase the low frequency band sizes by a lot.

* 7th order filters over some of the ranges, particularly the low or mid bands where we can greatly increase the band size. 

* 

After some product browsing, a reasonable target is probably a filter bank of 6 for an SP6T. This can be done.


