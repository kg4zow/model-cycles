# Model:Cycles `.mcpst` File Format

John Simpson `<jms1@jms1.net>` 2022-04-09

&#x26A0;&#xFE0F; **This document is under construction.**

If you are seeing this notice, be aware that the information below is definitely incomplete and possibly incorrect.

**Don't trust anything you read here.** *(Or anywhere else, for that matter.)*

# High Level File Format

The `.mcpst` file itself is a ZIP file containing two files:

* **`manifest.json`** - items identifying what's in the file and presumably what machine(s) it's for. The important part is the `Payload` key, which contains the name of the other file in the ZIP file.

    ```json
    {
      "FormatVersion": "1.0",
      "ProductType": [
        "27"
      ],
      "Payload": "xyzzy",
      "FileType": "Sound"
    }
    ```

    I'm not sure, but I suspect "ProductType 27" means it's for the Model:Cycles. The "Project" files have the same basic format (a ZIP file containing a `manifest.json` and a binary blob), and those `manifest.json` files also contain "ProductType 27".

* **Binary file** - a binary file containing information about a Model:Cycles "Preset", which includes the following:

    * Which "machine" (synthesis engine) to use, i.e. Kick, Snare, Metal, Perc, Tone, or Chord.
    * Parameters for that engine, i.e. Pitch, Color, Shape, etc. (list of specific parameters?)
    * Some other parameter values which I didn't think would be stored as part of a "pattern", such as Decay, Delay Send, and Reverb Send.

    The idea is, if you know which engine and parameters to use, you can manually "dial in" a preset that somebody else may have discovered, without having to deal with connecting the Model:Cycles to a computer and copying the `.mcpst` files into it.

**The contents of the binary file are what I didn't know.** Obviously this is no longer true, I have figured out *some* of it, which is why I'm writing this document.

From what I've been told on Reddit, a few other people have figured out how to unzip the file and *found* the binary file within, but nobody seems to have taken the next step and figured out the format of the binary file (or if they did, they didn't put it online where the search engines could find it).

# Binary File Format

This is *currently* what the output looks like...

```
$ mcpst-dump original.mcpst
0000 | ab cd ef 12  4d 43 53 43  00 00 00 70  01 00 1d 2c | ....MCSC...p...,
0010 | 10 57 84 3b  41 38 51 e0  41 38 a0 10  00 00 00 0c | .W.;A8Q.A8......
0020 | 41 35 48 f0  41 38 8e 20  40 0d 08 ce  40 0e 5d aa | A5H.A8. @...@.].
0030 | be ef ba ce  00 00 00 02  00 00 00 00  50 72 65 73 | ............Pres
0040 | 65 74 31 00  00 00 00 00  00 00 00 00  00 00 70 00 | et1...........p.
0050 | 04 00 40 00  00 00 00 00  00 00 00 00  40 00 04 00 | ..@.........@...
0060 | 40 00 18 00  28 00 26 00  34 00 00 00  00 00 40 00 | @...(.&.4.....@.
0070 | 00 00 2a 00  3c 00 00 00  00 00 40 00  00 00 00 00 | ..*.<.....@.....
0080 | 00 00 00 00  00 00 00 00  00 00 00 00  40 00 00 7f | ............@...
0090 | ba ce f0 0c  00 00 00 00  00 00 00 00  00 00 00 00 | ................

Name        "Preset1"
Machine     Tone
Pitch        +0.0
Color       2.000
Shape       40
Sweep       38
Contour     52
Decay       42
Delay Send  OFF
Reverb Send OFF
```

### `003c-0047` Preset Name

This is the name that was "dialed into" the Model:Cycles when the Preset file was originally created. Note that if the `.mcpst` file is later renamed, the name *inside* the file doesn't change.

The name is up to 12 bytes long, stored as ASCII characters. If the name is shorter than 12 bytes, there will be a zero byte after the last character (as shown at `0043` in the example above). In some cases there may be remnants of a previous name *after* that zero byte, these are ignored if present.

### `005e` Machine

Which "machine" (synthesis engine) to use to play the sound.

* `00` = Kick
* `01` = Snare
* `02` = Metal
* `03` = Perc
* `04` = Tone
* `05` = Chord

### `0060-0061` Pitch

When playing a sound, frequency-shift it up or down by a number of semitones. The sound can be shifted up to two octaves up or down, by *tenths* of semitones.

Byte `0060` contains the "whole number" portion, with `0x40` meaning 0, `0x28` meaning -24, and `0x58` meaning +24.

Byte `0061` contains the "fractional" portion, as a binary fraction (i.e. the number of 256th's).

| Bytes     | Value | Notes     |
|:----------|------:|:----------|
| `28 00`   | -24.0 | minimum   |
| `3e e6`   | - 1.1 |           |
| `3f 00`   | - 1.0 |           |
| `3f e6`   | - 0.1 |           |
| `40 00`   |   0.0 |           |
| `40 19`   | + 0.1 |           |
| `40 33`   | + 0.2 |           |
| `40 4c`   | + 0.3 |           |
| `40 66`   | + 0.4 |           |
| `40 80`   | + 0.5 |           |
| `40 99`   | + 0.6 |           |
| `40 b3`   | + 0.7 |           |
| `40 cc`   | + 0.8 |           |
| `40 e6`   | + 0.9 |           |
| `41 00`   | + 1.0 |           |
| `41 80`   | + 1.5 |           |
| `42 00`   | + 2.0 |           |
| `58 00`   | +24.0 | maximum   |

### `0062` Color

Machine parameter, interpreted differently by different "machines". See the [user manual](https://www.elektron.se/support/?connection=modelcycles#manuals) for more information.

Uses values from 0-127, with some "machines" presenting the value differently.

- Kick (10)
- Snare (0)
- Metal (16)
- Perc (15)
- Tone (2.000)
    - Has 128 discrete values, shown on screen as ...

        | Byte  | Value     | Byte  | Value | Byte  | Value | Byte  | Value |
        |------:|:----------|------:|:------|------:|:------|------:|:------|
        | `00`  | 0.0125    | `20`  | 2.400 | `40`  | 4.400 | `60`  | 6.400 |
        | `01`  | 0.075     | `21`  | 2.500 | `41`  | 4.500 | `61`  | 6.500 |
        | `02`  | 0.125     | `22`  | 2.600 | `42`  | 4.600 | `62`  | 6.600 |
        | `03`  | 0.250     | `23`  | 2.750 | `43`  | 4.750 | `63`  | 6.750 |
        | `04`  | 0.500     | `24`  | 2.800 | `44`  | 4.800 | `64`  | 6.800 |
        | `05`  | 0.750     | `25`  | 2.900 | `45`  | 4.900 | `65`  | 6.900 |
        | `06`  | 0.900     | `26`  | 2.970 | `46`  | 4.950 | `66`  | 6.930 |
        | `07`  | 0.990     | `27`  | 2.997 | `47`  | 4.995 | `67`  | 6.993 |
        | `08`  | 1.000     | `28`  | 3.000 | `48`  | 5.000 | `68`  | 7.000 |
        | `09`  | 1.001     | `29`  | 3.003 | `49`  | 5.005 | `69`  | 7.007 |
        | `0a`  | 1.010     | `2a`  | 3.030 | `4a`  | 5.050 | `6a`  | 7.070 |
        | `0b`  | 1.100     | `2b`  | 3.100 | `4b`  | 5.100 | `6b`  | 7.100 |
        | `0c`  | 1.150     | `2c`  | 3.150 | `4c`  | 5.150 | `6c`  | 7.150 |
        | `0d`  | 1.200     | `2d`  | 3.200 | `4d`  | 5.200 | `6d`  | 7.200 |
        | `0e`  | 1.250     | `2e`  | 3.250 | `4e`  | 5.250 | `6e`  | 7.250 |
        | `0f`  | 1.300     | `2f`  | 3.300 | `4f`  | 5.300 | `6f`  | 7.300 |
        | `10`  | 1.400     | `30`  | 3.400 | `50`  | 5.400 | `70`  | 7.400 |
        | `11`  | 1.500     | `31`  | 3.500 | `51`  | 5.500 | `71`  | 7.500 |
        | `12`  | 1.600     | `32`  | 3.600 | `52`  | 5.600 | `72`  | 7.600 |
        | `13`  | 1.750     | `33`  | 3.750 | `53`  | 5.750 | `73`  | 7.750 |
        | `14`  | 1.800     | `34`  | 3.800 | `54`  | 5.800 | `74`  | 7.800 |
        | `15`  | 1.900     | `35`  | 3.900 | `55`  | 5.900 | `75`  | 7.900 |
        | `16`  | 1.980     | `36`  | 3.960 | `56`  | 5.940 | `76`  | 7.920 |
        | `17`  | 1.998     | `37`  | 3.996 | `57`  | 5.994 | `77`  | 7.992 |
        | `18`  | 2.000     | `38`  | 4.000 | `58`  | 6.000 | `78`  | 8.000 |
        | `19`  | 2.002     | `39`  | 4.004 | `59`  | 6.006 | `79`  | 9.000 |
        | `1a`  | 2.020     | `3a`  | 4.040 | `5a`  | 6.060 | `7a`  | 10.00 |
        | `1b`  | 2.100     | `3b`  | 4.100 | `5b`  | 6.100 | `7b`  | 11.00 |
        | `1c`  | 2.150     | `3c`  | 4.150 | `5c`  | 6.150 | `7c`  | 12.00 |
        | `1d`  | 2.200     | `3d`  | 4.200 | `5d`  | 6.200 | `7d`  | 13.00 |
        | `1e`  | 2.250     | `3e`  | 4.250 | `5e`  | 6.250 | `7e`  | 14.00 |
        | `1f`  | 2.300     | `3f`  | 4.300 | `5f`  | 6.300 | `7f`  | 15.00 |

- Chord (32)
    - Some values have names which are shown with the values.
        - `32` = `ROOTPOS`
        - `62` = `1ST INV`
        - `74` = `2ND INV`
        - `84` = `3RD INV`

### `0064` Shape

Machine parameter, interpreted differently by different "machines". See the [user manual](https://www.elektron.se/support/?connection=modelcycles#manuals) for more information.

Uses values from 0-127.

- Kick (16)
- Snare (127)
- Metal (46)
- Perc (38)
- Tone (40)
- Chord (4)
    - Only values `01`-`38` are used
    - These values have specific names

        | Byte  | Chord Name    | Byte  | Chord Name    |
        |:-----:|:--------------|:-----:|:--------------|
        | `01`  | `Unison x 2`  | `20`  | `m7b5`        |
        | `02`  | `Unison x 3`  | `21`  | `M7b5`        |
        | `03`  | `Unison x 4`  | `22`  | `M#5`         |
        | `04`  | `Minor`       | `23`  | `m7#5`        |
        | `05`  | `Major`       | `24`  | `M7#5`        |
        | `06`  | `sus2`        | `25`  | `mb6`         |
        | `07`  | `sus4`        | `26`  | `m9no5`       |
        | `08`  | `m7`          | `27`  | `M9no5`       |
        | `09`  | `M7`          | `28`  | `Madd9b5`     |
        | `10`  | `mMaj7`       | `29`  | `Maj7b5`      |
        | `11`  | `Maj7`        | `30`  | `M7b9no5`     |
        | `12`  | `7sus4`       | `31`  | `sus4#5b9`    |
        | `13`  | `dim7`        | `32`  | `sus4add#5`   |
        | `14`  | `madd9`       | `33`  | `Maddb5`      |
        | `15`  | `Madd9`       | `34`  | `M6add4no5`   |
        | `16`  | `m6`          | `35`  | `Maj7/6no5`   |
        | `17`  | `M6`          | `36`  | `Maj9no5`     |
        | `18`  | `mb5`         | `37`  | `Fourths`     |
        | `19`  | `Mb5`         | `38`  | `Fifths`      |


### `0066` Sweep

Machine parameter, interpreted differently by different "machines". See the [user manual](https://www.elektron.se/support/?connection=modelcycles#manuals) for more information.

Uses values from 0-127.

- Kick (16)
- Snare (8)
- Metal (48)
- Perc (48)
- Tone (38)
- Chord (43)

### `0068` Contour

Machine parameter, interpreted differently by different "machines". See the [user manual](https://www.elektron.se/support/?connection=modelcycles#manuals) for more information.

Uses values from 0-127.

- Kick (24)
- Snare (0)
- Metal (0)
- Perc (109)
- Tone (52)
- Chord (24)

### `0072` Decay (Amp Decay)

Not a "machine parameter", but the value seems to be stored in the Preset file.

Uses values from 0-127.

- Kick (28)
- Snare (40)
- Metal (20)
- Perc (26)
- Tone (42)
- Chord (64)

### `0076` Delay Send

Not a "machine parameter", but the value seems to be stored in the Preset file.

Uses values from 0-127, with 0 shown as `OFF`.

- Kick (OFF)
- Snare (OFF
- Metal (OFF)
- Perc (OFF)
- Tone (OFF)
- Chord (OFF)

### `0078` Reverb Send

Not a "machine parameter", but the value seems to be stored in the Preset file.

Uses values from 0-127, with 0 shown as `OFF`.

- Kick (OFF)
- Snare (OFF
- Metal (OFF)
- Perc (OFF)
- Tone (OFF)
- Chord (OFF)

## Not sure yet

### LFO Speed

- Range -64 to 63

### Volume+Dist

- Range 0-127

### Swing

- Range 50% to 80%

### Delay Time

- Range 1-128

### Delay Feedback

- Range 0-198


## Not part of a Preset

### Reverb Size

- Range 1-127 plus INF
- **Shared by all tracks**

### Reverb Tone

- Range -64 to 63
- **Shared by all tracks**

### Nudge

- **Only applies to trigs**

### Chance

- **Only applies to trigs**

### Conditional Trigger

- **Only applies to trigs**
