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

| Byte | Description | API field | Comment |
|------|-------------|-----------|---------|
|  0 | --- |
|  1 | Mode | Mod | IDU working mode | |
|  2 | Fan speed | Quiet, WdSpd, Tur | IDU fan speed | quiet=7 (also flag at 10.5), low=1, medium-low and low = 2, medium-high and high = 3, turbo = 4, off = 0 |
|  3 | Half degree temp bit flags | -/TemRec | evaporator temp (4), set temp in Fahrenheit (6) - corresponds to **TemRec** field in API |
|  4 | Selected temp. | SetTem | not +40 shifted|
|  5 | Air Temp | TemSen |
|  6 | --- |
|  7 | Evaporator temp | InEvaTem | half deg is byte 3 bit 4 |
|  8 | some flag | 0/1
|  9 | ---
| 10 | Quiet(5)/SE(7) flags. | Quiet/? | SE mode also locks set temp to 27C
| 11-16 | ---

Switching fan speed between medium-low and medium and between medium-high and high does not affect what is reported to Outdoor Unit. Probably a consequence of the early units having only 3 stage fan instead of 5.

Setting Quiet mode via API does strange things:
- 2 is a normal value
- 1 does the same as 2
- 3 actually turns of the fan completely, but if in cooling or dry mode, it doesn't cause the expansion valve to close (or compressor to stop), so the temp can quickly fall below 0
- 4 sets fan to 'low' and disables 10.5 flag
- 5 is same as 1,2

### IDU 0x32 0x2X (50, 32)

| Byte | Description | API field | Comment |
|------|-------------|-----------|---------|
|  0 | --- | 
|  1 | on/off flag (7) | Pow | 128/0
|  2 | --- |
|  3 | night/sleep mode flag (3) | Slp |
| 4-12 | --- |
| 13 | 16
| 14-16 | --- |

I have not noticed any effects of enabling night mode - it doesn't slow down the fan, limit ODU power or anything

### ODU  0x31 0x1X (49, 16)

| Byte | Description | API field | Comment |
|------|-------------|-----------|---------|
|  0 | --- |
|  1 | overall status | 
|  2 | compressor Freq (Hz) | CompressorFqy
|  3 | H/E fan speed | | units unknown
|  4 | ---
|  5 | unknown
|  6 | flag 0/1 | set to one during/around valve close
|  7 | expansion valve setting
|  8 | unknown
|  9 | ---
| 10 | Inlet air temp | TemsSenOut, OutEnvTem |
| 11 | Outlet air temp
| 12 | Compressor temp | CompressorTem
| 13 | ---
| 14 | ---
| 15 | unknown
| 16 | ---

### ODU  0x32 0x1X (50, 16)

| Byte | Description | Comment |
|------|-------------|---------|
|  0-9 | --- |
| 10 | Heat exchanger temp
| 11 | ---
| 12 | ---
| 13 | Valve/gas temp
| 14 | Return/liquid temp | or the other way around, not sure
| 15 | ---
| 16 | ---

### ODU  0x40 0x1X (64, 16)

| Byte | Description | Comment |
|------|-------------|---------|
|  0 | flag 0/1 
|  1 | Compressor Freq cmd | requested value - 49.16.2 follows soon after
|  2 | ---
| 3-5 | unknown 
| 6-16 | ---
