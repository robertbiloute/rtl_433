# Capture and analyze devices

## TL;DR

Capture sample data with `-S unknown`. Note down the expected measurement values from a read-out or head unit.
Check the spectrogram by dropping samples on https://triq.org/iqs/ (it should [look "busy" like this](https://triq.org/iqs/#/honeywell/2Gig-DW10/g001_344.975M_250k.cu8))
Try analyzing each sample with `rtl_433 -A gfile.cu8` to see if there is some real data.
Use the analyzer hints to create a plausible `-X` decoder and demod the data codes.
Then upload some zipped samples to an issue and post a description and tabled codes and values per sample file.

## Verify a transmission

rtl_433 processes radio data in multiple stages. You can follow the stages and verify the data at each point.

First a radio data packet is found and framed.

Get on overview of the band. Check if the transmission is visible and in the expected frequency range.
Use CubicSDR, Gqrx, SigDigger, SDR#, SDRangel or similar SDR UIs to verify you receice a signal.
If you have the SDR reciever on a headless machine try `rtl_tcp` to transport data to a GUI.

::: tip
A quick substitute for an SDR UI is to record a sample, e.g. `-w file_433.92M_250k.cu -T 60` (adjust for the actual frequency and sample rate).
Now drop that .cu8 sample file on https://triq.org/pdv/ to visually inspect the spectrogram (a sideways view of the common SDR waterfalls).
:::

::: warning
Do not plug the receiver directly in a USB port, avoid noise and use a short usb cable.
:::

## Grab a sample

Note the frequency, pick a frequency a little off, e.g 50k above or below.
Then grab the signal with rtl_433, e.g. `rtl_433 -f 433.92M -S unknown`
Visually verify the samples in https://triq.net/iqs
::: tip
The modes for the sample grabber are
- `-S all`: grab all frames found
- `-S unknown`: grab frames that are not decoded by any decoder
- `-S known`: grab frames successfully decoded by some decoder
:::

The band covered is equal to the sample rate.
At the default `433.92M` and `250k` sample rate that's `433.67 MHz` to `434.17 MHz`.
For the `868M` default sample rate of `1024k` that's `867.5 MHz` to `868.5 MHz`.
For the `868M` it's like good to pick `868.3M` for a band of `867.8 MHz` to `868.8 MHz`.

To get a clean signal remove the receiver antenna and place the device at 10cm to the receiver, that mostly isolates the transmissions.

## Analyze the data packet

Then next stage is demodulation of OOK or FSK data.
A run of pulse/gap (OOK) or mark/space (FSK) timings is generated by the demod.
Run `rtl_433 -A SAMPLE.cu8` to get an overview of the timings,
or `rtl_433 -w OOK:- SAMPLE.cu8` to see the raw data.
Write the pulses to a file with `rtl_433 -w SAMPLE.ook SAMPLE.cu8`
and visualize the file with https://triq.net/pdv

::: warning
You need to give the sample rate if it's not 250k, look at the file name, e.g. use `rtl_433 -s 1000k -A SAMPLE_1000k.cu8`
:::

For advanced analysis you can also try out SigRok's Pulseview with `rtl_433 -W out.sr SAMPLE.cu8`.

Be sure to also try with higher sensitivity: `-Y autolevel -Y magest -M noise -M level`

Try different sample rates, for 433M try `-s 1024k`, for 868M try `-s 250k` or  `-s 2048k`.

Try different demods, for 433M try `-Y minmax`, for 868M try `-Y classic`.

## Build a flex decoder

Now build a flex decoder to slice the data into bits.
Use the suggestion or make a guess based on the analyzed pulse data on the coding.

## Document data codes

The last stage is the protocol decoding from the bit data.
Build a table of codes and the expected sensor values to identify where the bytes are and what is contained.
Preferably put the codes and annotations in a [BitBench](https://triq.net/bitbench).

## Example commands

- capture samples not decoded by rtl_433
  `rtl_433 -S unknown`
- capture samples of every received frame
  `rtl_433 -S all`
- anaylze a capture to get an overview of the timings
  `rtl_433 -A SAMPLE.cu8`
- show the raw data pulse data from a captured sample
  `rtl_433 -w OOK:- SAMPLE.cu8`
- convert pulse data from a capture to OOK file
  `rtl_433 -w SAMPLE.ook SAMPLE.cu8`
- try to read codes from a captured sample
  `rtl_433 -X '...' SAMPLE.cu8`
- open a captured sample in SigRok Pulseview
  `rtl_433 -W SAMPLE.sr SAMPLE.cu8`
