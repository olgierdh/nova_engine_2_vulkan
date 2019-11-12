# General idea of talk 

This talk should present in a simple and easy to digest form how we ported Nova engine to Vulkan API. Basic assumption is that I'm not going to present any detail from OpenGL or Vulkan world. I'm going to create an abstraction in order to build intuition of differences between OpenGL and Vulkan. In addition to that I want to present the whole process of getting to the final solution. 

# Chapters

## I - Building abstract model of OpenGL and Vulkan APIs 

* What is state machine ? 
* OpenGL as a global state machine 
* Vulkan as a local named states 

## II - Idea of exection model for OpenGL and Vulkan APIs

* OpenGL uses swap and executes whole "command chain"
* Vulkan uses idea of Command Buffer 
* Show the differences using some simple example i.e. Clear Color or draw triangle

## III - Idea of abstract command buffer 

* Works for all modern APIs
* Can use it on OpenGL as an emulation layer
* Show it on same example with sample implementation in pseudocode

## IV - Implementation of abstract command buffer using C++ and Variant type

* Show custom Variant implementation ? 
* Show our approach to generic device abstraction
* Show how it supports the abstract command buffer idea

## V - Summary 

* Results - OpenGL with abstract command buffer faster than previous implementation
* Abstraction cost can be low if used properly

## The End
Thank you

