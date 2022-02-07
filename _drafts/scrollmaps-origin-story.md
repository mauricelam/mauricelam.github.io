[ScrollMaps](https://github.com/mauricelam/ScrollMaps) is a simple browser extension for Chrome and Firefox that allows users to pan the map using the two-finger-scrolling gesture.

It started out back in 2012, and at the time the dominant form of mouse wheels were those clicky, one-dimensional mouse wheel that have distinct clicks. But at the same time, a new form of scrolling is starting to emerge, which is the two-finger-scrolling gesture, with smooth and inertial scrolling. The introduction of two-finger-scrolling was made possible by the multi-touch technology pioneered by the iPhone and transplanted onto laptops in the form of a multi-touch trackpad. Similarly, smooth and inertial scrolling was popularized by the iPhone's fling and scroll interactions.

![macos-multi-touch-gestures-social-card](https://user-images.githubusercontent.com/1264702/152716830-5934b606-d9c8-4654-a8b5-e6f4ea642c68.jpeg)

For Google Maps, since it was heavily optimized for the traditional form of mouse scroll wheels at the time, the resulting experience did not handle the smooth scrolling from multi-touch trackpads very well. At the time, it zoomed one zoom level per scroll-event, considering only whether the delta is positive or negative, but not the magnitude of it. With the smooth scrolling feature, these `wheel` events are fired tens or even hundreds of times every second, each with a small delta of a few pixels. This means that Google Maps interpreted as scrolling tens of times every second, which obviously was not a good experience.

This is when I got frustrated with the interaction – I was impressed with the smooth inertial scrolling in almost every other situation. Having some background previously with javascript and Chrome extensions, I was thinking to myself maybe I can take this matter into my own hands, and start building a browser extension for it. Originally the plan was for this to be a weekend project, so I allocated one weekend to explore how to capture the right `wheel` events on the right HTML elements, and translating them into the right "drag" gestures.

## Wheel event capturing and simulation

## "The back of the drawer"

At the time, I was also reading Walter Isaacson's biography on Steve Jobs, and the idea of focus and perfectionism had an effect on me.

> "When you're a carpenter making a beautiful chest of drawers, you're not going to use a piece of plywood on the back, even though it faces the wall and nobody will ever see it. You'll know it's there, so you're going to use a beautiful piece of wood on the back. For you to sleep well at night, the aesthetic, the quality, has to be carried all the way through."

So to practice this notion of perfectionism, I spent the next few months perfecting the smallest details of the extension to the extent possible, introducing code and feature that most users would not notice or care about, but that improves the experience silently in the background. ScrollMaps has a number of these features, probably the most amongst all my projects, and here are a few examples:

#### Scrollability detection

Scrollability is one of those features that was added for perfection. ScrollMaps was built with this functionality where you can request for scrolling to be disabled until you click on the maps portion, which avoids the problem where you are scrolling down the page, and halfway down there is an embedded Google Maps, and it suddenly starts zooming / panning instead of letting you scroll further down the page. This in itself was a nice feature (which years later Google solved by adding a [developer flag](https://developers.google.com/maps/documentation/javascript/interaction) and let the developer choose the desired behavior), but to perfect the last bit, I decided to implement "scrollability detection", which is a feature that will detect whether the current page where the Google Map is shown on is scrollable or not. If the page itself is not scrollable, we can always allow the map to capture the `wheel` event and use that for panning.

That detection may sound like a simple piece of code, but when you take into account edge cases like `iframes` and different scrollbar settings, there is actually a surprising amount of subtlety involved (which I definitely didn't expect going into it). This detection in itself can be the topic of another post of its own.

#### API injection


## Links

* GitHub repository: https://github.com/mauricelam/ScrollMaps
* Chrome webstore: https://chrome.google.com/webstore/detail/scrollmaps/jifommjndpnefcfplgnbhabocomgdjjg
* Firefox addon: https://addons.mozilla.org/en-US/firefox/addon/scrollmaps/
* Blog post back in 2014 describing the problem: https://www.popularmechanics.com/technology/apps/reviews/a10753/scrollmaps-is-the-google-maps-fix-you-need-16931788/
