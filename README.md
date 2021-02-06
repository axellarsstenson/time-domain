# time-domai
# What it does

The TimeDomainRecord class extends the View class used in Android Applications. It is used to dynamically display audio data during the recording process. The amplitude over time graph is displayed in 5 second intervals and will clear at the end of the 5 seconds.

# How it works (Overview)

For an Android application, audio can be recorded with an AudioRecord object. This object can record PCM audio using a buffer and store that raw audio data in a .pcm file. During the part of the recording process where the data is being read from the buffer and output to the file, we will also send a sample average of that data to the TimeDomainRecord object to analyze and display in real time.

If the recording stops, we stop and reset the display of the data on the TimeDomainRecord object and get it ready for use again.

# How it works (Detail)

PCM Audio Recording

PCM Audio is the backbone of the TimeDomainRecord object and it's worth taking some time to understand it if the you do not already.

PCM stands for Pulse Code Modulation and is a format for encoding an audio waveform in the time domain as a series of amplitudes. There are many different formats that can be used to change the percision of the recording, but for our purposes we will be discussing Single Channel (Mono), 16-Bit PCM Audio, recorded at a Sample Rate of 44100Hz. The reason for this is that this is relatively simple to understand but still gives us a high precision audio recording.




# How to use

 
