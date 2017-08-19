
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

The most significant difference between the CSS Line Grid and CSS Rhythmic Sizing proposals is that the latter proposal doesn't actually establish a grid, but instead tries to simulate it by ensuring that the he height of all relevant objects matches the step size.  This places the burden on developers who want to maintain the rhythm to avoid features that aren't integrated with step sizing, and also a burden on developers of future CSS features to integrate all of them with step sizing so that they won't place such a burden on developers.  This makes me inclined to think that the CSS Rhythmic Sizing will work in simple cases but fail in more complex (and realistic) ones.

On the other hand, the CSS Line Grid proposal defines certain elements as establishing a grid, and certain points as aligning to that grid.  This means that if some CSS features don't integrate with the line grid, a small amount of content may be unaligned, but the next use of a feature (blocks, lines) that integrates with the line grid will bring the content back into alignment with the grid.

One of the costs of the line grid proposal is that it requires disabling performance optimizations for moving content vertically during relayout without redoing layout on the internals of that content.  (These optimizations are also disabled in some other cases, such as the presence of floats next to the content.)  However, the optimization only needs to be disabled when moving content by a distance that is not a multiple of the grid height.  When most content fits itself to the grid, such changes will mostly cover small areas of content, though this depends somewhat on what mechanism is used to enable or disable the grid, and in particular whether the mechanism for doing so allows reenabling a grid established by a further ancestor that was disabled on a nearer ancestor (see next section).

### How to enable the grid?

Given that the primary use case is simply to switch into a mode where lines maintain vertical rhythm, it would be good to support enabling that use case by setting a single CSS property.

Fundamentally there are three (or four) operations:

1. establishing a line grid based on the font and `line-height` of an element
2. choosing to snap (a) lines and (b) blocks (in that element and its descendants) to that grid (this could count as two if you separate lines and blocks), and
3. tuning the alignment of blocks that are snapped to the grid.

The operation of establishing the grid must be done by a non-inherited property, whereas the others could be done by either an inherited property or non-inherited property.

The CSS Line Grid proposal puts (1) in a non-inherited `line-grid` property, (2a) in an inherited `line-snap` property, and (2b) and (3) in an inherited `box-snap` property.  CSS Rhythm doesn't technically have (1), but also maintains the same separation between the other properties, while splitting (3) into a set of subproperties for finer control.

I suggest an alternative approach:  putting (1) and (2) in a single non-inherited property, and using properties from CSS Box Alignment for (3).  The initial `auto` value of the property would use the same grid as the parent but not establish a new one, an `establish` value would establish a line grid on the element and use it on that element and any `auto` children (and *their* `auto` children, etc.), and a `none` value would stop using a line grid established by an ancestor.  This alternative approach has the advantage that line snapping could be enabled by a single property.  The tradeoff is that it has the disadvantage that it would not be possible (in this initial version) to enable line-snapping on an element A, disable line snapping on element B (a descendant of A), and then re-enable line-snapping on an element C (a descendant of B) *to the same grid* as the snapping done on A.  I'm not convinced that this use case is important enough to require that every developer who wants to use a line grid specify two CSS properties rather than one.  (If there are examples showing that this case is important, I'd like to see them.)

Making this design choice would still allow later switching to the approach that separates establishment from use by splitting the property to establish and use the grid into a shorthand, where the longhand covering establishing would distinguish between `establish` and the other values, and the longhand covering use would distinguish between `none` and the other values.  This could potentially also be extended to allow named grids, which would be even more powerful, if there were use cases to require that power.  It would, however, constrain the property on use of the grid to be non-inherited, which would be a slight complexity cost for implementations, since they would need to track the nearest ancestor with a non-`auto` value.

It's worth noting, however, that this design choice also has a limitation that may be valuable.  In particular, it limits the areas of the document where a grid needs to be propagated but snapping is not done to only the parts of the CSS formatting model that don't support snapping to the line grid.  This is likely to keep such regions small, which means that the scope of the disabling of the vertical movement performance optimization mentioned in the previous section is also likely to be small.

Finally, I propose that an additional operation be integrated into the enabling of the grid:

4. When the grid is enabled, change the rules for line height calculation to consider `line-height` only on blocks.

This would incorporate one of the two major motivations for `line-box-contain` (the other being to describe quirks mode behavior) without the addition of a new property.  This change would apply the `line-height` property only to blocks, and avoid adding extra space for inline boxes unless their bounds (rather than their bounds plus the half-leading inflating those bounds to the size of the `line-height`) overflows the line box.  This, on its own, tends to produce more reliable line heights, and when combined with line snapping, would drastically reduce the number of lines that need to be double-spaced in order to maintain vertical rhythm.

I think this change could, again, be combined with the single property that does (1) and (2).

### How to align intrusions within the grid?

As stated above, there is a requirement for either centering or start-aligning intrusions within the grid that do not have lines snapped to the grid (for example, blockquotes in a smaller font) (see the last of the Use Cases above).  It seems like (although I'm not sure) there may also be things that are not lines in the CSS model, but that in terms of the design fill the role of lines, and thus require baseline alignment.

Controlling this alignment is control (3) in the list of operations above about how to enable the grid.

The [CSS Line Grid Module](https://drafts.csswg.org/css-line-grid/) has just a single `box-snap` property, taking values `none`, `block-start`, `block-end`, `center`, `baseline` and `last-baseline`.  It defaults to `none` because the property also controls (2b).

The [CSS Rhythmic Sizing](https://drafts.csswg.org/css-rhythm/) draft has a more complicated model.  It controls operation (2b) with a property that sets the size (since the module has no control for (1)), `block-step-size`.  This property changes the size of the block, rather than its alignment within a space.  There are then additional properties: `block-step-insert` to say where the space adjustment goes (margin or padding), `block-step-align` to say whether the space adjustment goes on the block-start side, the block-end side, or (for centering) both (which, confusingly and perhaps mistakenly, means the block is aligned to the opposite side) and an `auto` value to honor `align-self`, and `block-step-round` to say whether the adjustment is done by adding space, removing space, or whichever would be a smaller adjustment.

I largely prefer the approach taken in the Line Grid module, because:
* I believe it fits better with the concept of a grid
* it allows baseline alignment, which may be important (although I'm not sure)
* it doesn't have the option for reducing space, which risks causing overlap

However, the presence of `block-step-align: auto` in the Rhythmic Sizing draft does point out that the `align-self` property could be used here.  There is no existing behavior to interact with since it [currently does not apply to blocks](https://drafts.csswg.org/css-align-3/#align-block).  Using `align-self` appears to provide all of the functionality provided by `block-step-align` or `box-snap`, without introducing a new property.

I'd suggest that the functionality provided by [`block-step-round`](https://drafts.csswg.org/css-rhythm/#block-step-round) and [`block-step-insert`](https://drafts.csswg.org/css-rhythm/#block-step-insert) may not be needed.  If there are use cases that require this functionality, I'd like to hear about them.

## Putting things together

### Properties

TODO: Write

### Examples

TODO: Write
