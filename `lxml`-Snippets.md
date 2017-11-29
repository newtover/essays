There happen situations when you build an XML based on a stream of events. This quickly leads you to a problem of appending multiple text nodes as children to the same element. This is not a problem for the DOM interface itself, but it is for the interface of ElementTree that `lxml` is implementing.

A solution is to build a etree from SAX events:
* https://kurtraschke.com/2010/09/lxml-inserting-elements-in-text/
* http://lxml.de/sax.html#building-a-tree-from-sax-events