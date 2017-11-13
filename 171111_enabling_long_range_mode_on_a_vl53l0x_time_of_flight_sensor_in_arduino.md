# Enabling long range mode on a VL53L0X Time of Flight sensor in Arduino

by snm, November 11th, 2017

[Adafruit VL53L0X Time of Flight Distance Sensor - ~30 to 1000mm](https://www.adafruit.com/product/3317)

Wire up to a Raspberry Pi Zero: 3V3, GND, SDA, and SCL.

i2cdetect -y 1 finds the device at address 0x29.

Interface with [VL53L0X_rasp_python](https://github.com/johnbryanmoore/VL53L0X_rasp_python)

```sh
make
cd python
python VL53L0X_example.py
```

Move your hand in front of the sensor, it should detect how far it is away. Their example runs for a hundred iterations and uses better accuracy mode, but for my purposes I'm more interested in range, so I switched to long range mode. Simple modified example:

```python
#!/usr/bin/python
import time
import VL53L0X

tof = VL53L0X.VL53L0X()
tof.start_ranging(VL53L0X.VL53L0X_LONG_RANGE_MODE)
timing = tof.get_timing()

while True:
    distance = tof.get_distance()
    print "%d cm" % (distance / 10,)
    time.sleep(timing/1000000.00)

tof.stop_ranging()
```

Works but I'd like to use it on an Arduino instead. Using Arduino.app, run the example from [Adafruit_VL53L0X](https://github.com/adafruit/Adafruit_VL53L0X), except add to the beginning of setup(), to configure I2C's SDA on pin D4 and SCL on pin D5:

```c
void setup() {
  Wire.pins(D4, D5); // sda, scl
```

To confirm the device shows up at address 0x29, we can use [i2c_scanner](http://playground.arduino.cc/Main/I2cScanner), also making the same change as above.

Now the Arduino example runs on the device, but it doesn't reach very far. How to enable long range mode with this library? Here are all the modes, from vl53l0x_python.c in https://github.com/johnbryanmoore/VL53L0X_rasp_python:

```c
#define VL53L0X_GOOD_ACCURACY_MODE      0   // Good Accuracy mode
#define VL53L0X_BETTER_ACCURACY_MODE    1   // Better Accuracy mode
#define VL53L0X_BEST_ACCURACY_MODE      2   // Best Accuracy mode
#define VL53L0X_LONG_RANGE_MODE         3   // Longe Range mode
#define VL53L0X_HIGH_SPEED_MODE         4   // High Speed mode
```

This enables a bunch of specific configuration parameters:

```c
case VL53L0X_LONG_RANGE_MODE:
    printf("VL53L0X_LONG_RANGE_MODE\n");
    if (Status == VL53L0X_ERROR_NONE)
    {
        Status = VL53L0X_SetLimitCheckValue(pMyDevice[object_number],
                    VL53L0X_CHECKENABLE_SIGNAL_RATE_FINAL_RANGE,    
                    (FixPoint1616_t)(0.1*65536));                   

        if (Status == VL53L0X_ERROR_NONE)
        {
            Status = VL53L0X_SetLimitCheckValue(pMyDevice[object_number],
                        VL53L0X_CHECKENABLE_SIGMA_FINAL_RANGE,          
                        (FixPoint1616_t)(60*65536));                    

            if (Status == VL53L0X_ERROR_NONE)
            {
                Status =
                    VL53L0X_SetMeasurementTimingBudgetMicroSeconds(pMyDevice[object_number], 33000);

                if (Status == VL53L0X_ERROR_NONE)               
                {
                    Status = VL53L0X_SetVcselPulsePeriod(pMyDevice[object_number],
                                VL53L0X_VCSEL_PERIOD_PRE_RANGE, 18);            

                    if (Status == VL53L0X_ERROR_NONE)               
                    {
                        Status = VL53L0X_SetVcselPulsePeriod(pMyDevice[object_number],
                                    VL53L0X_VCSEL_PERIOD_FINAL_RANGE, 14);          
                    }
                }
            }
        }
    }
    break;
```

The Adafruit library hardcodes these values:

```c
  // Enable/Disable Sigma and Signal check
  if( Status == VL53L0X_ERROR_NONE ) {
      Status = VL53L0X_SetLimitCheckEnable( pMyDevice, VL53L0X_CHECKENABLE_SIGMA_FINAL_RANGE, 1 );
  }

  if( Status == VL53L0X_ERROR_NONE ) {
      Status = VL53L0X_SetLimitCheckEnable( pMyDevice, VL53L0X_CHECKENABLE_SIGNAL_RATE_FINAL_RANGE, 1 );
  }

  if( Status == VL53L0X_ERROR_NONE ) {
      Status = VL53L0X_SetLimitCheckEnable( pMyDevice, VL53L0X_CHECKENABLE_RANGE_IGNORE_THRESHOLD, 1 );
  }

  if( Status == VL53L0X_ERROR_NONE ) {
      Status = VL53L0X_SetLimitCheckValue( pMyDevice, VL53L0X_CHECKENABLE_RANGE_IGNORE_THRESHOLD, (FixPoint1616_t)( 1.5 * 0.023 * 65536 ) );
  }
```
  
It's easy enough to paste this code into Adafruit_VL53L0X::begin(), taking care to remove the array reference (replacing `pMyDevice[object_number]` with `pMyDevice`). Now the sensor can detect a respectably longer distance:

```
Reading a measurement... Distance (mm): 204
Reading a measurement... Distance (mm): 209
Reading a measurement... Distance (mm): 247
Reading a measurement... Distance (mm): 298
Reading a measurement... Distance (mm): 349
Reading a measurement... Distance (mm): 409
Reading a measurement... Distance (mm): 448
Reading a measurement... Distance (mm): 481
Reading a measurement... Distance (mm): 503
Reading a measurement... Distance (mm): 527
Reading a measurement... Distance (mm): 537
Reading a measurement... Distance (mm): 539
Reading a measurement... Distance (mm): 549
Reading a measurement... Distance (mm): 555
Reading a measurement... Distance (mm): 559
Reading a measurement... Distance (mm): 591
Reading a measurement... Distance (mm): 570
Reading a measurement... Distance (mm): 602
Reading a measurement... Distance (mm): 608
```

If you want my changes they are available here: https://github.com/satoshinm/Adafruit_VL53L0X/commits/longrange

