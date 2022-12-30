### Long lists of items

A common repeating performance problem in React JS is a complex and long list of items.

Say the app needs to render many cards (in some cases, even hundreds) which are composed of images (or even better – carousels of images), titles, links, ratings and they need to be responsive. 

The slowdown on low-spec laptops and mobile devices may be noticed very quickly. 

This happens because React needs to observe each change in every element of the list. This process consumes lots of resources. 

Luckily, it isn’t that difficult to fix – **virtualized lists** come to the rescue. 

**Key take-away:** long list of items can result in poor performance on low-spec devices. To prevent the issue use virtualization.

A virtualized list is a list that renders only the content visible on the screen and blanks out all other elements until the user decides to scroll down or up in order to see them, thus, saving lots of valuable resources. 

The most popular library for the list virtualization is [react-window](https://github.com/bvaughn/react-window).

![Image](https://global-uploads.webflow.com/622fa4d65a5fab0c3465af07/627938c3c5ead274ae6626b9_react-js-problems-lists.jpeg)

As seen in the graphic above, only the items that can be currently viewed on the screen are rendered. The rest is blanked out.