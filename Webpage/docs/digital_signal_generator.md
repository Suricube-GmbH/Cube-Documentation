# Digital signal generator
They are based on the analog signal generators, except that their output range is restricted to 1 bit instead of 16 ones.

## Input
- Digital sections array :
```json
    TODO
```
- Section order array :
```
    TODO
```
- Iteration constraints type and value. **type** determines if the DSG should run continously, pause after a certain number of sections or pause after a certain number of repeats. **value** determines the 
*number* of iterations after which the SG pauses. 
```
    TODO
```

## Output
A digital signal, which is forwarded to a digital output channel.

## Hardware parameters
These are values you shouldn't have to touch or change yourself.

- upshifting
- downshifting
- cycles_per_iteration