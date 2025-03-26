# [Simulation-of-ThreePhase-Photovoltaic-Grid-connected-System](https://github.com/labourer-Lucas/Simulation-of-ThreePhase-Photovoltaic-Grid-connected-System)

## Introduction

This project design three-phase PV grid connected system using PLECS. In this README file, the process of design will be introdunce in detail.

![Q53](/Users/lucaslee/Desktop/Simulation-of-Distributed-Photovoltaic-Grid-connected-System/assets/Q53.png)

The simulation content is listed as follows:

Part 1 – **Boost converter for PV applications**

- PV string characterization
- Design of a boost converter
- Modulation and open loop operation of the boost converter
- Implementation of a Maximum Power Point Tracker



 Part 2 – **Grid Synchronization**

- Basic quadrature PLL (Phase Locked Loop) – principles
- Controller design
- Response to unbalanced voltages
- Response to harmonics
- Moving Average Filter PLL



 Part 3 – **Current and Power control of three-phase inverters**

- Three phase modulation
- Dq modelling
- Current control for L filter
- LCL filter design
- Current control for LCL filter
- Dc bus capacitor design
- Dc bus voltage control
- Droop control
- Instantaneous power measurement



 Part 4 – **System Integration**

- Integration of current control and synchronization
- Integration of the dc bus voltage control
- Integration of the boost converter
- Integration of the power control
- Tests under different grid conditions

## Pre-requirement

The simulation is based on the software [PLECS](https://www.plexim.com/download/standalone).

## Part 1 – **Boost converter for PV applications**

![Circiut diagram of PV system with boost converter](/Users/lucaslee/Desktop/Simulation-of-Distributed-Photovoltaic-Grid-connected-System/assets/Circiut diagram of PV system with boost converter.png)

### Inductor selection

In this section, we are going to set a current ripple and calculate the inductance in the worst case, which is the max power point.

The average value of current through the inductor is:

$$
\overline{I_L} = \frac{P_{PV_{MPP}}}{V_{PV_{MPP}}}=\frac{7667W}{595.6V}=12.87A
$$

In the equation above, $P_{PV_{MPP}}$ and $V_{PV_{MPP}}$ are the power and voltage of the PV module at the maximum power point. Then, we choose a current ripple of 10%, and the ripple current of the inductor $\Delta \overline{I_L}$ can be calculated as:

$$
\Delta{I_L}=\beta \times \overline{I_L}=10\% \times 12.87A=1.287A
$$

The ripple current can also be calculated in representation of voltage applied on it:

$$
\Delta I_{L}=\frac{1}{L}\int_{0}^{DT_{s}}(V_{b}-V_{PV_{MPP}})dt
$$

Combining the above equation with previous expressions, we get:

$$
\begin{aligned}
L_{min} &=\frac{1}{\Delta I_{L}}\left(V_{b}-V_{PV_{MPP}}\right)\frac{V_{PV_{MPP}}}{V_{b}}\cdot\frac{1}{f_{s}}\\
&=\frac{1}{1.287A}\left(800V-595.6V\right)\frac{595.6V}{800V}\cdot\frac{1}{70kHz}\\
&=1.69mH
\end{aligned}
$$

Therefore, we choose $L=2mH$ in our design for some margin.

### Capacitor Selection

In our design, the converter has to comply with the requirement that the maximum amplitude of the input voltage components above 150 kHz should be smaller than 5 mV.

<img src="/Users/lucaslee/Desktop/Simulation-of-Distributed-Photovoltaic-Grid-connected-System/assets/Symplified model of circuit.png" alt="Symplified model of circuit" style="zoom:50%;" />

**Figure 1: Simplified model of circuit when analyzing capacitor**

Simply modeling the PV string as a current source, we simplify the circuit as shown in **Figure 1**. The alternative part of the current is denoted as $I_c$. **Figure 2** shows the shape of $I_C$.

<img src="/Users/lucaslee/Desktop/Simulation-of-Distributed-Photovoltaic-Grid-connected-System/assets/Ic.png" alt="Ic" style="zoom: 50%;" />

**Figure 2: Current through capacitor vs time**

For the waveform of $I_C$, the amplitude of the $n$-th harmonics $\hat{I_{cn}}$ is:

$$
\hat{I_{Cn}}=\frac{\Delta I_L \sin(n\pi D)}{2\pi^2D(1-D)n^2}
$$

At the maximum power point, $D=\frac{V_{PV_{MPP}}}{V_b}=0.7445$. Therefore, the amplitude of the third harmonics of $I_C$ is:

$$
\begin{aligned}
    \hat{I_{C3}} &=\frac{\Delta I_L \sin(n\pi D)}{2\pi^2D(1-D)n^2}\\
    &=\frac{1.287A\times \sin(2\pi \times 0.7445)}{2\pi^2\times 0.7445\times (1-0.7445)\times 3^2}\\
    &=0.0255A
\end{aligned}
$$

The impedance of the capacitor at the third harmonics is:

$$
Z_{C3}=\frac{1}{j2\pi f_{s3} C}
$$

The third harmonics of the voltage is:

$$
V_{C3}=I_{C3}\times Z_{C3}
$$

If we select $V_{C3}=5mV$, then we obtain:

$$
C=\frac{I_{C3}}{V_{C3}\times 2\pi f_{C3}}=3.865\mu F
$$

Therefore, the capacitance should be more than $3.865\mu F$. In our design, we choose $C=10 \mu F$.

### Controller design

In this part, we are going to design the controller for the boost converter. The control system of our design is shown in Figure below. There are two control loops: an outer loop for voltage control and an inner loop for current control. 

$C_v(s)$ and $C_i(s)$ represent the voltage controller and current controller, respectively. The reference voltage of the PV module is denoted as $v_{{pv}}^{*}$. The small vibrations of the duty cycle and DC bus voltage are represented by $\tilde{d}$ and $\tilde{v}_b$, respectively.

The system can be modeled as:

$$
\begin{aligned}         
    \dot{\mathbf{x}} &= \mathbf{A}\mathbf{x} + \mathbf{B}\mathbf{u} \\
    \mathbf{x} &=
    \begin{bmatrix}
        i_L \\
        v_{\mathrm{pv}}
    \end{bmatrix}, \quad
    \mathbf{u} =
    \begin{bmatrix}
        \tilde{d} \\
        \tilde{v}_{\mathrm{b}}
    \end{bmatrix}.
\end{aligned}
$$

![Controlsystem](/Users/lucaslee/Desktop/Simulation-of-Distributed-Photovoltaic-Grid-connected-System/assets/Controlsystem.png)

#### Modeling the system

In this section, we model the boost converter system using small signal analysis.

When $S_1$ is on ($S=1$), applying Kirchhoff's Voltage Law, we have:

$$
\begin{aligned}
    L\dot{i_{i}} &= v_{b} - v_{pv} - i_{L}R_{L} \\
    C\dot{v}_{pv} &= i_L + i_{PV}
\end{aligned}
$$

The state-space form of the above equations is:

$$
\begin{aligned}
    \dot{\mathbf{x}} &= \mathbf{A_1} \mathbf{x} + \mathbf{B_1} \mathbf{u} \\
    \mathbf{A_1} &= \begin{bmatrix}
        -\frac{R_{L}}{L} &-\frac{1}{L} \\
        \frac{1}{C} & 0
    \end{bmatrix}, \quad 
    \mathbf{B_1} = \begin{bmatrix}
        0 &\frac{1}{L} \\
        0 & 0
    \end{bmatrix}
\end{aligned}
$$

When $S_1$ is off ($S=0$), applying Kirchhoff's Voltage Law, we get:

$$
\begin{aligned}
    L\dot{i_{i}} &= -v_{pv} - i_{L}R_{L} \\
    C\dot{v}_{pv} &= i_L + i_{PV}
\end{aligned}
$$

The state-space form of the above equations is:

$$
\begin{aligned}
    \dot{\mathbf{x}} &= \mathbf{A_2} \mathbf{x} + \mathbf{B_2} \mathbf{u} \\
    \mathbf{A_2} &= \begin{bmatrix}
        -\frac{R_{L}}{L} &-\frac{1}{L} \\
        \frac{1}{C} & 0
    \end{bmatrix}, \quad 
    \mathbf{B_2} = \begin{bmatrix}
        0 & 0 \\
        0 & 0
    \end{bmatrix}
\end{aligned}
$$

The system can be described as:

$$
\begin{cases}
    \mathbf{\dot{x}} = \mathbf{A_{1}}\mathbf{x} + \mathbf{B_{1}}\mathbf{u}, & t \in [0, dT_s] \\
    \mathbf{\dot{x}} = \mathbf{A_{2}}\mathbf{x} + \mathbf{B_{2}}\mathbf{u}, & t \in [dT_{s}, T_{s}]
\end{cases}
$$

State-space averaging techniques are used to obtain a set of equations describing the system over one switching cycle. After applying the averaging technique, we obtain:

$$
\dot{\bar{\mathbf{x}}} = [\mathbf{A_{1}}d + \mathbf{A_{2}}(1-d)]\bar{\mathbf{x}} + [\mathbf{B_{1}}d + \mathbf{B_{2}}(1-d)]\bar{\mathbf{u}}
$$

Thus, the state-space average model for the boost converter is:

$$
\dot{\bar{\mathbf{x}}} =
\begin{bmatrix}
    -\frac{R_{L}}{L} & -\frac{1}{L} \\
    \frac{1}{C} & 0
\end{bmatrix}
\bar{\mathbf{x}} +
\begin{bmatrix}
    0 & \frac{d}{L} \\
    0 & 0
\end{bmatrix}
\bar{\mathbf{u}}
$$

Using standard linearization techniques and applying perturbations:

$$
\begin{cases}
    i_{L} = I_{L} + \tilde{i}_{L} \\
    v_{PV} = V_{PV} + \tilde{v}_{PV} \\
    v_{b} = V_{b} + \tilde{v}_{b} \\
    d = D + \tilde{d}
\end{cases}
$$

Finally, the linearized model of the system is:

$$
\begin{aligned}
    \begin{bmatrix}
        \dot{\tilde{i}}_L \\ \dot{\tilde{v}}_{PV}
    \end{bmatrix}
    =
    \begin{bmatrix}
        -\frac{R_L}{L} & -\frac{1}{L} \\
        \frac{1}{C} & 0
    \end{bmatrix}
    \begin{bmatrix}
        \tilde{i_L} \\
        \tilde{v}_{\mathrm{pv}}
    \end{bmatrix}
    +
    \begin{bmatrix}
        \frac{V_b}{L} & \frac{D}{L} \\
        \frac{1}{C} & 0
    \end{bmatrix}
    \begin{bmatrix}
        \tilde{d} \\
        \tilde{v}_{\mathrm{b}}
    \end{bmatrix}
\end{aligned}
$$

#### Transfer function

In this section, we derive the transfer function of the system from the state-space model above.

Since the output of the system is $v_{PV}$ and $i_L$, we can write the output matrix as:

$$
\mathbf{y}=
\begin{bmatrix}
    1 & 0\\
    0 & 1
\end{bmatrix} 
\begin{bmatrix}
    i_L \\
    v_{\mathrm{pv}}
\end{bmatrix}
$$

The transfer function is calculated using the following formula:

$$
G(s)=\frac{Y(s)}{U(s)}=C(sI-A)^{-1}B+D
$$

Therefore, the transfer function is:

$$
G(s)=
\begin{bmatrix}
    G_{id}(s) & G_{ib}(s)\\
    G_{vd}(s) & G_{vb}(s)
\end{bmatrix}
=
\begin{bmatrix}  
    \frac{C\,V_b \,s}{C\,L\,s^2 +C\,R_L \,s+1} & \frac{C\,D\,s}{C\,L\,s^2 +C\,R_L \,s+1}\\
    \frac{V_b }{C\,L\,s^2 +C\,R_L \,s+1} & \frac{D}{C\,L\,s^2 +C\,R_L \,s+1}
\end{bmatrix}
$$

After redrawing the control loop instead of using the state-space model, we obtain the block diagram shown in Figure below.

![controlloop](/Users/lucaslee/Desktop/Simulation-of-Distributed-Photovoltaic-Grid-connected-System/assets/controlloop.png)

#### Current controller design

To design the current controller, we select the transfer function:

$$
G_{id}(s)=\frac{CV_{b}s}{CLs^{2}+CR_{L}s+1}
$$

According to the previous sections, we have determined the device parameters:

$$
\begin{cases}
    C = 10\mu F \\
    R_L = 0.0463\Omega \\
    L = 2mH \\
    V_b = 800V \\
    f_s = 70kHz
\end{cases}
$$
Our PI controllers result in a phase margin of 50$^\circ $ and crossing frequencies of 1/10 of the switching frequency(7 kHz).

![image-20250326205548092](/Users/lucaslee/Desktop/Simulation-of-Distributed-Photovoltaic-Grid-connected-System/assets/image-20250326205548092.png)

Aided by pidtool in Matlab, the parameters of PI controller are:
$$
\begin{cases}
K_p=0.08197\\
K_i=3027
\end{cases}
$$

#### Voltage controller design

