# Airband AM Receiver with USRP B210 (GNU Radio)

A simple software-defined radio (SDR) receiver for aviation voice (AM, 118 to 137 MHz). The USRP B210 captures IQ samples, GNU Radio filters one 25 kHz channel, demodulates AM, and plays the audio on the laptop. A live spectrum and waterfall show the band.

Status: the flowgraph builds and runs. An airband signal has not been confirmed yet (work in progress).

Signal chain:

    USRP B210 (RX2) -> Low Pass Filter (decimate by 8) -> AM Demod (decimate by 5) -> Multiply Const (volume) -> Audio Sink
            |
            +-> QT GUI Sink (spectrum + waterfall)

## Hardware

| Item | Details |
|---|---|
| SDR | Ettus USRP B210 (USB 3.0) |
| Antenna | Telescopic whip antenna on the **RX2** port (about 60 cm for airband) |
| Laptop | Dell Latitude 3410, Intel Core i7-10510U, 32 GB RAM, Ubuntu 22.04.5 LTS |

## Software

| Component | Version |
|---|---|
| Radioconda | 2025.03.14 |
| GNU Radio | 3.10.12 |
| UHD (USRP driver) | 4.8.0 |
| Python | 3.12.9 |

Radioconda installs into its own folder (`~/radioconda`) and does not change the system packages or other UHD versions on the laptop. See the FM receiver project for the install steps.

## Running

    source ~/radioconda/bin/activate
    cd ~/sdr_projects/02_airband_am/grc
    python3 airband_am.py

Or open `grc/airband_am.grc` in GNU Radio Companion and use **Run > Generate**, then **Run > Execute**. When finished, close the window and run `conda deactivate`.

## Using the receiver

1. Connect the antenna to RX2 and the B210 to a USB 3.0 port. Turn the laptop volume down first.
2. Move the `freq` slider (25 kHz steps, 118 to 137 MHz) to an airband frequency, for example the Delhi ATIS listed in public sources as 126.4 MHz. Verify frequencies before relying on them.
3. A narrow vertical line at the center of the waterfall is a station. Raise `volume` slowly until you hear audio.
4. Adjust `gain`: lower it (20 to 30) if the noise floor is high, raise it (50 to 60) if the signal is weak.

## Flowgraph parameters

| Block | Setting |
|---|---|
| Sample rate | 1.92 MS/s |
| USRP Source | `type=b200`, antenna RX2, 1 channel, **Stream channels = `[]`** |
| Low Pass Filter | Decimation 8, cutoff 5 kHz, transition 3 kHz (output 240 kS/s) |
| AM Demod | Channel rate 240 kS/s, audio decimation 5, audio pass 5 kHz, audio stop 5.5 kHz |
| Multiply Const | Float, constant = `volume` slider (AM audio is very quiet without it) |
| Audio Sink | 48 kHz |
| `freq` slider | 118 to 137 MHz, step 25 kHz |

## Troubleshooting

| Problem | Fix |
|---|---|
| `Generate Error: invalid syntax` | Set the USRP Source block's **Stream channels** to `[]` |
| No sound from the AM demodulator | Add the volume (Multiply Const) block, because AM output from the B210 is very quiet |
| Waterfall is only noise | Lower `gain`, use Auto Scale, check the Frequency Display tab, and move the antenna near a window |
| `O` characters in the terminal | Use `samp_rate = 960e3`, decimation 4 and AM Demod channel rate 120e3 |
| Device not found | Another program is using the B210. Close it and replug the radio |

## Repository layout

    02_airband_am/
      README.md
      grc/airband_am.grc     GNU Radio Companion flowgraph
      grc/airband_am.py      generated Python script
      docs/                  screenshots

Receive only. No transmission.
