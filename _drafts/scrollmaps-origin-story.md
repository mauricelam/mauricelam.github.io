ScrollMaps (TODO: add link) is a simple browser extension for Chrome and Firefox that allows users to pan the map using the two-finger-scrolling gesture.

It started out back in 2012, and at the time the dominant form of mouse wheels were those clicky, one-dimensional mouse wheel that have distinct clicks. But at the same time, a new form of scrolling is starting to emerge, which is the two-finger-scrolling gesture, with smooth and inertial scrolling. The introduction of two-finger-scrolling was made possible by the multi-touch technology pioneered by the iPhone and transplanted onto laptops in the form of a multi-touch trackpad. Similarly, smooth and inertial scrolling was popularized by the iPhone's fling and scroll interactions.

For Google Maps, since it was heavily optimized for the traditional form of mouse scroll wheels at the time, the resulting experience did not handle the smooth scrolling from multi-touch trackpads very well. At the time, it zoomed one zoom level per scroll-event, considering only whether the delta is positive or negative, but not the magnitude of it. With the smooth scrolling feature, these `wheel` events are fired tens or even hundreds of times every second, each with a small delta of a few pixels. This means that Google Maps interpreted as scrolling tens of times every second, which obviously was not a good experience.

![macos-multi-touch-gestures-social-card](https://user-images.githubusercontent.com/1264702/152716830-5934b606-d9c8-4654-a8b5-e6f4ea642c68.jpeg)

This is when I got frustrated with the interaction – I was impressed with the smooth inertial scrolling in almost every other situation. Having some background previously with javascript and Chrome extensions, I was thinking to myself maybe I can take this matter into my own hands, and start building a browser extension for it.

## Wheel event capturing

## Scrollability detection

## API injection
