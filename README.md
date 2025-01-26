# Classifying and Segmenting Pottery Sherds using the U-Net Architecture ![Version](https://img.shields.io/badge/version-1.0.0-blue)
Applying segmentation techniques to archaeological sherd classification for Terra Sigillata. 

*This is a project done at the University of Pennsylvania with the help of the classics department and Penn Museum. This is still an ongoing project and may have errors.*

## Table of Contents
* [Introduction](#Introduction)
* [Setup](#Setup)
* [Model Architecture](#model-architecture)

## Introduction

In archaeological excavations, the process of recording and classifying artifacts is a repetitive and time consuming task that requires specialists to manually document and draw each artifact. The fragmentary condition of most remains makes the process even more difficult. Faced with a limited amount of time and resources, archaeologists often compensate by collecting only proportions of the artifacts and recording limited characteristics. This can lead to incomplete datasets with poor data quality.   

The goal of this project is to build a reliable and versatile machine learning model that can classify excavated artifacts from photos taken from phones. Since taking photos with phones is not a specialized task, large datasets can be recorded with minimal time and resources.

The dataset used is provided by the Museum of London. The sherds dataset includes 5278 total RGB images of pottery sherds taken through a phone on a variety of different angles. 

## Setup

##### a. Load dataset folder 

Click open the files icon on the left bar in Google Colab. Drag and drop the dataset folder into the files. Wait for it to load. 

##### b. Configure hyperparameters 

In the 'hyperparameters' section, there are several hypermaraters that can be configured. 

* epoch_num -> number of epochs
* patience -> patience for early stopping (# of consecutive epochs where validation loss doesn't improve before training is broken)
* val_interval -> frequency that the model is evaluated
* alpha -> weighting for dice loss
* beta -> weighting for cross-entropy (classification) loss
* batch_size in train_dataloder -> training batch size
* batch_size in val_dataloader -> validation batch size
* lr in optimizer -> learning rate

Hyperparameters for the model architecture: 

* encoder_name -> what encoder model we are using(usually set to resnet34).
* encoder_weights -> weight initialization(can be set to either 'ImageNet' or 'Xavier').

For dataset size:

* The '[:some_number]' in the creating mask section -> cap on the number of samples per class

##### c. Run the code

Click 'Runtime' -> 'Run all'. 

## Model Architecture

For the purpose of classifying pottery sherds, I’ve decided to implement a custom model based on the UNet architecture to both segment and classify the sherds. This is chosen because in a real world setting where the background is noisy, having the ability to recognize the specific shape of the fragment and texture and correlate it to a class makes the model more robust to potential noise and irregularities. More importantly, I believe that the model will make a more accurate prediction if it is able to accurately segment the region of interest from the surrounding content. By using a shared latent space (learned representation/information) for both the segmentation and classification, I’m training the model to be able to extract features that would allow it to be proficient in both objectives. 


Another way to think about it is that training the model to both segment and classify is like pretraining the model on segmentation and then fine-tuning it on classification. It is a technique that works well with low resource datasets. 


The UNet architecture is commonly used in biological image segmentation as a robust architecture that operates with high accuracies on low datasets. It operates by having an encoder progressively reduce the dimensions of the input signal (image) while extracting features and a decoder progressive upsample the dimensions of the feature map to create the desired output, whether that is a segmentation map or a classification output. 

![Unet Image](./unet.ong)

In order to tailor the UNet model to our specific use-case, I’ve decided to add a separate classification branch. It will diverge at the feature map level (lowest level on the image) and upsample the feature map before passing it through a Sigmoid activation function to generate a classification output. Having this extra classification branch means that the loss function will be a combination of both cross-entropy loss for classification and dice-loss for segmentation. The weighting between them can be adjusted with hyperparameters. 

The complete pipeline of the model is shown below: 
The input image will have dimensions 128x128x3 (three channel RGB). 
The input image will be compressed into a feature map - first as a 64x64x27 feature map and then down to a 1024x1 latent representation. 
From here, two branches will be created:
For the segmentation branch, the latent representation will be upsampled until it reaches one channel, which would then be the segmentation map. 
For the classification branch, I will pass it through a series of fully connected dense regression layers. There will be nonlinearity introduced with ReLU. The final layer will consist of a softmax function that would classify the image. 
To do so, I will combine the dice loss for the segmentation task with the cross entropy loss for the classification task. 
This model will be trained with labeled data and masks. 


A weighted sampler is also implemented to ensure that classes with smaller numbers of data are sampled just as often as those classes with more data. This is added to address the dataset imbalance. 

## References
* Ronneberger, Olaf, Philipp Fischer, and Thomas Brox. "U-net: Convolutional networks for biomedical image segmentation." In Medical image computing and computer-assisted intervention–MICCAI 2015: 18th international conference, Munich, Germany, October 5-9, 2015, proceedings, part III 18, pp. 234-241. Springer International Publishing, 2015.
* Anichini, F. et al. 2020 Developing the ArchAIDE Application: A digital workflow for identifying, organising and sharing archaeological pottery using automated image recognition, Internet Archaeology 52. https://doi.org/10.11141/ia.52.7
* D. van Helden, E. Mirkes, I. Tyukin and P. Allison, "The arch-i-scan project: Artificial intelligence and 3d simulation for developing new approaches to roman foodways", Journal of Computer Applications in Archaeology, Aug 2022.


