---
layout: post
title:  "The Basics of Machine Learning without Math"
date:   2022-01-07 19:58:49 +0100
categories: machine learning
---


## Table of Contents
1. How is a neuronal network structured
2. What is a neuron
3. What is Backpropagation
4. How can we use this knowledge in practice


## How is a neuronal network structured
For many people a neuronal network is magic. If you never learnt what happens behind the scenes it looks like you just throw things in and eventually it can solve a problem.

A neuronal Network, as the name already suggests, is a network of mutliple neurons.


<figure>
  <img src="1200px-Neural_network.svg.png" width="600" alt="Simplified representation of a Neural Network with two green input nodes, five blue intermediate nodes and one yellow output node." />
  <figcaption>
    A simplified representation of a neural network with two layers.
    <a href="https://commons.wikimedia.org/wiki/File:Neural_network.svg">(by Dake, Mysid. CC-BY 1.0)</a></figcaption>
</figure>




You give a input into it and through the interaction of the different neurons the result is formed.

## What is a Neuron

As you could see in the image above there are 3 types of neurons.
The green ones are input Neurons. The just have connections to their right and no other connections. These are used to input data into the neuronal network. <br>
The second type are the blue ones. How many collumns of them exist is not given and can be changed. They have connections to their left (input) and to their right (output)
They take the inputs, process them and then generate an output. <br>
The last type of neurons are the yellow one. They are at the end of the network. Their Output is the output of the whole network. <br>


All 3 types of the network can be descriped through one model

<img src="1*O7YSSqlOdQuNgMrH7J_3dg.png" width="600"><br>
https://miro.medium.com/max/428/1*O7YSSqlOdQuNgMrH7J_3dg.png<br><br>

A neuron has multiple inputs. The inputs are the outputs of all neurons in the previous collum.
These inputs get weights. For example if a input is 0.6 and the weight is 0.1 we take 0.6 * 0.1 and then we have the "real" input. Every input has its own weight. All of the inputs multiplied with the weigth are added together. This sum is now put into a function. The function determines how much output is generated for the given sum. Often the function looks like this:

<img src="1*oePAhrm74RNnNEolprmTaQ.png" width="600"><br>
https://miro.medium.com/max/357/1*oePAhrm74RNnNEolprmTaQ.png<br><br>
The output of this function (in this case RELU) is the output of the neuron and therefore the input for the next collum of neurons.<br>
If we have a description of the arrangement of the neurons and a list of all weights we can reproduce the network over and over with the same outputs for the same inputs. And these two informations are the core of a neuronal network. How the Neurons are arrenged and what the weights are. 

## What is Backpropagation
The structure of a network impacts its perfomance but the best variable to shape it are the weights. If we get the right weights its should be perfect. Obviously getting good weights is hard. I know i said this is without math but we need a little bit. A neuronal network is like a function with millions of parameters(weights) which produces a deterministic result. We can imagen this like a 3D map. 


<img src="gradient-descent-local-minima.png" width="600"><br>
https://bdtechtalks.com/wp-content/uploads/2020/04/gradient-descent-local-minima.png <br><br>
The lower we get the better our network performs. Finding the lowest point overall (Global Minima) is at the moment impossible. We take a other approach. We start at a random point somewhere. So we just use random weights. Luckily we have a way to find the way which leads us from our point. This is called negative gradiant. If you are intereseted in math you can research this on your own. <br>
Now lets start learning. We guess some weights and start throwing inputs into the network. We compare the result of the network with the right results. Then we use the negative gradiant and adjust the weights with these metrics. This process is repeated thousand or even millions of times until we find a local minima. If we reach it we cant get out and have to take a look how our network performs. If the accurecy of the network is enough we are finished. When the result are not good enough we have to start again with other starting weights and let the network learn.

## How can we use this Knowledge in Practice

A good and easy way to start is to use pytorch 
https://pytorch.org/
This library allows us to build and train a network with nearly zero prior knowledge. So what are you waiting for?

<img src="have-a-good-day-cats.gif">