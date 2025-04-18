I've created a new profile for the Passat GTE because the SOC expressions used for the e-Golf and e-Up seem to only provide approximate SOC values. My analysis has shown that the value B4 represents the State of Charge with respect to the gross battery capacity as a 1-byte value (0-255). However, since an upper and lower reserve exists to protect the battery, the 0% SOC displayed in the car corresponds to a B4 value of 60 (MinB4) and 100% SOC corresponds to a B4 value of 238 (MaxB4). This means that the usable SOC range is between approximately 23% (60/255) and 93% (238/255) of the gross capacity. The resulting effective state of charge of the battery can be generally represented by shifting the offset by MinB4 and scaling to 100% using the following expression:

**General expression**

$$
SOC = \frac{B4-MinB4}{MaxB4-MinB4} * 100
$$

Since different VW models may have different batteries and/or charging/discharging limits despite using the same PID, the SOC expression should be individually adapted for each model with respect to the MinB4 and MaxB4 values. For this purpose, the additional parameter RAWSOC can be used, which directly reflects the value of B4. The value of RAWSOC must be read out at 0% and 100% SOC displayed in the car and used as MinB4 and MaxB4 in the general expression, resulting in the expression for the **specific expression for VW Passat GTE (Facelift2019):**

**1. Read RAWSOC using existing profile with PID 22028C1**

>**URL:** *http://wican_xxxxxxxxxxxx.local/autopid_data*

**Result**

```json
{
   "5B-HybrBatPackRemLife":93.33,
    "42-ControlModuleVolt":14.41,
    "SOC":xx.xx,
    "RAWSOC":xxx
}
```

**2. Determination of MinB4/MaxB4**

    Car Display: 0%   => RAWSOC: 60  => MinB4
    Car Display: 100% => RAWSOC: 238 => MaxB4 

**3. Insert the specific values ​​into the general expresssion**

SOC = ((B4 - *MinB4*) / (*MaxB4* - *MinB4*)) * 100

SOC = ((B4 - 60) / (238 - 60)) * 100

SOC = ((B4 - 60) / 178) * 100

**4. Examples for different values of B4**

	B4 = 60:  ((60-60)  / 178) * 100 = 0.00
	B4 = 100: ((100-60) / 178) * 100 = 22.47
    B4 = 200: ((100-60) / 178) * 100 = 78.65
	B4 = 238: ((238-60) / 178) * 100 = 100.00
    