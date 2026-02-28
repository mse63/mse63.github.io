---
title: 3D Printed RC Hovercraft
layout: default
filename: hovercraft
order: 3
--- 

# 3D Printed RC Hovercraft

## Demo Video
Here is a video demo of the completed project:
<iframe width="640" height="385" src="https://www.youtube.com/embed/mySd611Zj0Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Design
I began by making my own impellers. I wanted to create them out of PLA,as it is the easiest and most common 3D printing material. Unfortunately, it has a relatively low glass transition temperature of about 60°C, This led to an issue where the motor would heat up and melt the PLA impeller, which clearly wasn't good for either of them.
After numerous iterations, the problem was solved with an impeller which which went _around_ the motor, sucking air away from it as it spins, cooling the motor down enough that the impeller would never begin to melt:

<img src="impeller.png">

I then designed and 3D printed an inner and outer body for the hovercraft which redirected the air downwards, and letting the hovercraft hover, and it was finally ready to test!

The first issue I ran into was due to awkward propeller placement. My initial design has the propeller position on top of the hovercraft:

<img src="hovercraft_old.png">

This led the propeller to apply a large torque to the body, forcing it to nose down and dig into the ground. This problem was fixed in the final design (shown in the video above), by relocating the propeller to the back of the hovercraft, so that its thrust was more in line with the hovercraft's center of mass.

After a bit of trial and error in finding a skirt material that is durable enough to handle sliding against asphalt, while not having lots of friction, I eventually used a vinyl shower curtain, fully completing my hovercraft.

## Future Development
This project is completed, so there is no future development.
If I _were_ to revisit this project, I'd be interested in creating a hull that can be printed in one piece, rather than separate pieces that screw together. I'd also try more experimentation with hull shape. Particularly, I'd be interested in seeing whether any hull shape could result in behaviour which is stable enough to make a skirt unnecessary on level ground or still water.
