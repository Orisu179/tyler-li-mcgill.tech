---
title: "Google Summer of Code August Update" 
description: "This is an update for GSoC"
pubDate: "July 31, 2024"
heroImage: "/itemPreview.png"
---

### Midi is finished
Midi support should be ready for testing, most of the features have been implemented including a Midi Keyboard.
Check out the [midi](https://github.com/Orisu179/AmatiPP/pull/16) branch to see it!

Midi will be automatically enabled when the user write **declare options "[MIDI:on]";**

Doing this took awhile due to me having trouble with declaring two global variables, apparently I needed to specify them outside of header files or classes, 
which was a bit counterintuitive to me. But other than that implementing the features is pretty straightforward 

### MetaData
Currently, in Faust, you can specify UI metadata in code which will contain changes to the UI. 
When I was writing the proposal, I did not know this feature wasn't already implemented in Amati.
If I had known I would've put this in the proposal, and IMO, it would provide a better user experience than some of the stuff I wrote in my original proposal.
I think having these functionality would be more important than stuff like keyboard shortcuts or opening faust files in an external editor

For my approach, I tried to use a builder design pattern, but it only somewhat follows it. What I would do is iterate over all the metadata and if there are value key pairs,
the related function will be called which build on top of the attachment/slider. The slider/VTS attachment will both be returned.

Currently, only a few of the metadata are implemented. I had run into problem with the scale in particular due to some weird behaviours that I am still trying to figure out.
I am trying to use [ValueConverter](https://github.com/grame-cncm/faust/blob/master-dev/architecture/faust/gui/ValueConverter.h) to convert the values, but the slider doesn't seem to update.
So what I set a skewfactor to the slider. It worked, but now the value changed is not right, even though it is on my debugger, so it's something I am still trying to figure out. 

### Plans for the rest of the months

- Get slider working
- Polyphonic support
- Fix the bug that's related to VST/CLAP formats
- SVG generation
- Enable Scrolling in the parameters section
- Implement Most of the metadata stuff

I think the above would cover most of my primary goals, the rest are stretch goals which are less important IMO.
If possible, I would like an extension of two weeks (?) to polish up the project, but otherwise I could still finish in time, but just might not be all of the above
If the above isn't finished on time, I will still continue to work on this project in my spare time and hopefully finish the rest. 