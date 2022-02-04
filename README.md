# jao-specification
The specification for the jao file format.

# What is JAO?
JAO stands for Json Animated Objects. It is a specification for defining event-driven, animated UI components using a JSON file. This specification was created initially for use in game development, but can probably be used for other purposes.

# Why?
Defining the visual behaviour of UI components outside of your code can help with a few things:
- Decouple the logical behavior (code & logic) from the visual behavior (graphics & animation) of the UI component;
- Allow for the development and test of the visual behavior of UI components before integrating them into your code, leading to a quick develop-test-fix cycle;
- Preview & tweaking of the UI of a game without having to run the game by using a visual editor/visualizer.

# Use Case: Rushbeat
The main reason why that was developed for Rushbeat was the development of menus and UI components:
- The menu has a lot of animation and eye candy which required tweaking. Being developed by a single person, time is a crucial resource -- the use of JAO and the JAO Visualizer has saved a lot of time with UI development;
- The entire game UI uses a skin system. Using JAO allow for the skin (appearance) to be decoupled from the code (logic). Users of the game can create their own skins without having to write code.

Here is an example of what the UI of Rushbeat looks like:
https://www.youtube.com/watch?v=iVOMaDSx6Us

Menus & Gameplay graphics have extensively used the JAO library. Every graphical component (except by the widgets in the settings screen) have been drawn and animated using JAO files (even music/sound is played back and defined inside the JAO files).

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
                  "library": "...",
                  "name": "...",
                  ...
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
               "library": "...",
               "name": "...",
               ...
            ]
         },
         ...
      ]
   },
   ...
]
```

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
            "library": "...",
            "name": "...",
            ...
         ]
      },
      ...
   ]
}
```
A layer is composed by a data type specification and a list of events. Layers are rendered from the first (bottom) to the last (top).

### Data Type Specification
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

### Events
Each layer carries a list of events, it can have zero or more events in this list.
```
"events": [
   {
      "name": "initialize",
      "actions": [
         "library": "...",
         "name": "...",
         ...
      ]
   },
   {
      "name": "select",
      "actions": [
         "library": "...",
         "name": "...",
         "when"
      ]
   },
   ...
]
```

### Event
An event defines the behaviour of this layer when a given event starts. Imagine a menu option:
- It will have an event called `select`, that may trigger the animations to show the UI component selected in the screen, like a highlight;
- It will have an event called `unselect`, that may trigger the animations to remove the highlight from the UI component.

The event will describe *what* happens when this event is triggered, which is a list of actions. An event can have zero or more actions.

As each layer has a list of events, if you want to have multiple layers reacting to the activation of the same event, each layer must have an event with the same name (see examples below).

#### The 'initialize' event
Events can have any name, except by `initialize`. This is a reserved event name for the initialization of the layer. You may want to define attributes of the layer when it is initialized -- ex.: set the opacity of a layer to 0 in a layer that will fade into the screen. For those cases you will want to use actions inside the `initialize` event to setup the initial state of the layer.
