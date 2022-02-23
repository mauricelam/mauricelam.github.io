[ScrollMaps](https://github.com/mauricelam/ScrollMaps) is a simple browser extension for Chrome and Firefox that allows users to pan the map using the two-finger-scrolling gesture.

It started out back in 2012, and at the time the dominant form of mouse wheels were those clicky, one-dimensional mouse wheel that have distinct clicks. But at the same time, a new form of scrolling is starting to emerge, which is the two-finger-scrolling gesture, with smooth and inertial scrolling. The introduction of two-finger-scrolling was made possible by the multi-touch technology pioneered by the iPhone and transplanted onto laptops in the form of a multi-touch trackpad. Similarly, smooth and inertial scrolling was popularized by the iPhone's fling and scroll interactions.

![macos-multi-touch-gestures-social-card](https://user-images.githubusercontent.com/1264702/152716830-5934b606-d9c8-4654-a8b5-e6f4ea642c68.jpeg)

For Google Maps, since it was heavily optimized for the traditional mouse wheels at the time, the resulting experience did not handle the smooth scrolling from multi-touch trackpads very well. At the time, it zoomed one zoom level per scroll-event, ignoring the magnitude of the scroll. With the smooth scrolling feature, these `wheel` events are fired tens or even hundreds of times every second, each with a small delta of a few pixels. This means that Google Maps responded with by zooming tens of times every second, making it impossible to control the zoom level reasonably.

This is when I got frustrated with the interaction – I was impressed with the smooth inertial scrolling in almost every other situation. Having some background previously with javascript and Chrome extensions, I thought maybe I can take this matter into my own hands, and start building a browser extension for it. Originally the plan was for this to be a weekend project, so I allocated one weekend to explore how to capture the right `wheel` events on the right HTML elements, and translating them into the right "drag" gestures.

## Wheel event capturing and simulation

At its core, the idea of ScrollMaps is very simple. It captures the javascript [`wheel`](https://developer.mozilla.org/en-US/docs/Web/API/Element/wheel_event) events, and [dispatches](https://developer.mozilla.org/en-US/docs/Web/Events/Creating_and_triggering_events) a "drag" event in response. Those of you who are familiar with Javascript may know that ordinary "drag" within the webpage is not one standardized event, but is instead [`mousedown`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousedown_event), [`mousemove`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousemove_event), and [`mouseup`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseup_event) events chained together. (Side note: There is a [`drag`](https://developer.mozilla.org/en-US/docs/Web/API/Document/drag_event) event in Javascript, but that's for dragging and dropping files between the the web content and the native OS outside of the browser).

Because the drag detection is done through detection of this chain of events, it means that each web app will have a slightly different detection logic. For example, some apps may decide to only look at the `movementX` and `movementY` attributes to determine how much the drag is, while others may compare that value to the `mousedown` event to calculate how much it has moved in total. In fact, this is one of the things that broke when a [new redesign of Google Maps](https://www.theverge.com/2013/5/15/4333374/google-maps-redesign-2013-io-event) was launched in 2013. This means ScrollMaps is essentially reverse engineering Google Maps' drag detection mechanism in order to dispatch the required events.

## "The back of the drawer"

At the time, I was also reading Walter Isaacson's biography on Steve Jobs, and the idea of focus and perfectionism had an effect on me.

> "When you're a carpenter making a beautiful chest of drawers, you're not going to use a piece of plywood on the back, even though it faces the wall and nobody will ever see it. You'll know it's there, so you're going to use a beautiful piece of wood on the back. For you to sleep well at night, the aesthetic, the quality, has to be carried all the way through."

So to practice this notion of perfectionism, I spent the next few months perfecting the smallest details of the extension to the extent possible, introducing code and feature that most users would not notice or care about, but that improves the experience silently in the background. ScrollMaps has a number of these features, probably the most amongst all my projects, and here are a few examples:

#### Scrollability detection

Scrollability is one of those features that was added for perfection. ScrollMaps was built with this functionality where you can request for scrolling to be disabled until you click on the maps portion, which avoids the problem where you are scrolling down the page, and halfway down there is an embedded Google Maps, and it suddenly starts zooming / panning instead of letting you scroll further down the page. This in itself was a nice feature (which years later Google solved by adding a [developer flag](https://developers.google.com/maps/documentation/javascript/interaction) and let the developer choose the desired behavior), but to perfect the last bit, I decided to implement "scrollability detection", which is a feature that will detect whether the current page where the Google Map is shown on is scrollable or not. If the page itself is not scrollable, we can always allow the map to capture the `wheel` event and use that for panning.

That detection may sound like a simple piece of code, but when you take into account edge cases like `iframes` and different scrollbar settings, there is actually a surprising amount of subtlety involved (which I definitely didn't expect going into it). This detection in itself can be the topic of another post of its own.

#### API injection

While the implementation described above works well on Google.com/maps, and iframe-embedded Google Maps, those are only one portion of Google Maps usages on the web. Another, arguably more important usage is through the Google Maps APIs. Because Google Maps APIs run by loading the javascript and rendering the content directly in the hosting app's domain, ScrollMaps' content scripts won't be able to reach it by default. For example, on Yelp pages where there is a Google Map embedded, those are done using the Google Maps API, and the resulting content is rendered directly under the yelp.com domain. This means any URL-based targeting or content scripts won't be able to reach those Google Maps. Instead, we needed to inject javascript into every page to trigger our wheel event capturing code.

There are a lot more interesting details to share, such as how to detect usages of the Google Maps API, and making it work on sites that load the Google Maps javascript at different points in time, that I think it will be worth its own post to share all the technical details.


## Conclusion

If there is a moral for the story of ScrollMaps, it is to start pursuing your ideas right away, and relentlessly pursue perfection. ScrollMaps wasn't the grandest idea, nor was it a technical marvel, but how it solves a real problem that users have, combined with the polish to deliver that extra step to help the users combined to make it what it is today.

## Links

* GitHub repository: https://github.com/mauricelam/ScrollMaps
* Chrome webstore: https://chrome.google.com/webstore/detail/scrollmaps/jifommjndpnefcfplgnbhabocomgdjjg
* Firefox addon: https://addons.mozilla.org/en-US/firefox/addon/scrollmaps/
* Edge addon: https://microsoftedge.microsoft.com/addons/detail/scrollmaps/mdhhlgkmnlaiofbbemcmigjleiiefmga
* Blog post back in 2014 describing the problem: https://www.popularmechanics.com/technology/apps/reviews/a10753/scrollmaps-is-the-google-maps-fix-you-need-16931788/
