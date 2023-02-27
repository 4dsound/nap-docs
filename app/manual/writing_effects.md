Writing spatial sound effects {#effects}
===

Introduction
===
This is a tutorial to show developers how to write their own spatial sound effects that work within the 4DSOUND system. So what exactly is a spatial sound effect? Within the spatial sound framework, an effect is defined as a single block of DSP processing within the processing chain of a sound object. The DSP and its parameters can have unique properties for each particle within the sound object. 
As an example of a spatial sound effect we will write a feedback delay of which the delay time is controlled by the distance from the particle to the vantage point.

Architecture of a spatial effect
===
A spatial effect is defined in C++ by three classes that work together to describe the behaviour of the effect. The [Effect](@ref nap::spatial::Effect), the [EffectInstance](@ref nap::spatial::EffectInstance) and The [EffectProcessor](@ref nap::spatial::EffectProcessor). Because Effect and EffectProcessor classes usually are not very large and because all three classes' roles are strongly entangled it is advised to define them together in a single header/cpp file dedicated to the effect.

In order to define a spatial effect, include the following headers in your effect's header:

~~~cpp
// Spatial includes
#include <Spatial/SpatialAudio/Effect.h>
#include <Spatial/SpatialAudio/EffectProcessor.h>
~~~

Effect
---
The [Effect](@ref nap::spatial::Effect) is the resource of the effect that is used to deserialize the effect or to instantiate it in the environment script. This can be compared to the [Component](@ref nap::Component) resource. The Effect resource usually doesn't have many properties except sometimes for some metadata about the effect like maximum boundaries for things like buffersizes. It exists because it can be deserialized or created from script and it knows what EffectInstance to instantiate.

Here is the definition for our own Effect resource (should be enclosed within the nap::spatial namespace):
~~~cpp
// Forward declaration of the effect instance
class MyDelayEffectInstance;

// Declaration of the effect resource
class NAPAPI MyDelayEffect : public Effect<MyDelayEffectInstance
{ 
    RTTI_ENABLE(EffectBase) 
};
~~~

EffectInstance
---
the [EffectInstance](@ref nap::spatial::EffectInstance) represents a runtime instance of the effect, similarly to a [ComponentInstance](@ref nap::ComponentInstance) for components. The EffectInstance initializes the effect's [parameters](@ref nap::Parameter) and its internal variables in the init() method and updates the DSP processing of the effect over time according to the state of the parameters in the update() method:
~~~cpp
/**
 * Instance of the delay effect
 * This is the class that manages parameters and controls the DSP for each particle
 */
class NAPAPI MyDelayEffectInstance : public EffectInstance<MyDelayEffectProcessor> {
    RTTI_ENABLE(EffectInstanceBase)
public:
    MyDelayEffectInstance(EffectBase& effect, EntityInstance& entity, int channelCount) : EffectInstance(effect, entity, channelCount) {}
    bool init(utility::ErrorState& errorState) override;

private:
    void update(double deltaTime) override;

    ParameterFloat *mFeedback = nullptr;                // Feedback parameter
    ParameterFloat *mDelayTimeMultiplier = nullptr;     // Multplier parameter for the delay time
};
~~~

Note that `MyDelayEffectInstance` is derived from `EffectInstance<MyEffectProcessor>`, which is a templated class with the type of the EffectProcessor as template argument. In order to be able to define this template the EffectProcessor (which is described below) needs to be defined before the EffectInstance in the header.

The two parameters of the effect are created in the `MyDelayEffectInstance::init()` method like this:
~~~cpp
bool MyDelayEffectInstance::init(utility::ErrorState& errorState)
{
    // Define the parameters for the effect
    mDelayTimeMultiplier = getParameterManager().addParameterFloat("delayTimeMultiplier", 1.f, 0.f, 100.f);
    mFeedback = getParameterManager().addParameterFloat("feedback", 0.1f, 0.f, 1.f);
    return true;
}
~~~

The definition of our example delay effect's `MyDelayEffectInstance::update()` method looks like this:
~~~cpp
void MyDelayEffectInstance::update(double deltaTime)
{
    for (auto& processor : getProcessors())
        for (auto particle = 0; particle < getActiveParticleCount(); ++particle)
        {
            // Calculate the delay time from the distance to the vantage point
            auto distance = getParticleMeasurer(particle)->getDistanceToVantagePoint();
            auto delayTime = (distance / 343.f) * mDelayTimeMultiplier->mValue * 1000.f;

            // Set delay time and feedback for the particle
            auto delayNode = processor->getNode(particle);
            delayNode->setTime(delayTime, 50.f);
            delayNode->setFeedback(mFeedback->mValue);
        }
}
~~~
Note how we iterate through the effect instance's effect processors. Mostly the EffectInstance will only contain one single EffectProcessor. However when we are dealing with a so called "input effect", the instance will manage multiple processors: one for each [SpatialSource](@ref nap::spatial::SpatialSource) connected to the sound entity.

For each EffectProcessor we iterate through its active particles, grab the corresponding audio [Node](@ref nap::audio::Node) and update it using the current values of the effect's parameters. We use a [ParticleMeasurer](@ref nap::spatial::ParticleMeasurer) to acquire the distance to the vantage point.

EffectProcessor
---
The [EffectProcessor](@ref nap::spatial::EffectProcessor) defines the DSP processing of the effect. The EffectProcessor is owned and controlled by the EffectInstance. The EffectProcessor has multiple channels of processing, one for each particle.

Here is the header of our example effect's EffectProcessor:
~~~cpp
/**
 * Effect processor of the delay effect.
 * This class takes care of the DSP processing.
 */
class NAPAPI MyDelayEffectProcessor : public ParallelNodeEffectProcessor<audio::DelayNode>
{
    RTTI_ENABLE(EffectProcessor)
public:
    // Inherited constructor from base class
    using ParallelNodeEffectProcessor<audio::DelayNode>::ParallelNodeEffectProcessor;
};
~~~
Note that this class has no need to define any methods of its own, because it's derived from the templated base class ParallelNodeEffectProcessor. This class takes as template argument an audio [Node](@ref nap::audio::Node) type. This Node is required to only have a single audio input and output because it is supposed to implement the effect processing for one single particle.

When one single Node is not enough to implement the audio processing for a single particle for this effect, it is possible to derive an effect processor from the base class [EffectProcessor](@ref EffectProcessor) directly, and overwrite its method `EffectProcessor::createAudioObject()` that returns an [AudioObjectInstance](@ref nap::audio::AudioObjectInstance) that can implement a more complicated DSP containing multiple nodes.

Embedding the effect in a spatial sound object
===
RTTI definitions
---
In order to be able to use the effect in a sound object the Effect resource, the EffectInstance and the EffectProcessor types have to be rtti defined, so they will be recognized by the json deserializer and within the environment script. Rtti definitions are placed in the cpp file outside the namespaces:
~~~cpp
// Rtti define the effect resource
RTTI_DEFINE_CLASS(nap::spatial::MyDelayEffect)

// Rtti define the effect processor
RTTI_BEGIN_CLASS_NO_DEFAULT_CONSTRUCTOR(nap::spatial::MyDelayEffectProcessor)
RTTI_END_CLASS

// Rtti define the effect instance
RTTI_BEGIN_CLASS_NO_DEFAULT_CONSTRUCTOR(nap::spatial::MyDelayEffectInstance)
RTTI_END_CLASS
~~~
Add effect type to application
---
In order for the effect type to be recognized by the environment script we need to add the Effect resource to the app structure. This can be done using napkin by opening the `project.json` file and adding a resource of the type `nap::spatial::MyDelayEffect`. The `mID` property will be used to refer to the effect in the environment script and the `name` property will be used for the OSC tag to control the effect's parameters.

Add effect to sound object's effect chain
---
The last step is te add our effect in the effect chain of a sound object. The effect chains of the different types of sound objects (sources, sound entities, spaces) are defined in the `environment.py` script in de `data` directory of the 4dsound application. In respectively the methods `createSources()`, `createSoundEntities()` or `createSpaces()` we add the following line:
~~~
effects.append("MyDelayEffect")
~~~
Add this line in between two similar lines for the effects in between which we want to place our new effect within the effect processing chain. 

Once we have done this we should be able to test and use our effect. Run the engine, open the parameters tab en search for the string we entered for the `Name` property of the effect in napkin. This should filter out the parameters defined for the new effect.
