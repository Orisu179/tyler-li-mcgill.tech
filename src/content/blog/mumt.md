---
title: "MUMT 306 Final project"
description: "This is the final project report for MUMT 306 class"
pubDate: "Dec 26, 2024"
heroImage: "/project.png"
---

This blog post describes what I did for the final project of the course MUMT306 in Fall 2024
at McGill University.

I have built a simple wavetable synthesizer with MIDI support and ADSR for the volume with the [JUCE framework](https://juce.com/).
It currently looks like the following:
![project](/project.png)

A wave table is an array where it stores a fragment of a waveform, which is a plot of signal over time. Think of a sine wave plotted
on an Amplitude-Time graph (with y-axis being the amplitude and x-axis being the index).

For the synth plugin itself, there's a gain slider, a selection of wavetable types, and an ADSR parameters.
They are pretty self-explanatory, but currently only the sine wave is implemented. Also, since it's a VST plugin, it can be used in a DAW 
and supports any MIDI input.

When programming the synth, I have run into a few small problems. One of which is a notable click 
when a note stops playing. This is due to note off setting the gain to 0 with a sharp drop-off. 
This is why I thought of implementing ADSR as a way to fix this issue, along with allowing the user to create more interesting sounds
than a simple sine wave. 

For the wavetable, the samples is stored in a std vector within an oscillator class, and the synth will have a vector of oscillators.
Each time a note is pressed, an oscillator will be created. This leads to a problem where with too many notes, the volume could get too loud.
So the oscillator gain will be scaled down depending on the size of the oscillator vector.

That's about it for the project, if you want to try it out for yourself, the source code could be found here: https://github.com/Orisu179/wavetable_synth 
