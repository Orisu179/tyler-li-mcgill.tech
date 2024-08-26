---
title: "Google Summer of Code final report"
description: "Final report for GSoC 2024"
pubDate: "August 26, 2024"
heroImage: "/Amati/amati++.png"
---

# Background
I participated in Google Summer of Code this summer with GRAME on a VST/CLAP plugin that allows the user to edit and compile Faust code in real-time. This essentially means that the user can program their own audio plugin within a digital audio workstation or any plugin host.

I was particularly excited to work on this project due to my interest in music technology and the Faust language in general. It is a functional audio programming language that compiles into various platforms, other non-domain specific languages (like C++, C, LLVM bitcode), and architectures. For more information, check out the [Faust website](https://faustdoc.grame.fr/).

For Amati++, I used a pre-existing project named Amati as a starting point and built on top of it, hence the name. Amati is a VST plugin that embeds the dynamic Faust compiler, which then allows it to be run in some plugin host like a digital audio workstation. The creator of Amati, Grégoire Locqueville, was kind enough to not only permit me to use the project as a starting point but also joined as one of the mentors for this summer. Of course, that also brings me to thank my other three mentors Thomas Rushton, Stéphane Letz, and Kamil Kisiel. They have all provided me with valuable insights and great help during this program. I cannot appreciate them enough for taking the time to discuss the project and helping me with roadblocks that I have run into.

I used the [JUCE framework](https://juce.com/) for this project. It is an audio plugin framework that’s open source and supports numerous formats including VST3 and CLAP (with an additional plugin). This allows one to have one code base and compile it to multiple different formats instead of using the VST/CLAP SDK. Furthermore, JUCE has a fairly robust, although a bit hard to work with GUI library. Most importantly though, it’s what the original Amati project used. This allowed me to use some of its existing code as a starting point for the project.

# Project Goals
During my proposal, I grouped my goals into three categories, primary, medium priority, and stretch goals.
### Primary goals:
1. Support CLAP format as an additional build target on top of VST3. CLAP is a more modern format developed by u-he and Bitwig which has better multithreading performance along with more customizable parameter modulation. For more information on the CLAP format, check out their [website](https://u-he.com/community/clap/)
2. Redesign of interface and UI refactor into FlexBox model.
3. Support for MIDI input. This means a MIDI architecture has to be implemented in JUCE (juce-midi.h will be used as a starting point), and CLAP format-specific functions
4. Support for polyphonic instruments
### Medium priority goals:
1. A MIDI keyboard UI will be displayed if the plugin is on a MIDI channel
2. Generate a live diagram of the Faust file with and display it with SVG
3. Testing with the catch2 framework
### Stretch goals:
1. Option to use an external editor (i.e. VSCode, Neovim, JetBrains IDEs, etc), when the Faust file is saved in the external editor, changes will be reflected within the application
2. Settings will persist throughout each session via some sort of cache
3. Keyboard shortcuts for compiling and other actions (e.g. switching between tabs)
# What I have achieved
While working on the project I realized that there are other more impactful improvements than features like keyboard shortcuts. So priorities of some of the goals have changed. Namely, the stretch goals one and three were not worked on at all. For 2, I realized that the original Amati project already caches them in the form of XML. Other than those, all primary goals have been achieved, and the first of the three medium-priority goals was also achieved, with SVG and testing currently in progress.

CLAP support was pretty simple due to the existence of the [JUCE CLAP extension](https://github.com/free-audio/clap-juce-extensions).
For the UI refactor, I have changed the color theme of the plugin, and a few minor changes to the UI from the original to make it look a bit better than the default JUCE look. I planned on making more changes, but it seems like Flexbox in JUCE is still not well-supported and a bit buggy.

When I first wrote my proposal, I looked at other popular code editors (like VScode or JetBrains IDEs) and used features in those as references. However, after spending more time with Amati, I decided that some quality-of-life features should be given priority over those. Firstly, in Faust, there are metadata that the user can specify to parameters. I have implemented some of them like ```hidden``` or ```scale``` which affects the ```hslider```/```vslider``` primitives. For more information, check out the [official documentation](https://faustdoc.grame.fr/manual/syntax/#scalexx-metadata) Another feature is that since the sliders/buttons are fixed-sized, there can be situations where some parameters can’t be displayed due to them being out of screen size. Thus I have implemented a scrollable panel using Juce’s viewable component.

Next was MIDI support, while the implementation for it was straightforward, there were a lot of problems with parameters not updating. For example, when a MIDI message changes a parameter, it won’t be updated in the interface. To fix this, I had to check if the values are the same in the [AudioProcessorValueTreeState](https://docs.juce.com/master/classAudioProcessorValueTreeState.html) which I used for storing parameter state, and the parameter state within the Faust instance. While I came up with a more efficient way of checking at the time, MIDI isn’t the only way to modify parameters. So while inefficient, it’s what my mentor, Stéphane Letz, and I have decided on.

For polyphonic support, I used regular expressions to get the metadata within ```declare options```. So if the user has ```declare options "[midi:on][nvoices:n]";``` in their code (where n is the maximum number of voices that they want to use), it means that they want to use a polyphonic instrument. For more information, check out the (official documentation)[https://faustdoc.grame.fr/manual/midi/#midi-polyphony-support]. However, I was getting an “undefined effect” error when compiling, this is due to the Faust compiler itself complaining. For polyphonic instruments in Faust, the user can define an ```effect``` value which will then automatically be connected to the process. However, if the user doesn’t define it, an error will appear. Thus, I had to append a line of code to the user-defined Faust program source code to make it go away. It defines the effect to nothing: 
```
effect = _;
``` 
if the user doesn’t define it themselves. With Libfaust, there are two factory classes used to generate a Faust instance, a regular DSP factory and a polyphonic DSP factory. 
At first, I wanted to have one factory and cast between them, but due to the polyphonic factory being a subclass of the regular factory, I could not cast upwards. So in the situation where the user compiles polyphonic code and then tries to compile without it, the program will crash. So as a solution, I included both regular and poly factories and selected the appropriate factory two depending on whether the user writes Faust code that requires polyphonic support.
# The current state
## Here’s what the current plugin looks like:
### Editor
![Editor](/public/amati++.png)
### Parameter 
![Parameter](/public/param.png)
### Console 
![Console](/public/console.png)
### MIDI keyboard
![MIDI keyboard](/public/midi.png)
### Settings
![Settings](/public/settings.png)

In the original, the faust script is compiled right as it loads, and I didn’t like that. I feel like the user should have full control over when the program should compile. Thus I changed it so the user has to press the compile button for the program to compile. I have also added a start and stop button so the user can choose when they want to run the Faust program, as opposed to starting right away.

There’s also a new MIDI keyboard section that the user can use, just in case the DAW/or whatever the user is using doesn’t have a virtual keyboard.

For polyphonic and MIDI, the plugin should know automatically if they are required through the use of metadata like this:
```
declare options "[midi:on]";
```
or 
```
declare options "[midi:on][nvoices:12]";
``` 
for polyphonic. Regular expressions were used to match these patterns.

# What's left to do
I want to finish SVG generation, along with writing some test cases. There are also some metadata features that I could implement. I will continue working on this project even after the GSoC period.

Some other features that I think would be nice to have would be a selectable color theme which the users can change. The bases for this option are already there, but I need to think about how the structure of this can fit in with the rest of the source code.
# Things that I have learned
I learned a lot of C++, CMake and the JUCE framework. I tried to implement a couple design patterns like builder or singleton (which didn’t go well), and how MIDI works.

I have also learned about what it takes to update a plugin parameter while the audio thread is running, and JUCE’s way of handling it through the Audio Processor Value Tree State.
# Conclusion
Overall, I enjoyed this project immensely, it gave me an opportunity to work on my own schedule and contribute to an existing ecosystem.
Check out the source code (and releases) on Github at the following link: https://github.com/Orisu179/AmatiPP

Thanks to Google for sponsoring this program, and GRAME for accepting my proposal and creating Faust. To finish this off, I want to give a thanks to the four of my mentors:
- Kamil Kisiel, for willing to meet up with me in person last year despite me being a total stranger, and agreeing to mentor me for this project.
- Grégoire Locqueville for his amazing pre-existing work on the original Amati project.
- And of course, huge thanks to two of my mentors, Thomas and Stephane for their bi-weekly support throughout the summer

