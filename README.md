# time-domain
# What it does

The TimeDomainRecord class extends the View class used in Android Applications. It is used to dynamically display audio data during the recording process. The amplitude over time graph is displayed in 5 second intervals and will clear at the end of the 5 seconds.

# How it works

For an Android application, audio can be recorded with an AudioRecord object. This object can record PCM audio using a buffer and store that raw audio data in a .pcm file. During the part of the recording process where the data is being read from the buffer and output to the file, we will also send a sample average of that data to the TimeDomainRecord object to analyze and display in real time.

If the recording stops, we stop and reset the display of the data on the TimeDomainRecord object and get it ready for use again.

# PCM Audio Overview

**PCM Audio Recording**

PCM Audio is the backbone of the TimeDomainRecord object and it's worth taking some time to understand it if the you do not already.

PCM stands for Pulse Code Modulation and is a format for encoding an audio waveform in the time domain as a series of amplitudes. In laymans terms, this means that the values stored represent loudness, while the length of the file is equal to the length of the recording. 

There are many different formats that can be used to change the percision of the recording, but for our purposes we will be discussing Single Channel (Mono), 16-Bit PCM Audio, recorded at a Sample Rate of 44100Hz. The reason for this is that this is relatively simple to understand but still gives us a high precision audio recording.

To create the AudioRecord object we use:

`AudioRecord record = new AudioRecord(MediaRecorder.AudioSource.MIC, SAMPLE_RATE, CHANNEL_IN_CONFIG, ENCODING_TYPE, minBufferSize);`

**ENCODING_TYPE** in our case will be equal to AudioFormat.ENCODING_PCM_16BIT. 16-Bit PCM Audio stores a 16 bit percision audio amplitude value per sample. This gives your recording 2^16 (or 65,536) possible values to use for amplitude. In contrast, consider 8-Bit PCM audio, which gives you just 2^8 (or 256) possible values to use for amplitude. This lower value of percision can be manipulated slightly to keep audio standards high, but especially at lower sample rates will usually result in rough, digital sounding audio that is not ideal unless you are trying to save memory.

**SAMPLE_RATE** is equal, in our case to 44100. This means that 44,100 samples are taken every second. Each sample of 16-Bit PCM Audio contains 16 bits of data.

The diagram below shows the structure of 

![PCM16Diagram](https://github.com/axellarsstenson/time-domain/blob/main/images/PCMDiagram.png)

**CHANNEL_IN_CONFIG** will be AudioFormat.CHANNEL_IN_MONO. This means that all audio played back will come out at the same amplitude. For example, if I am wearing headphones, both ears will hear the same thing. Audio Record also allows for multiple channel recording, but for our purposes, we only will be using single channel, or Mono audio. Two channels of 16-Bit Audio would mean we have 16 * 2 = 32 Bits of audio per sample.

**minBufferSize**

Because we need to not only record the audio, but put write it to a file, we need to use a buffer. A buffer is a data structure that is filled with some subsection of data from one data structure, writes that subsection to something (in our case a file), and then is cleared and does it again. This means that we will be writing our information to the file one chunk of information at a time. The size of that chunk is the buffer size.

The AudioRecord object handles buffering with some internal methods.

We can get the length of the minimum buffer needed to record audio by using:

`int minBufferSize = AudioRecord.getMinBufferSize(SAMPLE_RATE, CHANNEL_IN_CONFIG, ENCODING_TYPE);`

This will be the size of our buffer and represents the number of samples that will be added to the buffer before recording them to the .pcm file. This size is given to us by the AudioRecord object, and in our case the buffer size is 3528.

The **total memory** used for a recording depends on three different options, mentioned above, and can be calculated as follows:

Sample Rate * Bit Rate * Number of Channels = Bits Per Second of Recording Time. 

For our example above, we have a Sample Rate of 44100Hz, a Bit Rate of 16, and 1 Channel so we have 44100 * 16 * 1 = 705600 bits per second, 705.7 kbits/s or 88.2 kb/s. This means that approximately every 11 (1000 / 88.2 = 11.34) seconds, we will record 1 Mb of audio data.

# How to use

**Recording to File** 

Somewhere in the class that is using the TimeDomainRecord, before recording, we will need to set up an OutputStream using:

`File recordFile = new File(fileName);
 recordFile.createNewFile();

 OutputStream outputStream = new FileOutputStream(recordFile);
 BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(outputStream);
 DataOutputStream dataOutputStream = new DataOutputStream(bufferedOutputStream);`
 
 And then start recording to our AudioRecord object with:

`record.startRecording();`

We now need to set up the buffer to use to move data between the AudioRecord object and our OutputStream.

Since we are using 16-Bit Audio, the easiest way to handle the samples will be to use a data type that is also 16-bit, which is why we will be using the Short data type.

Declare the audioData array

`short[] audioData = new short[minBufferSize];`

and then use

`int numberOfShortRead = record.read(audioData, 0, minBufferSize);`

to read the data into the audioData array. numberOfShortRead will be the number of shorts added to audioData and will be 0 when there is no longer data to read from the AudioRecord object.

After filling a buffer's worth of data we need to write that data to a file. All we need to do here is loop through the buffer and use

`dataOutputStream.writeShort(audioData[i]);`

to write each short into the file.

**Sending Data to TimeDomainRecord**

Since the percision of the recording we are doing is so high (44100 samples per second), we will need to create an average value for some period that will be used to display that period's average amplitude on our graph.

To do this, we include an averaging function in the `for` loop that loops through the buffer:

```
double sum = 0;

for (shortsWritten = 0; shorts < numberofShortsRead; shortsWritten++) {
   dataOutputStream.writeShort(audioData[i]);
   sum += audioData[i] * audioData[i];

   if (numberOfShortRead > 0) {
     final double amplitude = sum / shortsDividedByFour;
     squareAmp = (float) Math.sqrt(amplitude);
     timeDomainRecord.addLine(squareAmp);
     }

     sum = 0;
}
```

This will average the amplitude over a period of time equal to the Sample Rate / minBufferSize and then add that to the TimeDomainRecord view. As we mentioned above, our buffer is of size 3528 and our sample rate is 44100, so we get 12.5 lines per second. 1 line is equal to 0.08 seconds.

Since this is to be used in a Mobile app, we will have a variety of different display sizes. In order to calculate the width of vertical amplitude bars that will fit within our display we will need to get the viewWidth. Hypothetically, lets say our device has a width of 500. While we can theoretically display 500 vertical bars, we are restricted in percision by the size of our buffer. We can actually display a number of vertical bars equal to sampleRate/minBufferSize * numberOfSeconds which is 12.5 * 5 = 62.5 or 62. We have 62 bars and a widget of 500, so each bar will be 500/62 = 8 across.

We could make this more percise by averaging and sending parts of the buffer, instead of the whole thing, but the intent of this exercise is to stay simple for ease of understanding.

Since the AudioRecord object returns the same value for minBufferSize if the variables entered are the same, we have run these calculations inside of our View as well. This means that we have set our AmplitudePaint to size = 8, and created the Short array with the correct size of 62.

Once data has been added to the Short array inside the TimeDomainRecord object, we call invalidate() from inside the View, which forces the View to be redrawn, and the amplitude graph bar will be drawn, now with a non-zero value for the first column.

 
