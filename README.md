SoundNet
========

Code for paper "SoundNet: Learning Sound Representations from Unlabeled Video" by Yusuf Aytar, Carl Vondrick, Antonio Torralba. NIPS 2016

We learn rich natural sound representations by capitalizing on large amounts of unlabeled sound data collected in the wild. We leverage the natural synchronization between vision and sound to learn an acoustic representation using two-million unlabeled videos. We propose a student-teacher training procedure which transfers discriminative visual knowledge from well established visual models (e.g. ImageNet and PlacesCNN) into the sound modality using unlabeled video as a bridge.

<img src='http://web.mit.edu/vondrick/soundnet/soundnet.jpg'>

Requirements
============
 - torch7
 - torch7 audio (and sox)
 - torch7 hdf5 (only for feature extraction)
 - probably a GPU
 
Pretrained Model
================
We provide pre-trained models that are trained over 2,000,000 unlabeled videos. You can download the 8 layer and 5 layer models [here](https://drive.google.com/file/d/0B-xMJ5CYz_F9S09HU0ZKd3EtWnc/view?usp=sharing). We recommend 

Feature Extraction
==================

Using our network, you can extract good features for natural sounds. You can use our provided script to extract features. First, create a text file where each line lists an audio file you wish to process. We use MP3 files, but most audio formats should be supported. Then, extract features into HDF5 files like so:

```bash
$ list=data.txt th extract_feat.lua
```

where `data.txt` is this text file. It will write HDF5 files to the location of the input files with the features. You can then do anything you wish with these features. 
 
By default, it will extract the `conv7` layer. You can extract other layers like so:
 
```bash
$ list=data.txt layer=24 th extract_feat.lua
````
 
Advanced
--------
 
 If you want to write your own feature extraction code, it is very easy in Torch:

```lua
net = torch.load('soundnet8_final.t7')

sound = audio.load('file.mp3')

if sound:size(2) > 1 then sound = sound:select(2,1) end -- select first channel (mono)
sound:mul(2^-23)                                        -- make range [-256, 256]
sound = sound:view(1, 1, -1, 1)                         -- shape to BatchSize x 1 x DIM x 1
sound = sound:cuda()                                    -- ship to GPU
sound = sound:view(1, 1, -1, 1):cuda()

net:forward(sound)
feat = net.modules[24].output:float()
```

Training
========

The code for training is in `main_train.lua`, which will train the 8 layer SoundNet model. You can also use `main_train_small.lua` to train the 5 layer SoundNet model. To start training, just do:

```bash
$ CUDA_VISIBLE_DEVICES=0 th main_train.lua
```

The code for loading the data is in `data/donkey_audio.lua`. The training code will launch several threads. Each thread reads a different subset of the dataset. It will read MP3 files into a raw waveform. For the labels, it reads a large binary file that stores the class probabilities computed from ImageNet and Places networks.

If you want to fine-tune SoundNet on your own dataset, you can use `main_finetune.lua`.

Data
====

We plan to release 2 million MP3s and their corresponding class probabilities soon. Stay tuned.

References
==========

If you use SoundNet in your research, please cite our paper:

    Learning Sound Representations from Unlabeled Video 
    Yusuf Aytar, Carl Vondrick, Antonio Torralba
    NIPS 2016
