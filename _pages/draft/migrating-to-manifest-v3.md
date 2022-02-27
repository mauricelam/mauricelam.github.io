# Migrating ScrollMaps to Manifest V3

On November 2020, [Chrome announced that manifest V3](https://developer.chrome.com/docs/extensions/mv3/intro/platform-vision/) will be the future of Chrome extensions, with a timeline to only accept manifest v3 extensions into the Chrome webstore by early 2022, and disabling all older extensions still using manifest v2 in 2023.
  
Obviously, this means ScrollMaps have to migrate to manifest v3. There are many changes introduced with manifest v3, some of them more user-focused and others more developer-focused, but for this post, let's focus on a few of them:
  
* Host permissions are "runtime" only
* Promise-based APIs
* Migration of background pages to service workers
