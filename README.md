# jao-specification
The specification for the jao file format.


<a name="what"/></a>
# What is JAO?
JAO stands for Json Animated Objects. It is a specification for defining event-driven, animated UI components using a JSON file. This specification was created initially for use in game development, but can probably be used for other purposes.

# Table of Contents
- [What is JAO?](#what)
- [How it works](#howworks)
- [Why?](#why)
- [Specification - File Format](#file)
  - [Specification - JSON file](#json)
  - [Layers](#layers)
  - [Layer](#layer)
  - [Data Type Specification](#datatype)
  - [Events](#events)
  - [Event](#event)
  - [The `initialize` event](#initialize)
  - [Action](#action)
    - [Action Libraries](#actionlib)
    - [The `when` field](#whenfield)
    - [`initialize` actions](#initializeaction)
- [Initialization and Reset](#initreset)
  - [Problems with Reset](#probreset)
- [Time Expressions](#timeexp)
- [Interpolation parameters](#interpolation)
- [Displaying text](#text)
- [Real-Time vs. Frame-Based](#reatimeframebased)
- [Examples](#examples)
- [FAQ](#faq)
- [Use Case: Rushbeat](#usecase)

<a name="howworks"/></a>
# How it works
Consider the following UI component from a game menu:

![example](https://user-images.githubusercontent.com/6846892/152474221-be3966b1-66a5-43f3-92fb-03fb9de99753.png)

You may want it to have a few animations that are triggered when events happen. An example is when the screen is loaded:

![example1](https://user-images.githubusercontent.com/6846892/152474515-5b574e3e-b840-41f0-8819-410640cb594b.gif)

Then, you may want to have an animation triggered when you select the component

![example2](https://user-images.githubusercontent.com/6846892/152474666-3dfc1d63-b468-4002-9bc6-ac81e5b4b951.gif)

Finally, an animation when the item is unselected:

![example3](https://user-images.githubusercontent.com/6846892/152474742-34cc7cc2-faf8-44aa-8a2a-0bf12abf125e.gif)

Usually, you may need to use animation tools to create such animations or write a lot of code to create more complex animations. The JAO file format aims at the creation of such UI component animation items reducing the amount of code used and allowing for reusability of those components.

All animations you have seen above are a single jao file with three events (init, select, unselect) and 8 layers (7 images and one sound sample you will not hear here).

<a name="why"/></a>
# Why?
Defining the visual behaviour of UI components outside of your code can help with a few things:
- Decouple the logical behavior (code & logic) from the visual behavior (graphics & animation) of the UI component;
- Allow for the development and test of the visual behavior of UI components before integrating them into your code, leading to a quick develop-test-fix cycle;
- Preview & tweaking of the UI of a game without having to run the game by using a visual editor/visualizer.

Menus & Gameplay graphics have extensively used the JAO library. Every graphical component (except by the widgets in the settings screen) have been drawn and animated using JAO files (even music/sound is played back and defined inside the JAO files).

<a name="file"/></a>
# Specification - File Format
A jao file follows this structure:
```
root
┠─ jao.json
┠─ sample.png
┖─ images
   ┖─ another_sample.png
```

1. The root can be either a filesystem folder or a zip file, with the extension changed to `.jao`.
2. A file named `jao.json` has to be placed at the root folder. This file contains all the description of the events and animations.
3. You can have files (like images) at either the root folder or inside nested folders. All paths inside the `.jao` file are relative to the root folder.

**Notes**:
- For production usage, it is better to use the `.jao` file. The filesystem folder is useful for development, when you are tweaking the component.
- The file formats for the resources inside the `.jao` file do not have an specified type. The supported types are dependent on the implementation.

<a name="json"/></a>
# Specification - JSON file
The file `jao.json` will be a plain JSON file, following this format (schema to be defined):
```
{
   "layers": [
      {
         "dataType": {
            "type": "...",
            "attributes": {
               "sampleAttr": "...",
               ...
            }
         },
         "events": [
            {
               "name": "initialize",
               "actions": [
                  {
                     "library": "...",
                     "name": "...",
                     ...
                  }
               ]
            },
            ...
         ]
      },
      ...
   ]
}
```
You can see above a brief example of what a `jao.json` looks like.

<a name="layers"/></a>
## Layers
The jao file is organized in layers. Each layer is some kind of element in your animation (ex.: a `.png` image). You can have zero or more layers in your jao file.
```
"layers": [
   {
      "dataType": {
         "type": "...",
         "attributes": {
            "sampleAttr": "...",
            ...
         }
      },
      "events": [
         {
            "name": "initialize",
            "actions": [
               {
                  "library": "...",
                  "name": "...",
                  ...
               }
            ]
         },
         ...
      ]
   },
   ...
]
```

<a name="layer"/></a>
## Layer
The layer is this object inside the `"layers"` list:
```
{
   "dataType": {
      "type": "...",
      "attributes": {
         "sampleAttr": "...",
         ...
      }
   },
   "events": [
      {
         "name": "initialize",
         "actions": [
            {
               "library": "...",
               "name": "...",
               ...
            }
         ]
      },
      ...
   ]
}
```
A layer is composed by a data type specification and a list of events. Layers are rendered from the first (bottom) to the last (top).

<a name="datatype"/></a>
### Data Type
```
"dataType": {
   "type": "...",
   "attributes": {
      "sampleAttr": "...",
      ...
   }
}
```

Each layer inside the list of layers has to represent soem kind of entity. An example is an image (like a `.png` file), but that can be anything: images, sounds and even pure event-control layers. This is up to the implementation library to define.

Also, a data type specification can carry a list of attributes to define the overall behavior of a data type. For example, if your data type is something that carries an image (like a `sprite` or `animation`), you may want an attribute to define the file path of the file to be used (see examples below). The defition of attributes is up to the implementation, there is no standard or specification about it.

<a name="events"/></a>
### Events
Each layer carries a list of events, it can have zero or more events in this list.
```
"events": [
   {
      "name": "initialize",
      "actions": [
         {
            "library": "...",
            "name": "...",
            ...
         }
      ]
   },
   {
      "name": "select",
      "actions": [
         {
            "library": "...",
            "name": "...",
            "when": "..."
         }
      ]
   },
   ...
]
```
<a name="event"/></a>
### Event
```
{
   "name": "select",
   "actions": [
      {
         "library": "...",
         "name": "...",
         "when": "..."
      }
   ]
},
```
An event defines the behaviour of this layer when a given event starts. Imagine a menu option:
- It will have an event called `select`, that may trigger the animations to show the UI component selected in the screen, like a highlight;
- It will have an event called `unselect`, that may trigger the animations to remove the highlight from the UI component.

The event will describe *what* happens when this event is triggered, which is a list of actions. An event can have zero or more actions.

As each layer has a list of events, if you want to have multiple layers reacting to the activation of the same event, each layer must have an event with the same name (see examples below).

<a name="initialize"/></a>
#### The `initialize` event
```
{
   "name": "initialize",
   "actions": [
      {
         "library": "...",
         "name": "...",
         ...
      }
   ]
}
```
Events can have any name, except by `initialize`. This is a reserved event name for the initialization of the layer. You may want to define attributes of the layer when it is initialized -- ex.: set the opacity of a layer to 0 in a layer that will fade into the screen. For those cases you will want to use actions inside the `initialize` event to setup the initial state of the layer.

Actions used in the `initialize` event are also special actions, as they are not time-driven (more about this below).

The `initialize` event is optional and can be suppressed, you just need to add it in case you have to initialize things before running the other events.


<a name="action"/></a>
### Action
```
{
   "library": "...",
   "name": "...",
   "when": "...",
   "atributes": {
      "target_x": 200,
      ...
   }
}
```
Actions describe discrete actions to be executed on top of the layer data. They define *what* has to be done and *when* it will be done. An example for a layer that has an image:
- You can translate the image in the X axis;
- You can set the opacity of the image;
- You can scale the image across the Y axis.

Those are examples of actions that can be added to an event.

Actions will always have:
- A `name` field: it defines which action to be executed.
- A `library` field: defines which action library to look for an action with the name provided (more below).
- A `when` field: defines when the action will run, considering that the moment of the activation of the event is 0.

Aside from this, it can also have a list of attributes that will define the behavior of the action (ex.: which position in the X axis should the image be translated to).

<a name="actionlib"/></a>
#### Action Libraries
Each action has a name and a library. The idea is that the user should be able to import multiple libraries of actions inside a single project, even if actions have the same name across different libraries.

<a name="whenfield"/></a>
#### The `when` field
Each action has a `when` field that will define when the action has to be executed. This field has to be expressed as a time expression (more below), assuming that time is 0 when the event is triggered.

<a name="initializeaction"/></a>
#### `initialize` actions
```
{
   "library": "...",
   "name": "...",
   "atributes": {
      "target_x": 200,
      ...
   }
}
```
As it happens with events, there are special types of actions for the `initialize` event. Actions used in the `initialize` event are different in the sense that they will not have the `when` field as they are triggered right when the layer is initialized or reset (more below).

While it is expected that the regular actions will execute actions interpolated across a given amount of time (more below), `initialize` actions will be executed imediately, right when the jao file is loaded (or when it is reset).

<a name="initreset"/></a>
# Initialization and Reset
It is expected that the jao object will execute all of its initialize events before other events are triggered. When *exactly* it happens is up to the implementation. 

When an event is triggered, it is expected that a timer (or clock) will be attached to this event, so that the actions listed inside this event are executed across the time by following the time expressions inside the `when` field. It is also expected that the implementation allows a jao object to be *reset*. By reseting it means this:
- All actions inside the `initialize` event are executed.
- The event clock will be reset to 0 (ex.: actions with `when` set to 0 will run again).

<a name="probreset"/></a>
## Problems with Reset
It is expected that for some workflows it will be inconvenient to execute the `initialize` event actions again when the object is reset. In such cases, it is advisable to add an attribute in the `dataType` that will define if the element is resetable:
```
"dataType": {
   "type": "...",
   "attributes": {
      "resetable": false
   }
}
```
The code of each action can look at this attribute and decide if it should reset or not when asked to do it so.

<a name="timeexp"/></a>
# Time Expressions
Time-based fields are expected to suport time expressions:
```
<type of time> <number>
```

`<type of time>` is expected to accept at least `second` as a value and it should accept a floating point number as a `<number>` (ex.: `4.2`). Some examples of other (probably) valid values:
- `second 0`
- `minute 1`
- `second 2.5`
- `milisecond 200`
- `frame 10`

Those expressions are provided as strings inside the JSON:
```
{
   "library": "...",
   "name": "...",
   "when": "second 0",
   "atributes": {
      ...
   }
}
```

Whether the implementation will support an specific type of time or not is up to the developer. At least `second` has to be supported.

<a name="interpolation"/></a>
# Interpolation parameters
It is expected that actions will most times execute over a time period. For that reason, most actions may have an attribute `duration`. This is an example:
```
{
    "library": "jao.standards",
    "name": "FadeOverTime",
    "when": "second 2.76",
    "attributes": {
        "start_opacity": "1",
        "end_opacity": "0",
        "duration": "seconds 4.62"
    }
}
```
Fields other than `when` that define time (like the `duration` in the example above) are expected to support time expressions as well.

<a name="text"/></a>
# Displaying text
The jao specification does not define *how* you will be displaying text on the screen, but it is advisable that you support using labels to identify text layers so you can set text by referring to labels (the Java implementation does that).

<a name="reatimeframebased"/></a>
# Real-Time vs. Frame-Based
The jao specification does not define if the implementation of the library is done using real-time (using threads, for example) or are frame-based (calling the render routine every cycle). Still, interms of *behavior*, it is expected that a *second means a second* when it runs. You can add frame-based timing to your implementation using time expressions that define duration in frames, but it is expected that you can define durations and start/end times as seconds of fractions/multiples of seconds.

<a name="examples"/></a>
# Examples
Examples below are extracted from the Rushbeat standard skin and use the library implemented in Java.

## The structure of a jao file
A standard jao file may look like this:

![image](https://user-images.githubusercontent.com/6846892/152470811-e19cb8dc-b772-41e0-ae3f-10e1bc19e3b8.png)

Notice there is no specified hierarchy for the files -- everything you need is the `jao.json` at the root of the jao folder.

## Example of a background layer with fade-in
This is the most basic real world example, a background image that fades into the screen over the time of one second:
```
[
   "layers": {
      {
         "dataType": {
             "type": "sprite",
             "attributes": {
                 "path": "background.png"
             }
         },
         "events": [
             {
                 "name": "initialize",
                 "actions": [
                     {
                         "library": "jao.standards",
                         "name": "Position",
                         "attributes": {
                             "position_x": "0",
                             "position_y": "0"
                         }
                     },
                     {
                         "library": "jao.standards",
                         "name": "Opacity",
                         "attribute": "0"
                     }
                 ]
             },
             {
                 "name": "init",
                 "actions": [
                     {
                         "library": "jao.standards",
                         "name": "FadeOverTime",
                         "when": "second 0",
                         "attributes": {
                             "start_opacity": "0",
                             "end_opacity": "1",
                             "duration": "second 1"
                         }
                     }
                 ]
             }
         ]
      }
   }
]
```

### Breakdown
This layer displays an image, that can be found at the root of the jao file with the name `background.png`:
```
"dataType": {
    "type": "sprite",
    "attributes": {
        "path": "background.png"
    }
}
```

Inside the list of events we have the `initialize` action. It will make sure the image is positioned at `0,0` and the opacity is set to `0`:
```
{
  "name": "initialize",
  "actions": [
      {
          "library": "jao.standards",
          "name": "Position",
          "attributes": {
              "position_x": "0",
              "position_y": "0"
          }
      },
      {
          "library": "jao.standards",
          "name": "Opacity",
          "attribute": "0"
      }
  ]
}
```
The actions above will execute *before* the other events start running.

Then, we have the `fadein` event that will execute the fade-in:
```
{
  "name": "fadein",
  "actions": [
      {
          "library": "jao.standards",
          "name": "FadeOverTime",
          "when": "second 0",
          "attributes": {
              "start_opacity": "0",
              "end_opacity": "1",
              "duration": "second 1"
          }
      }
  ]
}
```
In the snippet above, you can see the name of the event is `fadein`. This action will execute right when the event is triggered because `when` is `second 0`, and it will set the opacity of the `background.png` being displayed by this layer over one second (because `duration` is `second 1`). The opacity will begin from `0` and should end at `1` after one second (in real time).

That means you will probably want to do something like this in your code (*warning, pseudocode*):
```
...
background = JaoLoader.load("./background.jao")
background.set_event("fadein")

while True:
   background.render()
...
```

## Example of a jao element with a text layer
You can implement layers that can display textm in such cases the Java implementation suport using labels. It also shows the usage of different attributes in the `dataType` definition:
```
{
    "layers": [
        {
            "dataType": {
                "type": "simple_bitmap_text",
                "attributes": {
                    "path": "font_song_name.fnt",
                    "align": "center",
                    "textid": "songname",
                }
            },
            "events": [
                {
                    "name": "initialize",
                    "actions": [
                        {
                            "library": "jao.standards",
                            "name": "Position",
                            "attributes": {
                                "position_x": "0",
                                "position_y": "0"
                            }
                        },
                        {
                            "library": "jao.standards",
                            "name": "Opacity",
                            "attribute": "0"
                        }
                    ]
                },
                {
                    "name": "update",
                    "actions": [
                        {
                            "library": "jao.standards",
                            "name": "FadeOverTime",
                            "when": "second 0",
                            "attributes": {
                                "start_opacity": "current",
                                "end_opacity": "1",
                                "duration": "second 0.2"
                            }
                        },
                        {
                            "library": "jao.standards",
                            "name": "SetPosition",
                            "when": "second 0",
                            "attributes": {
                                "position_x": "100",
                                "position_y": "0"
                            }
                        },
                        {
                            "library": "jao.standards",
                            "name": "TranslateOverTime",
                            "when": "second 0",
                            "attributes": {
                                "target_position_x": "0",
                                "target_position_y": "0",
                                "duration": "second 0.2"
                            }
                        }
                    ]
                }
            ]
        }
    ]
}
```

### Breakdown
This the `dataType` for displaying a simple bitmap text (using `.fnt` bitmap fonts) in the screen. Aside from the `path`, you can see other parameters can be provided as well:
```
"dataType": {
    "type": "simple_bitmap_text",
    "attributes": {
        "path": "font_song_name.fnt",
        "align": "center",
        "textid": "songname",
    }
}
```
The parameter `center` is obvious, but the `textId` has something interesting: by using this kind of argument you can store all references to the layers that have text and you are able to set the text from those layers as follows (*warning, pseudocode*):
```
background = JaoLoader.load("./background.jao")
background.set_text("this is some text", "songname)
```

With this approach, you can have many layers pointing to the same label and multiple layers with different text values in the same json.

You can see in the `update` event the following action:
```
{
    "library": "jao.standards",
    "name": "SetPosition",
    "when": "second 0",
    "attributes": {
        "position_x": "100",
        "position_y": "0"
    }
}
```
This action is an example of an action that does *not* rely on any time interpolation. This action is supposed to be executed as a one-shot action: once the `when` time is reached, it sets the position of the text and that is it. Not every action needs to execute something over time, but it needs to have a `when` attribute to define when it will be executed.

## Example of playing sounds
You may also want to play sounds alongside with your animation events. Again, the capability of playing sounds is defined by the implementation of the library:
```
{
   "layers": [
      {
         "dataType": {
             "type": "sample",
             "attributes": {
                 "path": "title.ogg",
                 "loop": true
             }
         },
         "events": [
             {
                 "name": "init",
                 "actions": [
                     {
                         "library": "jao.standards",
                         "name": "PlaySample",
                         "when": "second 0"
                     }
                 ]
             },
             {
                 "name": "transition",
                 "actions": [
                     {
                         "library": "jao.standards",
                         "name": "StopSample",
                         "when": "second 0"
                     }
                 ]
             }
         ]
     }
   ]
}
```

### Breakdown
The layer dataType can be something that is not visual -- this one defines a sample, for example:
```
"dataType": {
    "type": "sample",
    "attributes": {
        "path": "title.ogg",
        "loop": true
    }
}
```

Controlling the sample works the same way by using events, the implementation of the action will define what happens:
```
{
     "name": "init",
     "actions": [
         {
             "library": "jao.standards",
             "name": "PlaySample",
             "when": "second 0"
         }
     ]
 }
```
Here the `init` event will start the playback of the sample at `second 0` and will stop when the `transition` event is reached:
```
{
  "name": "transition",
  "actions": [
      {
          "library": "jao.standards",
          "name": "StopSample",
          "when": "second 0"
      }
  ]
}
```
Keep in mind that some kinds of actions may need to have start and stop actions, as they may happen in parallel with the rest of the code (ex.: when playing a background song).

## A file with multiple layers
You can have multiple layers just by adding on after the other inside the `layers` list:
```
{
    "layers": [
        {
            "dataType": {
                "type": "sprite",
                "attributes": {
                    "path": "background.png"
                }
            },
            "events": [
                {
                    "name": "initialize",
                    "actions": [
                        {
                            "library": "jao.standards",
                            "name": "Position",
                            "attributes": {
                                "position_x": "0",
                                "position_y": "0"
                            }
                        },
                        {
                            "library": "jao.standards",
                            "name": "Opacity",
                            "attribute": "0"
                        }
                    ]
                },
                {
                    "name": "init",
                    "actions": [
                        {
                            "library": "jao.standards",
                            "name": "FadeOverTime",
                            "when": "second 0",
                            "attributes": {
                                "start_opacity": "0",
                                "end_opacity": "1",
                                "duration": "second 1"
                            }
                        }
                    ]
                },
                {
                    "name": "leave",
                    "actions": [
                        {
                            "library": "jao.standards",
                            "name": "FadeOverTime",
                            "when": "second 0",
                            "attributes": {
                                "start_opacity": "1",
                                "end_opacity": "0",
                                "duration": "second 3"
                            }
                        }
                    ]
                }
            ]
        },
        {
            "dataType": {
                "type": "sprite",
                "attributes": {
                    "path": "overlay.png"
                }
            },
            "events": [
                {
                    "name": "initialize",
                    "actions": [
                        {
                            "library": "jao.standards",
                            "name": "Position",
                            "attributes": {
                                "position_x": "0",
                                "position_y": "0"
                            }
                        },
                        {
                            "library": "jao.standards",
                            "name": "Opacity",
                            "attribute": "0"
                        }
                    ]
                },
                {
                    "name": "init",
                    "actions": [
                        {
                            "library": "jao.standards",
                            "name": "SetScale",
                            "when": "second 0",
                            "attributes": {
                                "scale_x": "1.5",
                                "scale_y": "1.5"
                            }
                        },
                        {
                            "library": "jao.standards",
                            "name": "ScaleOverTime",
                            "when": "second 0",
                            "attributes": {
                                "target_scale_x": "1",
                                "target_scale_y": "1",
                                "duration": "seconds 60"
                            }
                        },
                        {
                            "library": "jao.standards",
                            "name": "FadeOverTime",
                            "when": "second 0",
                            "attributes": {
                                "start_opacity": "0",
                                "end_opacity": "1",
                                "duration": "second 1"
                            }
                        }
                    ]
                },
                {
                    "name": "leave",
                    "actions": [
                        {
                            "library": "jao.standards",
                            "name": "FadeOverTime",
                            "when": "second 0",
                            "attributes": {
                                "start_opacity": "1",
                                "end_opacity": "0",
                                "duration": "second 3"
                            }
                        }
                    ]
                }
            ]
        }
    ]
}
```

<a name="faq"/></a>
# FAQ
### Why not YAML?
Because then it would be called *yao*, which does not sound as good as *jao*.

### Has this been implemented and used *for real*?
Yes, see the use case below.

### Will that solve problem XYZ?
While it is possible that the jao format can be used in a variety of situations (game developement, web developent, other kinds of ui-driven applications) to know if it will *really* solve your problem is up to the following points:
- Is there an implementation that can be used with the technology stack that you are using?
- If not, are you willing to implement?
At this point there is only the Java implementation so if your use case is specific enough to use Java, it may be helpful.

### Is there a visual editor?
Not as of now, but it would be nice to have. For rushbeat a preview tool has been created, it helps building the screen layers, organizing the positioning of the jao files and testing the events. It is not production-ready and is very fiddly. Still, having a graphical editor would be great.

<a name="usecase"/></a>
# Use Case: Rushbeat
The main reason why that was developed for Rushbeat was the development of menus and UI components:
- The menu has a lot of animation and eye candy which required tweaking. Being developed by a single person, time is a crucial resource -- the use of JAO and the JAO Visualizer has saved a lot of time with UI development;
- The entire game UI uses a skin system. Using JAO allow for the skin (appearance) to be decoupled from the code (logic). Users of the game can create their own skins without having to write code.

Here is an example of what the UI of Rushbeat looks like:

<a href="http://www.youtube.com/watch?feature=player_embedded&v=iVOMaDSx6Us" target="_blank"><img src="http://img.youtube.com/vi/iVOMaDSx6Us/0.jpg" alt="Rushbeat Gameplay" width="400" height="240" border="10" /></a>
