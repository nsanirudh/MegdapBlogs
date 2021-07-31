# Hello World of ASR

ASR stands for Automatic Speech Recognition. Which essentially means taking a continuous audio speech signal
and converts it into it's equivalent text. This is an introductory blog around the process of performing ASR
using the workflow as would be in a kaldi environment.

What we need:

* Lots of audio files
* All their corresponding transcripts.

Factors that impact a good ASR engine are(Specific to Audio Input):

* Volume
* Number of speakers
* Pitch
* Silences
* Word sped
* Background Noise

# Why do we need ASR

# What do we need to create an ASR Engine 

In kaldi the workflow used to build an ASR engine is, an Acoustic model, a Language model and Lexicon model.
The acoustic model would help us understand the audio signal, whereas the language model would help us predict the
next word in a sequence. The lexicon model is a pronunciation model.

The primary objective of speech recognition is to build a statistical model to infer the text sequences 
(say “cat sits on a mat”) from a sequence of feature vectors.

There is some golden content to be found in phonetics and linguistics, but regardless ASR is about finding the most 
likely word sequence given an audio and train these probability models with the provided transcripts.

The basic idea around building an ASR engine would revolve around understanding the speech signal. We need to 
represent the audio file (.wav or .flac) into it's corresponding audio signal and extract features from it. That
would involve applying a series of mathematical operations to extract features. MFCC analysis or Mel Frequency Cepstrum 
Coefficient Analysis is the conversion of that audio signal to the essential speech features required to train an 
acoustic model. A spectrogram is the conversion of an audio signal into the frequency domain using the fourier transform. 

In ASR, understanding how we hear is more valuable than understanding how we speak in feature extraction.

# Acoustic, Lexicon & Language Model:

![ASR](resources/ASR.JPG "ASR")



