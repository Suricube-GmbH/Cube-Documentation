# Signal generation logic

## Principle
The Signal Generators are based on an integrator design :   


$$
Output_n = S^0_n          \\
S^0_{n+1}= S^0_n + \Delta t * S^1_n \\
S^1_{n+1}= S^1_n + \Delta t * S^2_n \\
S^2_{n+1}= S^2_n + \Delta t * S^3_n \\
S^3_{n+1}= S^3_n           
$$
!!! Note 
     The actual implementation is a bit more complex, as we’re working with (different) finite-bit number representations. We also need to do some unit conversions to take into account the cycle period, and to convert between output voltage and internal number.

A very natural way to control the output is to specify (S00, S10, S20, S30), and then let the integrator run for a period of time. After some time, we can then give a new set of values, changing the output. 
This system allows to generate any *3rd degree polynomial*, which in turn allows to approximate any waveform. 


## Example
[get picture from the Rust simulation]

## Sections and section order
In the cube, an output signal is defined like a piecewise function. Every piece (we call them sections) can be up to a 3rd degree polynomial, and is defined for a specific length. We then specify how the pieces are combined together to form the whole function. There are 2 questions to answer : 
How are the pieces chained together ? 
What kind of transition should there be between each section ? The cube is capable of doing discontinuous, continuous, C1-continous or C2-continous transitions.
Note : C2-transition is actually not a transition at all, as you’re not changing any of the internal integrator's values. 

## Raw vs voltage sections
There are 2 ways to define the sections :

  - **Raw sections** : the values are signed 64-bits numbers, and are sent as-is to the FPGA. 
  - **Voltage sections** : Instead of giving the values for the 4 first derivatives, you select instead the starting point, the ending point, and the slope/   derivative at the start and ending point. You then select which kind of interpolation you would have : constant, linear, quadratic or cubic. The values are converted from Volt , Volt/s, etc… into the appropriate values for the FPGA - taking into account the DAC calibration and the FPGA clock speed.
