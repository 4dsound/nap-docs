Writing python shapes {#pythonshapes}
=======================

# Introduction

This tutorial describes how to create and include a custom [PythonShape](@ref nap::spatial::PythonShape) in the 4DSOUND Engine. A [Shape](@ref nap::spatial::Shape) is a class that contains an algorithm which calculates transforms (positions, dimensions and orientations) of one or multiple [Particles](@ref nap::spatial::Particle) in real time. 

# Developing the Shape


A [PythonShape](@ref nap::spatial::PythonShape) is a [Shape](@ref nap::spatial::Shape) which is defined as a Python class. The Python class has to implement three functions: `__init__`, `getParticles`, and `update`.

## The 'init' function: intialising custom parameters

~~~{py}

    # ___init function___
    def __init__(self, entity, name, parameterManager):
        # store entity, name and parameter manager in class
        self.entity = entity
        self.name = name
        self.parameterManager = parameterManager

        self.waviness = self.parameterManager.addParameterFloat("waviness", 0., 0., 1., False) # amount of modulation of particle positions

~~~

The `init` function initialises the Python class. This is the place to create any custom parameters of the shape, using the functions of the [ParameterManager](@ref nap::spatial::ParameterManager).

For our example shape, we created a float parameter named 'waviness' that will control the amount of modulation of the particle positions. 

## The 'getParticles' function: calculating transforms in real-time


~~~{py}
    def getParticles(self, particleCount, speedAdjustedTime):

~~~

The `getParticles` function gets called on every control cycle while the shape is active. It should return a list of [SpatialTransforms](@ref nap::spatial::SpatialTransform) for the given desired particle count. 

SpatialTransforms have the following members:

- .Position (type nap.vec3): the position of the particle in meters
- .Scale (type nap.vec3): the dimensions of the particle in meters
- .Rotation (type nap.vec4): the orientation of the particle, in angle-axis format.


The input parameters of the function are:

- particleCount (int): indicates the desired number of particles. The amount of transforms returned should be around this number.

- speedAdjustedTime (float): Amount of time passed in "pseudo-seconds". The speed in which speedAdjustedTime increased is determined by the built-in `speed` parameter.

Note: returned particle transforms are in "object space", meaning that they will still be scaled, rotated and positioned in world space.


For our example shape, an algorithm is implemented here which position particles along a line over the x-axis. The particles' y-axis value is modulated based on a sine-wave function and the value of our custom `waviness` parameter:

~~~{py}
    def getParticles(self, particleCount, speedAdjustedTime):

        # Generate a line of particles with modulation of y-position based on a custom parameter.
        particles = []
        for i in range(particleCount):
            position = nap.vec3(-10. + 20. * (i / particleCount), 0, 0)
            positionOffset = self.waviness.Value * nap.vec3(0, math.sin(i + speedAdjustedTime), 0)
            dimensions = nap.vec3(0., 0., 0.)
            rotation = nap.vec4(0., 0., 1., 0.)
            particles.append(nap.SpatialTransform(position + positionOffset, dimensions, rotation))

        return particles
~~~

## The 'update' function

~~~{py}
    def update(self, deltaTime):
        pass
~~~

If you want to do certain actions based on the actual time passed in seconds, instead of the `speedAdjustedTime`, or when the shape is not selected, you can implement the `update` function. The update function is always called, even when the shape is not selected. `deltaTime` indicates the time passed since the last time update was called. 

In most cases, there is no reason to implement this function.




# Including the Shape






##Creating the PythonScript resource

To make NAP aware of the existence of an external file, it needs to be included in the project's data structure as a resource. In this case, we include our python script as a [PythonScript](@ref nap::PythonScript) resource.

To create a new PythonScript resource, follow the following steps:

- [Launch Napkin](@ref launching_napkin) and [open the 4DSOUND project](@ref napkin_project_management).


- Right click on the `Resources` item inside the resource panel
- Select `Create Resource`
- Select a `nap::PythonScript`

![](@ref content/4dsound_add_pythonscript.gif)

Now let's give it a readable name:

- Double click on the new resource to rename it.
- Change the name to `MyShapeScript`

And, let's make it refer to our python script file:

- Select the 	`MyShapeScript` resource.
- Inside the inspector panel: click on the `folder` incon next to `Path`
- Browse and select to the python file you want to include. 


##Creating the PythonShape resource

A PythonShape is a Shape which interfaces with an external python script. Now that the python script file is included as a resource, we can create a [PythonShape](@ref nap::spatial::PythonShape) resource, referring to the script.

- ([Launch Napkin](@ref launching_napkin) and [open the 4DSOUND project](@ref napkin_project_management).)
- Right click on the `Resources` item inside the resource panel
- Select `Create Resource`
- Select a `nap::spatial::PythonShape`

Let's give it a readable name:

- Double click on the new resource to rename it.
- Change the name to `MyShape`

The PythonShape resource has three `properties`:


1. `Name`: the name the shape is selected by. It also determines the prefix of the OSC addresses of all custom parameters of this shape.
2. `PythonScript`: a reference to the python script resource.
3. `Class`: the name of the class in the python script


Let's fill these in:

- Select the 	`MyShape` resource.
- Set `Name` to the name you want it to appear by.
- Set `Class` to the name of the python class (in this example: `Shape`).
- Click on the `link` icon next to `PythonScript` and select the `PythonScript` resource we just created.

![](@ref content/4dsound_pythonshape-resource.png)



##Adding the PythonShape to sound objects

Now that we have created the PythonShape, we can include it in our Sound Entities.

- Open the file `data/scripts/environment.py`.


- To make the Shape available to all 'sound entities' sound objects, add the following line to the function `createSoundEntities`:

~~~{py}
    shapes.append("MyShape")
~~~

Note: to make the Shape available to all 'source' sound objects as well, you can add the same line to the function `createSoundEntities`.


- Save the file `data/scripts/environment.py`.


# Controlling the Shape


After following all above steps, the new shape will now be available as an option for the `shapeType` parameter of the sound objects, under the name that you gave it. The custom parameters of the shape will be accessible via their respective OSC addresses (`/soundObjectName/shapeName/parameterName`). All available parameters are listed in the Parameter tab of the Control Panel.


![](@ref content/4dsound_pythonshape-in-action.gif)

The python shape in action, with a particle count of 25.


# Writing python 'transformations' and 'shape transformations'

It is also possible to create custom [Transformations](@ref nap::spatial::Transformation) and [ShapeTransformations](@ref nap::spatial::ShapeTransformation) in Python. The procedure is very similar as for writing and including python shapes. 

In Napkin, instead of creating a `PythonShape` resource, create a `PythonTransformation` or `PythonShapeTransformation` which links to a python script. 

In the environment script, transformations and shape transformations can be added using the lines `transformations.append("MyTransformation")` and `shapeTransformations.append("MyShapeTransformation")`.

Example scripts can be found in the `\data` folder.