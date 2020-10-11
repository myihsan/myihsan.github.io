---
layout: post
title: "Programmer Oriented UI Design: Content Area"
---

A common missing part in a non-deliverable UI design is the description of the content area for components. Let's see why we need them before delivering the design to programmers.

Here is a common design for the cell in a list.

| ![Common cell design](/assets/images/common-cell-design.png) |
|:-:|
| Common cell design |

How can this go wrong? There is nothing wrong with this design, but it's not enough.

Let's make the content area of those two labels visible.

| ![Visible content area](/assets/images/visible-content-area.png) |
|:-:|
| Visible content area |

You might already notice that we cannot tell the maximum width of the labels from this design.

Let's define them.

| ![Explicit content area](/assets/images/explicit-content-area.png) |
|:-:|
| Explicit content area |

It's better. But we still have no idea that what will happen when the content area is not enough for the text.

A common approach is to add an ellipsis at the end or after the last word in the content area.

| ![Truncating text](/assets/images/truncating-text.png) |
|:-:|
| Truncating text |

However, this is usually not acceptable. Here is another approach.

| ![Autosizing text](/assets/images/autosizing-text.png) |
|:-:|
| Autosizing text |

We can change the font size of the label to fit the content size.
This is effortless to implement in iOS and Android, but there is an issue in this approach–when the text is long enough, the font could be too small to be readable.

So we may consider making the content area extendable in the vertical direction.

| ![Extendable content area](/assets/images/extendable-content-area.png) |
|:-:|
| Extendable content area |

With this design, the vertical alignment of the right label also became specific.

We still have to choose the approach for the right label. After that, this design finally becomes deliverable.

We indeed have to spend more time on this little component, but it's always better to determine by ourselves than spend our time and the programer's time on unnecessary communication.
