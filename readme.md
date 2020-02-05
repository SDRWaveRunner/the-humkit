# Low Frequent Sound Detection
The hum-detector, recorder and analyser. This set of tools can be used to detect low-frequent sounds and infrasound. Analysing, recording and post-processing can be done with this toolkit.
The toolkit got it's name: **The HumKit**

## Background
Back in 2019 I got aware of the problems around infrasound and [The Hum] and the problems people can suffer from this.
As this was new for me, i decided to do some research on this.

### Definitions

* Low-frequent sound is defined as sounds with a frequency lower than 125Hz
* Infrasound is defined as sounds with a frequency lower than 20Hz [[1]] as it describes te frequency-range humans generally can not hear.



### Detecting sound
Usually sound is detected using a microphone and a recorder. In a computer-world this would be a microphone connected to a soundcard. The recording is done with software and processed and stored in files.

#### Normal software
A normal soundcard can record more signals than the human ear can hear. And as the sensitivity of the human ear is not equal for all the frequency one can hear, this curve, known as the dBA-curve [[2]], is applied in software to the recorded sound. Therefor we need to use some other software without any filtering.


#### GNUradio
GNUradio is, as described on the website [[3]]: GNU Radio is a free & open-source software development toolkit that provides signal processing blocks to implement software radios. It can be used with readily-available low-cost external RF hardware to create software-defined radios, or without hardware in a simulation-like environment. It is widely used in research, industry, academia, government, and hobbyist environments to support both wireless communications research and real-world radio systems.

With GNUradio we can do signal processing on digitized signals. A soundcard is an AD-converter for audio so GNUradio can be used to process the signals from the soundcard and apply all the filtering available in the toolkit.

## The software
The spectrum of interest are frequencies up to 125Hz. Sampling, processing and recording the spectrum above 125Hz is does not add extra information so this piece of spectrum is filtered from the spectrum and the samplerate is decimated to 512 samples per second.

While a soundcard is a plain analog-to-digital converter, there are no complex (IQ) samples available. So the choosen samplerate is 512 samples per second. This gives the possibility to measure up to 256Hz. [[4]]
After downsampling to this rate, a low-pass filter is applied. This filter is set to 150Hz. 
The resulting information is displayed both in waterfall as in frequency-domain.

Recording audio with a samplerate of 512 samples per second results in a raw recording of approximately 7.1MB per hour which makes long recordings feasable with a moderate SD-card for storage.

### Parts of the software
The software is devided into several pieces:

1. **low_frequency.grc**<br>
	This is the main toolkit for live inspection of signals. Choosing the right audio-card and microphone allows to display the low-frequent spectrum.
2. **low2_rec.grc**<br>
	This is the headless toolkit to record only the low-frequent part of the audio-spectrum. It writes 512 samples per second raw files from the filtered AD-stream.
3. **playback.grc**<br>
	Playing back the samples is possible, showing the same interface as low_frequency. The playback-rate can be increased to speed-up the analysis.
4. **upsamp.grc**<br>
	As the really low frequencies are not audible, this flowgraph is made to upsample the low-frequent signals from the recordings and play it with a higher pitch
5. **intelow.grc**<br>
	This is still an experimental tool to integrate samples over time, as used in radio-astronomy but in this case it is applied to audio. The usecase is to increase the level of really weak signals.

### Dependencies to the software.
The entire toolkit is written in GNUradio 3.7 without external dependencies. It's advisable to install the alsa toolkit to select the correct audio-device and set recording gain.
`apt install gnuradio alsa-utils`

## Required hardware
Test have been done on serveral hardware-platforms. A moderate laptop is required for the graphical display of low_frequency.grc and playback.grc flowgraphs.
Headless recording works fine on a raspbery-pi zero (or zero-W) and better.
Care must be taken for the (usb)soundcard. 

### Soundcard
Two different USB-soundcards have been tested to work correctly:

* GeneralPlus GPD8102B [[5]]
* C-Media CM-108 [[6]]

Both audio-cards have a 16 bit, single-channel microphone input. Both devices are capable of sampling at 48KHz. The C-Media appears to suffer from EMI-signals from 1KHz and up. This is not an issue for inspecting low-frequent signals.

### Microphone
Test have been done with several microphones. While dynamic microphones have a bigger apperture, the output-impedance from the microphone is low and the input-impedance from the soundcard is high. The resulting signals appear to have less noise but the signals are weaker too.
Electret-microphones are cheap and small and work great for this. All the experiments are done with an electret-lapel-microphone.

   
## Selecting the right input
When sampling signals, it is critical to select the right input with the right settings. First connect de USB device with the microphone. With the alsa-utils all the devices can be displayed with a short description. The command: 

`aplay -L`

Shows all this information. Somewhere in this list the following 2 devices are displayed:

````
hw:CARD=AUDIO,DEV=0
	USB  AUDIO, USB Audio
	Direct hardware device without any conversions
plughw:CARD=AUDIO,DEV=0
    USB  AUDIO, USB Audio
    Hardware device with all software conversions
````

The difference between the `hw` and the `plughw` device is described in detail [[7]]. As all the measurements are based on raw sampling of the AD-converter, software conversions of the sampled data are not desired. Thus the `hw` devices delivers the most trustworthy signals.
Several soundcards support sample-rates for capuring audio. Most common rates are 44.1KHz and 48KHz. As this hardware rates are supported by all the tested cards and 48KHz is a standard sample-rate, the software is designed to work with 48KHz or 48000sps.
Always select a sample-rate of 48000 (48KHz) as the decimation is based on this input-rate.

### low2_rec.grc
First, compile the flowgraph to python using Gnuradio Companion or on the commandline using `grcc -d . low2_rec.grc`. Then use the commandline switch `-d hw:CARD=AUDIO,DEV=0 ` to select the right input device.

### low_frequency.grc
The audio-source block has the option to add the selected device in the line "Device Name"

## Running the software

Start the software in Gnuradio Companion or on the commandline. Below the graphs are 2 sliders, named *min_level* and *max_level*. Use these 2 sliders to optimize the dynamic range of the display.

Use `alsamixer -c 1` or the graphical soundcard interface to increase or decrease the recording level from the soundcard. Depending on the signal-levels the microphone-boost needs to be switched off.


---
[The Hum]: https://thehum.info/
[1]: https://www.ncbi.nlm.nih.gov/pubmed/16934315
[2]: http://www.sengpielaudio.com/calculator-dba-spl.htm
[3]: https://www.gnuradio.org/about/
[4]: http://microscopy.berkeley.edu/courses/dib/sections/02Images/sampling.html
[5]: https://www.generalplus.com/vLVfLNaSVtSNnormal_download
[6]: https://www.cmedia.com.tw/products/USB20_FULL_SPEED/CM108AH
[7]: https://www-uxsup.csx.cam.ac.uk/pub/doc/suse/suse9.0/userguide-9.0/ch18.html



