# Structure

Every packet is 20 bytes: 2 header bytes, 17 data bytes and one byte checksum

## Header

Header format is AA BC
AA - table id, or something like that. the ones I found were 0x31 (49), 0x32 (50) and 0x40 (64)
B - sender - either 1 (ODU) or 2 (IDU).
C - it is a 1-based channel number, so 1, 2, etc depending how many internal units there are

So the headers for first channel would be
0x31 0x11, 0x32 0x11, 0x40 0x11 for ODU on channel #1
0x31 0x21, 0x32 0x21 for IDU #1

0x31 0x12, 0x32 0x12, 0x40 0x12 for ODU on channel #2
0x31 0x22, 0x32 0x22 for IDU #2

and so on.

## Checksum

That is just a modulo 256 sum of first 19 bytes, so literally a sum...

## Data

All temperatures except user setting are shifted by 40, so 21 is 61 etc.
Also all temperatures are always in Celsius scale. The only change when switched to Fahrenheit is that 6th bit of 49.32.3 is being used for 'half steps', but I haven't noticed it actually has any effect,
so basically Fahrhenheit scale is pretty fake and superficial - only the display changes. There is no way the measured air temperature can be read with greater resolution than 1C - the app, when showing room temperature, skips certain values.
The only place they can be seen is the IDU display when you tell it to briefly show room temp - and for 'half steps' that value will be different (higher) than the one displayed by app. So the IDU has greater resolution internally, but it is not exposed anywhere.

### IDU 0x31 0x2X (49, 32)

| Byte | Description | Comment |
|------|-------------|---------|
|  0 | --- |  |
|  1 | Mode | IDU working mode | |
|  2 | Fan speed | IDU fan speed | |
|  3 | Half degree temp values | 4th bit (8) for evaporator temp, 6th (32) for Fahrenheit selection |
|  4 | Selected temp. | |
|  5 | Air Temp | +40 |
|  6 | --- |
|  7 | Evaporator temp | +40, half deg is byte 3 bit 4 |
|  8 | some flag
|  9 | ---
| 10 | Quiet(5)/SE(4) flags.
| 11-16 | ---

