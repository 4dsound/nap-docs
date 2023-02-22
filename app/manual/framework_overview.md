Spatial sound framework overview
===

Module architecture
---
The spatial sound framework is written in C++ using the [NAP framework](https://nap.tech). Some knowledge about NAP might be useful to understand this manual. Furthermore the spatial sound framework makes heavy use of the [napaudioadvanced](https://github.com/stijnvanbeek/napaudioadvanced) module for audio processing implementations.


Below is a diagram that shows all the modules required to build the 4DSOUND engine, in which repositories they reside and how they depend on one another. Also see the [build instructions](@ref build_instructions) for more information on how to clone and deploy the required repositories.

![4DSOUND module architecture](/images/4dsound_modules.png)

In this chapter we will focus mostly on the two modules that are highlighted by the blue line in the diagram, as they contain the heart of the spatial sound framework: *spatialcore* and *spatialsoundobject*.  

Spatial core module
---
This module contains the core of the spatial sound framework. Its main task is providing an interface for panning sound entities consisting of sound particles in space. The main classes are the [`SpatialService`](@ref nap::spatial::SpatialService), the [`SpatializationComponent`](@ref nap::spatial::SpatializationComponent), [`Particle`](@ref nap::spatial::Particle) and the [`SpeakerSetup`](@ref nap::spatial::SpeakerSetup) baseclass.

**[SpatializationComponent](@ref nap::spatial::SpatializationComponent)**

A spatial sound entity is a NAP [`Entity`](@ref nap::Entity) that contains a [`SpatializationComponent`](@ref nap::spatial::SpatializationComponent). The SpatializationComponent keeps track of all the particles within the sound object. A [`Particle`](@ref nap::spatial::Particle) is one single audio signal along with its position in space. In other words, a spatial sound entity can be seen (or rather heard) as a cluster of sound particles in space. The particles are administered by the [`SpatializationComponent`](@ref nap::spatial::SpatializationComponent).

**[SpeakerSetup](@ref nap::spatial::SpeakerSetup)**

The [`SpeakerSetup`](@ref nap::spatial::SpeakerSetup) is an object that keeps track of all sound particles that exist within the system. Its main task is taking care that all the active particles will be projected onto the speaker system with the right panning that corresponeds to their positioning in space.
[`SpeakerSetup`](@ref nap::spatial::SpeakerSetup) is a base class that has to be subclassed in order to create a working speaker setup. Current implementations of speaker setups are [`HeadphonesSetup`](@ref nap::spatial::HeadphonesSetup) and [`MultiSpeakerSetup`](@ref nap::Spatial::MultiSpeakerSetup). 

**[SpatialService](@ref nap::spatial::SpatialService)**

The [`SpatialService`](@ref nap::spatial::SpatialService) is a NAP [`Service`](@ref nap::Service) that keeps track of all the spatial sound entities that exist within the system. It does so by internally keeping a list with references to all the `SpatializationComponents` of the existing sound entities. 

Apart from all the [`SpatializationComponents`](@ref nap::spatial::SpatializationComponent) the [`SpatialService`](@ref nap::spatial::SpatialService) also keeps track of the [`SpeakerSetups`](@ref nap::spatial::SpeakerSetup), and makes sure the particles managed by the [`SpatializationComponents`](@ref nap::spatial::SpatializationComponent) are registered with the [`SpeakerSetups`](@ref nap::spatial::SpeakerSetup) so they will be played back through the speakers with the proper panning for their positioning.

Spatial sound object module
---
The spatial sound object module provides a more refined and elaborate implementation of a sound entity called "spatial sound object". The spatial sound object, apart from a SpatializationComponent exposing sound particles in space, adds another collection of NAP components to the sound entity that provide a framework for the following characteristics of the sound object: 
- a sound source and a DSP effect processing chain for each particle's audio signal
- a shape algorithm that generates positions for the sound object's particles in space

**[SpatialAudioComponent](@ref nap::spatial::SpatialAudioComponent)**

This component provides a container for the audio sources and effect processing chains to generate the audio signals for each particle in the sound object. The audio sources managed by the SpatialAudioComponent are objects derived from the class [`SpatialSource`](@ref nap::spatial::SpatialSource). Currently three types of spatial sources exist:
- [ExternalInputSource](@ref nap::spatial::ExternalInputSource) receives audio input from a number of consecutive audio device input channels.
- [SoundObjectSource](@ref nap::spatial::SoundObjectSource) receives audio input form another sound object, also known as a "receive" input in 4DSOUND. Internally the audio outputw of all the active particles of the other sound object are mixed together in its [`MixdownComponent`](@ref nap::spatial::MixdownComponent) and passed as input to the `SoundObjectSource`.
- [TestSource](@ref nap::spatial::TestSource) a source that just outputs a test tone to test the sound object.

`SpatialAudioComponent` has methods in order to add sources: `addSource` and `addInputSource`.

Apart from sound sources, the `SpatialAudioComponent` manages three effect chains to process the input from the sound sources:
- The input effect chain contains effects that are applied to each of the spatial source's particle outputs separately.
- The regular effect chain is applied on a mix of the output of these separate input effect chains. The output of this effect chain serves as the input for the [`MixdownComponent`](@ref nap::spatial::MixdownComponent)
- The perception effect chain is applied to the output of the regular effect chain for each particle before it is sent to the [`SpatializationComponene`](@ref nap::spatial::SpatializationComponent) for distribution the the [`SpeakerSetup`](@ref nap::spatial::SpeakerSetup)

`SpatialAudioComponent` has methods in order to add effects to its effect chains: `addInputEffect` and `addEffect` and `addPerceptionEffect`.

**[ShapeComponent](@ref nap::spatial::ShapeComponent)**
...

**[SpatialTransformationComponent](@ref nap::spatial::SpatialTransformationComponent)**
...

