# Practical example of signal generation

## Logic 

!!!warning
    This is work in progress. It might be outdated.

Our goal is to generate a step pattern, which can then either be used in jigsaw or zigzag mode.
The most straightforward way would be to store every step as a section, and to go over 
the section in sequential order. However, doing this for a larger (say 2048) number of steps is 
not possible, as the internal memory for a ASG is only *8 kilobytes (~10 sections, and 
~4000 section orders)*.

!!!TODO
    Evaluate the actual memory for an ASG

Instead, what we'll do is to generate one section for each repetitive part of the final trace :

- a plateau : constant value for a certain time.
- a step : constant derivative for a short amount of time

Then, by using a continous transition for each new section, we can steadily make (quasi-)steps.

```vegalite
{
  "description": "Generated voltage signal",
  "data": {"values": [
    {"time": 0, "voltage": 0, "trace": "target"},
    {"time": 1, "voltage": 0, "trace": "target"},
    {"time": 1, "voltage": 1, "trace": "target"},
    {"time": 2, "voltage": 1, "trace": "target"},
    {"time": 2, "voltage": 2, "trace": "target"},
    {"time": 3, "voltage": 2, "trace": "target"},
    {"time": 3, "voltage": 3, "trace": "target"},
    {"time": 4, "voltage": 3, "trace": "target"},
    {"time": 4, "voltage": 4, "trace": "target"},
    {"time": 5, "voltage": 4, "trace": "target"},
    {"time": 5, "voltage": 0, "trace": "target"},
    {"time": 6, "voltage": 0, "trace": "target"},
    {"time": 6, "voltage": 1, "trace": "target"},
    {"time": 7, "voltage": 1, "trace": "target"},
    {"time": 7, "voltage": 2, "trace": "target"},
    {"time": 8, "voltage": 2, "trace": "target"},
    {"time": 8, "voltage": 3, "trace": "target"},
    {"time": 9, "voltage": 3, "trace": "target"},
    {"time": 9, "voltage": 4, "trace": "target"},
    {"time": 0.0, "voltage": 0, "trace": "effective", "type":"0"},
    {"time": 0.9, "voltage": 0, "trace": "effective", "type":"1"},
    {"time": 1.0, "voltage": 1, "trace": "effective", "type":"0"},
    {"time": 1.9, "voltage": 1, "trace": "effective", "type":"1"},
    {"time": 2.0, "voltage": 2, "trace": "effective", "type":"0"},
    {"time": 2.9, "voltage": 2, "trace": "effective", "type":"1"},
    {"time": 3.0, "voltage": 3, "trace": "effective", "type":"0"},
    {"time": 3.9, "voltage": 3, "trace": "effective", "type":"1"},
    {"time": 4.0, "voltage": 4, "trace": "effective", "type":"0"},
    {"time": 4.9, "voltage": 4, "trace": "effective"},
    {"time": 5.0, "voltage": 0, "trace": "effective"},
    {"time": 5.9, "voltage": 0, "trace": "effective"},
    {"time": 6.0, "voltage": 1, "trace": "effective"},
    {"time": 6.9, "voltage": 1, "trace": "effective"},
    {"time": 7.0, "voltage": 2, "trace": "effective"},
    {"time": 7.9, "voltage": 2, "trace": "effective"},
    {"time": 8.0, "voltage": 3, "trace": "effective"},
    {"time": 8.9, "voltage": 3, "trace": "effective"},
    {"time": 9.0, "voltage": 4, "trace": "effective"}
  ]},
  "layer": [
  {
    "mark": "line",
    "encoding": {
        "x": {"field": "time", "type": "quantitative"},
        "y": {"field": "voltage", "type": "quantitative"},
        "color": {"field": "trace", "type": "nominal"},
        "strokeDash": {"value": [5,2]}    
    }
  }, 
  {
    "transform" : [ {"filter": "datum.type > 0"} ], 
    "mark": {"type": "text", "angle": 0, "dx": -10, "dy":-30, "fontSize": 35},
    "encoding": {        
        "x": {"field": "time", "type": "quantitative"},
        "y": {"field": "voltage", "type": "quantitative"},
        "text": {"value": "⇘" }     
    }
  }
  ]
}

```

This method has 2 potential problems : 

1. We wanted to have an 'instant' change of value. To solve this, we need to make use 
of the fact that the ASG is running faster than the ADC is generating the signal : 
if it takes 4 ticks for the ADC to change value, and we do our slope in 2 ticks, then 
the computation is working, and the ADC will never has seen it. 
2. If we got some errors while doing floating point arithmetic or rounding, then 
they would accumulate over time, and we might not get the expected output. To solve this, 
we do a discontinous section transition at every time we reach the end of our pattern.

!!!TODO
    Need to change some clocking issue in the FPGA code to be actually able to solve 1.

## Code 

```rust
pub fn generate_steps(
    max_voltage: f64,
    min_voltage: f64,
    step_width_t: f64, // in ms
    step_number: u64,
    do_zigzag: bool,
    calibration: &ASGParameters,
) -> Signal {
    let step_height = (max_voltage - min_voltage) / (step_number as f64 - 1.0);
    //let rising_time = calibration.ms_to_ticks(0.005 / 71. * 3.) as i64; // currently set at 3 ticks
    let rising_time = calibration.ms_to_ticks(0.01) as i64; // currently set at 3 ticks
    // Constant on the min value
    let plateau_min_value = VoltageSection {
        duration_in_ms: step_width_t - calibration.ticks_to_ms(rising_time), // total step time is as given
        starting_level: min_voltage,
        starting_derivative: 0.0,
        ending_level: max_voltage,
        ending_derivative: 0.0,
        interpolation: Interpolation::Constant,
    }
    .convert_to_raw(calibration);

    // constant on the max value
    let plateau_max_value = VoltageSection {
        duration_in_ms: step_width_t - calibration.ticks_to_ms(rising_time), // total step time is as given
        starting_level: max_voltage,
        starting_derivative: 0.0,
        ending_level: max_voltage,
        ending_derivative: 0.0,
        interpolation: Interpolation::Constant,
    }
    .convert_to_raw(calibration);

    let step_up = VoltageSection {
        // if it's n ticks long, it will have (n-1) steps
        // so we need to have a steeper slope
        duration_in_ms: calibration.ticks_to_ms(rising_time - 1),
        starting_level: 0.0,
        starting_derivative: 0.0,
        ending_level: step_height,
        ending_derivative: 0.0,
        interpolation: Interpolation::Linear,
    }
    .convert_to_raw(calibration)
    .overwrite_length(rising_time as u64);

    let step_down = VoltageSection {
        // if it's n ticks long, it will have (n-1) steps
        // so we need to have a steeper slope
        duration_in_ms: calibration.ticks_to_ms(rising_time - 1),
        starting_level: 0.0,
        starting_derivative: 0.0,
        ending_level: -step_height,
        ending_derivative: 0.0,
        interpolation: Interpolation::Linear,
    }
    .convert_to_raw(calibration)
    .overwrite_length(rising_time as u64);

    // For jigsaw pattern : we go from max to min value directly
    let step_to_min = VoltageSection {
        duration_in_ms: calibration.ticks_to_ms(rising_time), // Close to actual value so interpolation is correct
        starting_level: min_voltage,
        starting_derivative: 0.0,
        ending_level: 0.0,
        ending_derivative: 0.0,
        interpolation: Interpolation::Constant,
    }
    .convert_to_raw(calibration)
    .overwrite_length(rising_time as u64);

    // If you change the order of the sections, also change the later code !
    let sections = vec![
        plateau_min_value, // 0
        step_up,           // 1
        step_down,         // 2
        step_to_min,       // 3
        plateau_max_value, // 4
    ];

    // Going up ↗ - zig zag and jigsaw
    let section_order_up = vec![
        SectionOrder {
            number: 0, // min_value
            transition: Transition::Continous,
        },
        SectionOrder {
            number: 1, // step_up
            transition: Transition::Continous,
        },
    ]
    .repeat(step_number as usize - 1); // we do the final step and the transition manually

    // Going down ↘ - for zig zag
    let section_order_down = vec![
        SectionOrder {
            number: 4, // Plateau max value
            transition: Transition::Continous,
        },
        SectionOrder {
            number: 2, // step_down
            transition: Transition::Continous,
        },
    ]
    .repeat(step_number as usize - 1); // we do the final step and the transition manually

    // Combining everything into the final signal
    Signal {
        sections: sections,
        section_order: if do_zigzag {
            // zigzag pattern
            vec![
                section_order_up,
                vec![
                    SectionOrder {
                        // Force plateau at max value - Resets rounding error
                        number: 4,
                        transition: Transition::Discontinous,
                    },
                    SectionOrder {
                        // A step where we don't move
                        number: 3, // step_to_min : constant + continous => no changes
                        transition: Transition::Continous,
                    },
                ],
                section_order_down,
                vec![
                    SectionOrder {
                        // Force plateau at min value - Resets rounding error
                        number: 0,
                        transition: Transition::Discontinous,
                    },
                    SectionOrder {
                        // A step where we don't move
                        number: 3, // step_to_min : constant + continous => no changes
                        transition: Transition::Continous,
                    },
                ],
            ]
            .concat()
        } else {
            // jigsaw pattern
            [
                //Adding missing final plateau, as well as final transition
                section_order_up,
                vec![
                    SectionOrder {
                        // Force plateau at max value
                        number: 4,
                        transition: Transition::Discontinous,
                    },
                    SectionOrder {
                        // Force step to min value
                        number: 3,
                        transition: Transition::Discontinous,
                    },
                ],
            ]
            .concat()
        },
    }
}
```

## Resulting JSON code
``` JSON
{
   "sections":[
      {
         "s0_0":6434,
         "s1_0":0,
         "s2_0":0,
         "s3_0":0,
         "length":5571,
         "interpolation":"constant"
      },
      {
         "s0_0":32696,
         "s1_0":258573883,
         "s2_0":0,
         "s3_0":0,
         "length":143,
         "interpolation":"linear"
      },
      {
         "s0_0":32696,
         "s1_0":-258573883,
         "s2_0":0,
         "s3_0":0,
         "length":143,
         "interpolation":"linear"
      },
      {
         "s0_0":6434,
         "s1_0":0,
         "s2_0":0,
         "s3_0":0,
         "length":143,
         "interpolation":"constant"
      },
      {
         "s0_0":58959,
         "s1_0":0,
         "s2_0":0,
         "s3_0":0,
         "length":5571,
         "interpolation":"constant"
      }
   ],
   "section_order":[
      {
         "number":0,
         "transition":"continous"
      },
      {
         "number":1,
         "transition":"continous"
      },
      {
         "number":0,
         "transition":"continous"
      },
      {
         "number":1,
         "transition":"continous"
      },
      {
         "number":0,
         "transition":"continous"
      },
      {
         "number":1,
         "transition":"continous"
      },
      {
         "number":4,
         "transition":"discontinous"
      },
      {
         "number":3,
         "transition":"discontinous"
      }
   ]
}
```
