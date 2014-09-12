---
layout: userguide-article
title: "Google services setup"
description: "Responsive web design patterns are quickly evolving, but there
              are a handful of established patterns that work well across the
              desktop and mobile devices"
introduction: "For full-width multi-column layouts, column drop simply stacks the columns
  vertically as the window width becomes too narrow for the content.  "
authors:
  - petelepage
article:
  written_on: 2014-04-30
  updated_on: 2014-04-30
  order: 4
collection: setup
---

{% wrap content%}

Eventually
this results in all of the columns being stacked vertically.  Choosing
breakpoints for this layout pattern is dependent on the content and will change
for each design.


Like the mostly fluid sample, content is stacked vertically in the smallest
view, but as the screen expands beyond 600px, the primary and secondary content
`div`'s take the full width of the screen.  The order of the `div`'s is set using
the order CSS property.  At 800px all three content `div`'s are shown, using the
full screen width.

Sites using this pattern include:

 * [Modernizr](http://modernizr.com/)
 * [Wee Nudge](http://weenudge.com/)


{% include modules/nextarticle.liquid %}

{% endwrap %}
