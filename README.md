# time-domain
# What it does

The TimeDomainRecord class extends the View class used in Android Applications. It is used to dynamically display audio data during the recording process. The amplitude over time graph is displayed in 5 second intervals and will clear at the end of the 5 seconds.

# How it works (Overview)

For an Android application, audio can be recorded with an AudioRecord object. This object can record PCM audio using a buffer and store that raw audio data in a .pcm file. During the part of the recording process where the data is being read from the buffer and output to the file, we will also send a sample average of that data to the TimeDomainRecord object to analyze and display in real time.

If the recording stops, we stop and reset the display of the data on the TimeDomainRecord object and get it ready for use again.

# How it works (Detail)

**PCM Audio Recording**

PCM Audio is the backbone of the TimeDomainRecord object and it's worth taking some time to understand it if the you do not already.

PCM stands for Pulse Code Modulation and is a format for encoding an audio waveform in the time domain as a series of amplitudes. There are many different formats that can be used to change the percision of the recording, but for our purposes we will be discussing Single Channel (Mono), 16-Bit PCM Audio, recorded at a Sample Rate of 44100Hz. The reason for this is that this is relatively simple to understand but still gives us a high precision audio recording.

**16-Bit PCM Audio** stores a 16 bit percision audio amplitude value per sample. This gives your recording 2^16 (or 65,536) possible values to use for amplitude. In contrast, consider 8-Bit PCM audio, which gives you just 2^8 (or 256) possible values to use for amplitude. This lower value of percision can be manipulated slightly to keep audio standards high, but especially at lower sample rates will usually result in rough, digital sounding audio that is not ideal unless you are trying to save memory.

**44100Hz** means that 44,100 samples are taken every second. Each sample of 16-Bit PCM Audio contains 16 bits of data.

![PCM16Diagram](https://github.com/axellarsstenson/time-domain/blob/main/images/PCMDiagram.png)

**Single Channel** means that all audio played back will come out at the same amplitude. For example, if I am wearing headphones, both ears will hear the same thing. Audio Record also allows for multiple channel recording, but for our purposes, we only will be using single channel, or Mono audio.

The **total memory** used for a recording depends on three different options, mentioned above, and can be calculated as follows:

Sample Rate * Bit Rate * Number of Channels = Bits Per Second of Recording Time. 

For our example above, we have a Sample Rate of 44100Hz, a Bit Rate of 16, and 1 Channel so we have 44100 * 16 * 1 = 705600 bits per second, 705.7 kbits/s or 88.2 kb/s. This means that approximately every 11 (1000 / 88.2 = 11.34) seconds, we will record 1 Mb of audio data.


# How to use

 
