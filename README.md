# HyperGAN
A versatile GAN(generative adversarial network) implementation focused on scalability and ease-of-use.

# Table of contents

* <a href="#changelog">Changelog</a>

* <a href="#quickstart">Quick start</a>
  * <a href="#minreqs">Minimum Requirements</a>
  * <a href="#qs-install">Install</a>
  * <a href="#qs-train">Train</a>
  * <a href="#qs-increase">Increasing Performance</a>
  * <a href="#qs-devmode">Development Mode</a>
  * <a href="#qs-runoncpu">Running on CPU</a>

* <a href="#configuration">Configuration</a>
  * <a href="#configuration-usage">Usage</a>

* <a href="#cli">The pip package `hypergan`</a>
 * <a href="#cli-train">Training</a>
 * <a href="#cli-sample">Sampling</a>
 * <a href="#cli-serving">Web Server</a>

* <a href="#api">API</a>

* <a href="#datasets">Datasets</a>
 * <a href="#supervised-learning">Supervised learning</a>
 * <a href="#unsupervised-learning">Unsupervised learning</a>
 * <a href="#createdataset">Creating a Dataset</a>
 * <a href='#downloadabledatasets'>Downloadable Datasets</a>
 
* <a href="#about">About</a>
  * <a href="#wgan">WGAN</a>

 

<div id="changelog"></div>

## Changelog

## 0.7 - "WGAN API" (samples to come)

* New loss function based on `wgan` :.  Fixes many classes of mode collapse!  Our implementation is discussed in <a href="#wgan">wgan implementation</a>
* Initial Public API Release
* API example: `colorizer` - re-colorize an image!
* API example: `inpainter` - remove a section of an image and have your GAN repaint it
* API example: `super-resolution` - zoom in and enhance.  We've caught the bad guy!
* 4 *new* samplers.  `--sampler` flag.  Valid options are: `batch`,`progressive`,`static_batch`,`grid`. 

## 0.6 ~ "MultiGAN"

* 3 new encoders
* New discriminator: `densenet` - based loosely on https://arxiv.org/abs/1608.06993
* Updated discriminator: `pyramid_no_stride` - `conv` and `avg_pool` together
* New generator: `dense_resize_conv` - original type of generator that seems to work well
* Updated generator: `resize_conv` - standard resize-conv generator.  This works much better than `deconv`, which is not supported.
* Several quality of life improvements
* Support for multiple discriminators
* Support for discriminators on different image resolutions

## 0.5 ~ "FaceGAN"

### 0.5.x

* fixed configuration save/load
* cleaner cli output
* documentation cleanup

<img src='https://raw.githubusercontent.com/255BITS/HyperGAN/master/doc/face-manifold-0-5-6.png'/>

### 0.5.0
* pip package released!
* Better defaults.  Good variance.  256x256.  The broken images showed up after training for 5 days.

<img src='https://raw.githubusercontent.com/255BITS/HyperGAN/master/doc/face-manifold.png'/>

### 0.1-0.4
* Initial private release

<img src='https://raw.githubusercontent.com/255BITS/HyperGAN/master/doc/legacy-0.1.png'/>
<img src='https://raw.githubusercontent.com/255BITS/HyperGAN/master/doc/legacy-0.1-2.png'/>


<div id="samples"/>


<div id='quickstart'/>

# Quick start

<div id='minreqs'/>

## Minimum requirements

1. For 256x256, we recommend a GTX 1080 or better.  32x32 can be run on lower-end GPUs.
2. CPU mode is _extremely_ slow.  Never train with it!


<div id='qs-install'/>

## Install hypergan


### Installing 0.6(Experimental):

```bash
  pip install hypergan --upgrade
```

### Installing 0.5(Stable)
```bash
  pip install hypergan==0.5.8 --upgrade
```

<div id='qs-train'/>

## Train

```bash
  # Train a 32x32 gan with batch size 32 on a folder of pngs
  hypergan train [folder] -s 32x32x3 -f png -b 32
```

<div id='qs-increase'/>

### Increasing performance

On ubuntu `sudo apt-get install libgoogle-perftools4` and make sure to include this environment variable before training

```bash
  LD_PRELOAD="/usr/lib/libtcmalloc.so.4" hypergan train my_dataset
```


<div id='qs-devmode'/>
## Development mode

If you wish to modify hypergan

```bash
git clone https://github.com/255BITS/hypergan
cd hypergan
python3 setup.py develop
```

<div id='qs-runoncpu'/>
## Running on CPU

Make sure to include the following 2 arguments:

```bash
CUDA_VISIBLE_DEVICES= hypergan --device '/cpu:0'
```

<div id='configuration'/>

# Configuration

Configuration in HyperGAN uses JSON files.  You can create a new config by running `hypergan train`.  By default, configurations are randomly generated using [Hyperchamber](https://github.com/255BITS/hyperchamber).

Configurations are located in:

```bash
  ~/.hypergan/configs/
```

<div id='configuration-usage'/>

## Usage

```bash
  --config [name]
```

Naming a configuration during training is recommended.  If your config is not named, a uuid will be used.

<div id='#cli'/>
# CLI

```bash
 hypergan -h
```

<div id='#cli-train'/>
## Training

```bash
  # Train a 256x256 gan with batch size 32 on a folder of pngs
  hypergan train [folder] -s 32x32x3 -f png -b 32 --config [name]
```

<div id='#cli-sampling'/>
## Sampling

```bash
  # Train a 256x256 gan with batch size 32 on a folder of pngs
  hypergan train [folder] -s 32x32x3 -f png -b 32 --config [name] --sampler static_batch --sample_every 5
```

One way a network learns:

[![Demo CountPages alpha](https://j.gifs.com/58KmzA.gif)](https://www.youtube.com/watch?v=tj3ZLNfcJFo&list=PLWW3WtkBA3MuSnAVS__D0FkENZzuTbHFg&index=1)


To create videos:

```bash
  ffmpeg -i samples/%06d.png -vcodec libx264 -crf 22 -threads 0 gan.mp4
```

<div id='#cli-serving'/>
## Web Server

```bash
  # Train a 256x256 gan with batch size 32 on a folder of pngs
  hypergan serve [folder] -s 32x32x3 -f png -b 32 --config [name]
```

To prevent the GPU from allocating space, see <a href='#qs-runoncpu'>Running on CPU</a>.



<div id="api"/>
# API

```python3
  import hypergan as hg
```
API is currently under development.  The best reference are the examples in the `examples` directory.

<div id="datasets"/>
# Datasets

To build a new network you need a dataset.  Your data should be structured like:

``` 
  [folder]/[directory]/*.png
```

<div id="createdataset"/>
## Creating a Dataset

<div id='supervised-learning'/>

## Supervised learning

Training with labels allows you to train a `classifier`.

Each directory in your dataset represents a classification.  

Example:  Dataset setup for classification of apple and orange images:
```
 /dataset/apples
 /dataset/oranges
```

<div id='unsupervised-learning'/>

## Unsupervised learning

You can still build a GAN if your dataset is unlabelled.  Just make sure your folder is formatted like

```
 [folder]/[directory]/*.png
```
where all files are in 1 directory.

<div id='downloadabledatasets'/>

## Downloadable datasets

* CelebA aligned faces http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html
* MS Coco http://mscoco.org/
* ImageNet http://image-net.org/

# Building

## hypergan build

Build takes the same arguments as train and builds a generator.  It's required for serve.

Building does 2 things:

* Loads the training model, which include the discriminator
* Saves into a ckpt model containing only the generator

# Server mode

## hypergan serve

Serve starts a flask server.  You can then access:

[http://localhost:5000/sample.png?type=batch](http://localhost:5000/sample.png?type=batch)

## Saves

Saves are stored in `~/.hypergan/saves/`

They can be large.

## Formats

```bash
--format <type>
```

Type can be one of:
* jpg
* png

## Arguments

To see a detailed list, run 
```bash
  hypergan -h
```

* -s, --size, optional(default 64x64x3), the size of your data in the form 'width'x'height'x'channels'
* -f, --format, optional(default png), file format of the images.  Only supports jpg and png for now.


## Discriminators

The discriminators job is to tell if a piece of data is real or fake.  In hypergan, a discriminator can also be a classifier.

You can combine multiple discriminators in a single GAN. 

### pyramid_stride

### pyramid_nostride


Progressive enhancement is enabled by default:

<img src='https://raw.githubusercontent.com/255BITS/HyperGAN/master/doc/progressive-enhancement.png'/>

Default.

### densenet

Progressive enhancement is enabled by default here too.

### resnet

Note: This is currently broken 

## Encoders

### Vae

For Vae-GANs

### RandomCombo

Default

### RandomNormal

## Generators

### resize-conv

Standard resize-conv.

### dense-resize-conv

Default.  Inspired by densenet.

## Trainers

### Adam

Default.

### Slowdown

Experimental.

<div id='about'/>

# About

Generative Adversarial Networks(2) consist of (at least) two neural networks that learn together over many epochs.
The discriminator learns the difference between real and fake data.  The generator learns to create fake data.

For a more in-depth introduction, see here [http://blog.aylien.com/introduction-generative-adversarial-networks-code-tensorflow/](http://blog.aylien.com/introduction-generative-adversarial-networks-code-tensorflow/)

A single fully trained `GAN` consists of the following useful networks:

* `generator` - Generates content that fools the `discriminator`.  If using supervised learning mode, can generate data on a specific classification.
* `discriminator` - Gives a value between 0 and 1 designating how `real` the input data is.
* `classifier` - Only available with supervised learning mode.  Classifies an image by type.  Some examples of possible datasets are 'apple/orange', 'cat/dog/squirrel'.  See <a href='#createdataset'>Creating a Dataset</a>.

HyperGAN is currently in open beta.

## WGAN

Our implementation of WGAN is based off the paper.  Hypergan's loss function is now:

```python
 d_fake = tf.abs(d_fake)
 d_real = tf.abs(d_real)
 d_fake = tf.reduce_mean(d_fake,axis=1)
 d_real = tf.reduce_mean(d_real,axis=1)
 d_loss = d_real - d_fake
 g_loss = d_fake
```

Additionally we are using `Adam` and do not perform value clipping.

abs/reduce_mean may both be considered hyperparams.

## Papers

* GAN - https://arxiv.org/abs/1406.2661
* DCGAN - https://arxiv.org/abs/1511.06434
* InfoGAN - https://arxiv.org/abs/1606.03657
* Improved GAN - https://arxiv.org/abs/1606.03498
* Adversarial Inference - https://arxiv.org/abs/1606.00704
* WGAN - https://arxiv.org/abs/1701.07875

## Sources

* DCGAN - https://github.com/carpedm20/DCGAN-tensorflow
* InfoGAN - https://github.com/openai/InfoGAN
* Improved GAN - https://github.com/openai/improved-gan
* Hyperchamber - https://github.com/255bits/hyperchamber

# Contributing

Contributions are welcome and appreciated.  To help out, just issue a pull request or file a bug report.

If you create something cool with this let us know!

In case you are interested, our pivotal board is here: https://www.pivotaltracker.com/n/projects/1886395

# Citation

If you wish to cite this project, do so like this:

```
  255bits (M. Garcia),
  HyperGAN, (2017), 
  GitHub repository, 
  https://github.com/255BITS/HyperGAN
```

