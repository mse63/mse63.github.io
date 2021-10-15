---
title: Servo Controller
layout: template
filename: servo
order: 1
show_tab: 1
--- 

# Servo Controller

## Demo Video
Here is a video demo of the project as it currently stands:
<iframe width="640" height="385" src="https://www.youtube.com/embed/mt5a9yana8U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Design
I will talk about super cool design stuff here.

## Future Development

I plan on adding a more advanced control mechanism. Currently, it simply looks at which direction to move the motor in, and slows down its the signal with a low-pass filter to prevent an extreme overshoot. This results in oscillatory behaviour because it is similar to bang-bang tuning. I would like to implement PID tuning (or some subset of it, such as PI or PD). Although implementing an exact integral or derivative function would be near impossible to do with analog circuitry, it can be well approximated with a low-pass or high-pass filter.
