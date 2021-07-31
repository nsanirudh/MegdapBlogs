# 1.Litreature and Useful info

## Phonetics (Article 1 Feature Extraction) 

	* Two types of sounds
		* Voiced \b\
		* Voiceless \th\
	* A syllable usually contains a vowel sound with or without consonants
	* Consonants are sounds that are articulated with a complete or partial closure of the vocal tract. It can be voiced or voiceless
	* Vowels are voiced sounds
	* Phonemes: The same letter in words can be pronounced differently
	* American English has approx. 44 phonemes 
	* The more successful speech recognizers ignore most we learn from phonetics or linguistics

#### Machine Learning in play

	* There are two main types of machine learning modeling that we can use
		* Generative modeling 
		* Discriminative modeling 
		* Link : nips01-discriminativegenerative.pdf (stanford.edu)
		* Link for joint and conditional probability : Joint Probability vs Conditional Probability | by Prathap Manohar Joshi | Medium
	* We extract features from the frequency domain, that means we apply a Fourier transform
	* The X in our network would be extracted feature vectors from the audio waveform

English speaking people know about 20K to 50K words. If we use words as W, the space for W will be unnecessarily large. In addition, English is not a phonetic language. Letters are not always pronounced the same in different words.
A less ambiguous connection with the pronunciation will be a bonus.
So what are the alternatives? Phones are more fundamental than words in speech.
In addition, many corpora are already phonetic transcripted or such transcription can be done automatically with pronunciation tables. Therefore, our acoustic model will be phone-based rather than word-based.

## Feature Extraction

To extract audio features, we use sliding windows of width 25ms and 10ms apart to parse the audio waveform. For each sliding window, we extract a frame of audio signals.
We apply Fourier Transform, manipulate it to make the perceived speech features stand out. Then we apply the inverse Fourier Transform. In the end, we extract 39 MFCC features for each frame (details later).
For speech recognition, we will use this feature vector to represent the audio signal.

## Notebooks for Dataset Analysis in Mozilla TTS

    AnalyzeDataset
    Check Spectograms
    CheckDataset SNR
    PhonemeCoverage

* Trained a test model on 600+ Hindi Data  files

* Analyse model output stats, analyse model performance, evaluate model decoding.

* Datasets to be solved on: 
  * IIT-Madras
  * ABP

---------------------------

# 2.Training Pipeline (Steps) 

## Mozilla TTS:

links:

* https://stackoverflow.com/questions/66307611/how-do-i-get-started-training-a-custom-voice-model-with-mozilla-tts-on-ubuntu-20
* https://github.com/mozilla/TTS/wiki/FAQ

#### 2.1 Can the Mozilla TTS Docker image be used for training (TL;DR: "No")

```buildoutcfg
 The Mozilla TTS docker image is really geared for playback and doesn't seem equipped to be used for training. At least,
 even when running a shell inside the container, I could not get training to work. But after figuring out what was 
 causing PIP to be unhappy, the process of getting Mozilla TTS up and running in Ubuntu turns out to be pretty 
 straightforward.
```

##### 2.2 Installing the env:

```commandline
sudo apt-get install espeak
git clone https://github.com/mozilla/TTS mozilla-tts
python3 -m venv mozilla-tts

cd mozilla-tts
./bin/pip install -e .
```

##### 2.3 Run everything from the TTS directory.

#### 2.4 Training process:

  * Short audio recordings (at least 100) that are:
    * In 16-bit, mono PCM WAV format.
    * Between 1 and 10 seconds each.
    * Have a sample rate of 22050 Hz.
    * Have a minimum of background noise and distortion.
    * Have no long pauses of silence at the beginning, throughout the middle, and at the end.

  * A metadata.csv file that references each WAV file and indicates what text is spoken in the WAV file.

  * A configuration file tailored to your data set and chosen vocoder (e.g. Tacotron, WavGrad, etc).

  * A machine with a fast CPU (ideally an nVidia GPU with CUDA support and at least 12 GB of GPU RAM; you cannot 
    effectively use CUDA if you have less than 8 GB OF GPU RAM).

## Prepare the data:

* Mozilla TTS supports several different data loaders, but one of the most common is LJSpeech. To use it,
  we can organize our data set to follow LJSpeech conventions.

```yaml
- metadata.csv
- wavs/
  - audio1.wav
  - audio2.wav
  ...
  - last_audio.wav
```

* The naming of the audio files doesn't appear to be significant. But, the files must be in a folder called wavs.
  You can use sub-folders inside wavs though, if so desired.
  The metadata.csv file should be in the following format:

```yaml
audio1|line that's spoken in the first file
audio2|line that's spoken in the second file
last_audio|line that's spoken in the last file
```

* There is no header line.
* The columns are joined together with a pipe symbol (|).
* There should be one row per WAV file.
* The WAV filename is in the first column, without the wavs/ folder prefix, and without the .wav suffix.
* The textual description of what's spoken in the WAV is written out in the second column, with all numbers and abbreviations spelled-out.

## Preparing the config.json File

To start, create a copy of the default Tacotron config.json file from the Mozilla repo. Then,
be sure to customize at least the audio.stats_path, output_path, phoneme_cache_path, and datasets.path file.


At minimum:

* audio.stats_path
* output_path
* phoneme_cache_path
* datasets.path

## Preparing the scale_stats.npy (Feature Extraction):

Most of the training configurations rely on a statistics file called scale_stats.npy 
that's generated based on the training set. You can use the ./TTS/bin/compute_statistics.py script inside the 
Mozilla TTS repo to generate this file. This script requires your config.json file as an input, 
and is a good step to sanity check that everything looks good up to this point.

code:

```bash
python ./TTS/bin/compute_statistics.py --config_path /path/to/your/project/config.json --out_path /path/to/your/project/scale_stats.npy
```

## training the model:

Code:

```bash
python ./TTS/bin/train_tacotron.py --config_path /path/to/your/project/config.json
```

--------------

# 3.Experiments Carried out:

carried out in AMD Ryzen Epyc VM with Telsa T4 16Gb variant.

## Experiment A

Step 1: Set path

```python
transcript_path = "/home/Megdap/SpeechData/HindiTTS/trans/"audio_path = "/home/Megdap/SpeechData/HindiTTS/wav/"
```

step 2: Verify every transcript with audio file:

```python
audio_names = []trn_names = []#First Step is to verify the audio to the relevant transcriptsfor i in range(len(os.listdir(transcript_path))):    filenames_trn = os.listdir(transcript_path)    cur_trn = os.path.splitext(filenames_trn[i])[0]    trn_names.append(cur_trn)        filenames_audio = os.listdir(audio_path)    cur_audio = os.path.splitext(filenames_audio[i])[0]    audio_names.append(cur_audio)  if (set(trn_names)==set(audio_names)):    print("The Audio files match the transcriptions")    
```

step 3: Create a single metadata.csv file for the transcriptions:

```python
#Create a single metadata.csv file for the transcriptionsfrom os.path import basename, splitextimport pandas as pd#Check your cwdprint(os.getcwd())# Create a metadata.csv file with filename and transcriptionfor file_ in os.listdir(transcript_path):        with open(transcript_path + '/' + file_) as f:            textf = os.path.splitext(file_)[0] + '|' + " ".join(line.strip() for line in f)            with open((transcript_path + '/' + 'metadata.csv'),'a') as f1:                f1.write(textf+"\n")                print("metadata.csv has been created!")
```

step 4: 

* Create scale_stats.npy and train the model:
* If you get a module not found error, what should do is add the python path to the directory with the files:

```bash
export PYTHONPATH="${PYTHONPATH}:/home/Megdap/TTS/TTS"
```

set to 300 epochs

## Experiment B

Experiments with IIIT-H data.

```python
transcript_path = "/home/Megdap/SpeechData/HindiTTS-IITM/metadata.csv"audio_path = "/home/Megdap/SpeechData/HindiTTS-IITM/wavs/"
```


Data-Cleaning:

* Certain transcriptions did not have the relevant audio files.(Removed them)
* Audio files were .mp3 files which are not supported by librosa and PyAudio
* converted the audio to .wav using ffmpeg
* sox does not support .mp3 files as well

Create:

* scale_stats.npy
* config.json file

Train:

* train_tacotron.py with the config, but we are getting a cuda OOM error even after reducing the batch size to 8.

