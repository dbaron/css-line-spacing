
# Better spacing of lines in CSS, an explainer

## Abstract

This is a proposal for features in CSS whose goal is to improve:

* evenness of line spacing
* matching of line spacing between adjacent areas (pages, columns, grid cells, other boxes)

to support design requirements that have been important in traditional typography but are difficult on the Web.

Such features have been discussed in the CSS working group a number of times.  This document explains the use cases, and discusses ways to move forward.

## Historical background

The rules for line spacing in CSS had to balance two competing requirements.

First, since design on the Web involves designing for a set of devices with different characteristics (sizes, available fonts, user preferences), the design needs to be resilient to changes in these characteristics.  In particular, changes between systems, device characteristics, or user preferences, should not cause content to overlap.

Second, designers generally want line spacing to be as consistent as possible, for various reasons (see use cases below).

The behavior of the CSS `line-height` property strikes a poor balance here, since it associates *additional* space (to become the size of the `line-height`) with each item on the line (thus adding more space than needed just to avoid overlap), rather than adding space to avoid overlap only when the visible contents of the line exceed a reasonable space allocated to the line and its spacing.

This document attempts to unify ideas from *four* proposals:
* The [CSS Line Grid Module](https://drafts.csswg.org/css-line-grid/) draft (2011-2017) (incubation / [exploring](http://fantasai.inkedblade.net/weblog/2011/inside-csswg/process) stage), which was addressing this problem
* The [CSS Rhythmic Sizing](https://drafts.csswg.org/css-rhythm/) draft (2016-2017) (incubation / [exploring](http://fantasai.inkedblade.net/weblog/2011/inside-csswg/process) stage), which was also addressing this problem
* The `line-box-contain` property ([2001 draft](https://www.w3.org/TR/2001/WD-css3-box-20010726/#the-line-box-contain), [2009 editor's draft](http://web.archive.org/web/20090225000117/http://dev.w3.org:80/csswg/css3-linebox/#LineStacking)) (2000-2003, 2008) (incubation / [exploring](http://fantasai.inkedblade.net/weblog/2011/inside-csswg/process) stage), which was related to providing more options for how line spacing could work
* The [CSS Box Alignment Module](https://drafts.csswg.org/css-align/) draft (2012-2017) ([refining / testing](http://fantasai.inkedblade.net/weblog/2011/inside-csswg/process) stage), which can be used for how block-line objects fit within a line grid

## Use cases

**Line spacing should be as even as possible, in order to establish visual rhythm.**
As much as possible, the distance from one baseline to the next should be the same.
[Some have argued](https://log.csswg.org/irc.w3.org/css/2017-08-04/#e848096) that this requirement is even more important for Chinese, Japanese, and Korean, where the edges of the characters all align to the same size, than for bicameral scripts like Latin with ascenders and descenders that make the lines visually irregular.

**It should be possible to make baselines in adjacent boxes align.**
This can be important for:

* opposite sides of the same sheet of paper
* facing pages
* adjacent boxes of various sorts (columns, grid cells, etc.)

However, it should also be possible to avoid alignment of baselines in adjacent boxes, since causing such alignment requires the insertion of extra spacing in some cases (e.g., around anything that interrupts the rhythm).

**It should be possible to choose different alignments for objects that interrupt the rhythm.**
Some objects will interrupt the rhythm of lines, such as block-level images or figures (necessarily), or quotations printed in a different font (optionally).  Designers wish to choose multiple alignments for such objects.  (See, for example, comments from 小林 敏 (KOBAYASHI Toshi, こばやし とし, Kobayashi-sensei) in the [2017 April 20 discussion](https://lists.w3.org/Archives/Public/www-style/2017May/0050.html) under "Rhythm discussion notes", particularly "Ideal is centered [...], but some things are better top-aligned, e.g. side notes.")

## Key technical decisions

### Grid or no grid?

The most significant difference between the CSS Line Grid and CSS Rhythm proposals is that the latter proposal doesn't actually establish a grid, but instead tries to simulate it by ensuring that the he height of all relevant objects matches the step size.  This places the burden on developers who want to maintain the rhythm to avoid features that aren't integrated with step sizing, and also a burden on developers of future CSS features to integrate all of them with step sizing so that they won't place such a burden on developers.  This makes me inclined to think that the CSS Rhythm will work in simple cases but fail in more complex (and realistic) ones.

On the other hand, the CSS Line Grid proposal defines certain elements as establishing a grid, and certain points as aligning to that grid.  This means that if some CSS features don't integrate with the line grid, a small amount of content may be unaligned, but the next use of a feature (blocks, lines) that integrates with the line grid will bring the content back into alignment with the grid.

One of the costs of the line grid proposal is that it requires disabling performance optimizations for moving content vertically during relayout without redoing layout on the internals of that content.  (These optimizations are also disabled in some other cases, such as the presence of floats next to the content.)  However, the optimization only needs to be disabled when moving content by a distance that is not a multiple of the grid height.  When most content fits itself to the grid, such changes will mostly cover small areas of content, though this depends somewhat on what mechanism is used to enable or disable the grid, and in particular whether the mechanism for doing so allows reenabling a grid established by a further ancestor that was disabled on a nearer ancestor (see next section).
### How to enable the grid?

TODO: write about how flipping fewer properties is better

TODO: write about integrating an implicit flip of `line-box-contain`

### How to align intrusions within the grid?

TODO: compare approaches in line-grid and rhythm, vs. using css-align mechanisms directly
