---
title: "When to scroll? The problem of infinite-sized UI elements inside a scrollviewer"
date: "2012-01-23"
---

While refactoring a WPF application, I've stumbled into a general problem in UI layout.

WPF has an element called ScrollViewer, which is basically a panel that contains elements and shows scroll bars when the content inside is too big to fit in the size of the ScrollViewer's visible area. Consider the following three cases.

### Case 1: Finite-sized UI content within a ScrollViewer

In this case, obviously the scroll bars should only appear if the visible area of the ScrollViewer is too small to show the content. If the window is large enough, there entire finite-sized content will fit in the screen and no scroll bars are necessary (although we may want to still show them, disabled, for stylistic reasons).

### Case 2: Infinite-sized UI content within a limited container (NOT a ScrollViewer)

Sometimes we have content that can use any size assigned to it. For example, a grid (in my case, DevExpress' WPF GridControl) of data rows. Assuming a huge (or even infinite) data source, the grid's size on the screen has no upper bound - it can always grow more and always show more content. The grid's size in the UI must be limited somehow. There are two ways to limit the size of such a UI control:

1. Set a maximum size on the control, or
2. Place the infinitely-sized control in a container that has a maximum size defined.

In other words, the infinite control must lie in a tree of containers where at least one container up the ancestry must have defined a maximum size. If no parent container specifies a maximum size, eventually the top-level container - the window - DOES have a maximum size and the infinite-sized control will fill the whole window (and nothing more).

### Case 3: Infinite-sized UI content within a ScrollViewer

Now, consider that the container of the infinite UI control (such as a grid with infinite rows in the data source), is actually container within a ScrollViewer. In this case, the ScrollViewer, being what it is, does NOT limit the size of its contents, so the infinitely-sized control will "explode" and will, by our algorithm, try to occupy an infinite size. Specifically in the case of a DevExpress WPF GridControl, the authors of that control know to detect that situation - and an exception is thrown, stating:

> DevExpress.Wpf.Grid.InfiniteGridSizeException was unhandled   Message="By default, an infinite grid height is not allowed since all grid rows will be rendered and hence the grid will work very slowly. To fix this issue, you should place the grid into a container that will give a finite height to the grid, or you should manually specify the grid's Height or MaxHeight. Note that you can also avoid this exception by setting the GridControl.AllowInfiniteGridSize static property to True, but in that case the grid will run slowly."

Problem is, sometimes we want to put that grid in a ScrollViewer so that when the screen is too small, the grid will assume some minimal size, and a scroll bar will be shown if the screen (or window) is smaller than the minimum. If the window is huge, what we want is to expand the grid to fill the available space in the window - as big as the window can be, with no limit. If someone is using the application on a 5000-inch screen, we want to use all that space. If someone is using a 1-inch screen, we want the grid to be 3 inches and show a scroll bar.

So, the solution seems simple enough: we can tell the ScrollViewer to have dual behavior:

1. If the available screen size is smaller than some minimum, allow scrolling, and set the size available for the content to be that minimum size.
2. If the available screen size is larger than the minimum, behave like a regular container that gives its children only the space it has on the screen.

For example, if we decide the content requires at least 100 pixels, if the ScrollViewer has 80 pixels available - make the content within the scrollable area exactly 100 pixels, and show scroll bars. If the size available for the ScrollViewer is 200 pixels (more than the minimum 100 pixels) - don't allow scrolling, and let the contained UI content use up to 200 pixels. Here's a WPF behavior for ScrollViewer that does exactly that:

```
    public class ScrollViewerMaxSizeBehavior : Behavior<ScrollViewer>
    {
        public static readonly DependencyProperty MinContentHeightProperty = DependencyProperty.Register("MinContentHeight", typeof(int),
            typeof(ScrollViewerMaxSizeBehavior), new UIPropertyMetadata() { PropertyChangedCallback = MinSizeChanged } );
        public int MinContentHeight
        {
            get { return (int)GetValue(MinContentHeightProperty); }
            set { SetValue(MinContentHeightProperty, value); }
        }

        public static readonly DependencyProperty MinContentWidthProperty = DependencyProperty.Register("MinContentWidth", typeof(int),
            typeof(ScrollViewerMaxSizeBehavior), new UIPropertyMetadata() { PropertyChangedCallback = MinSizeChanged });
        public int MinContentWidth
        {
            get { return (int)GetValue(MinContentWidthProperty); }
            set { SetValue(MinContentWidthProperty, value); }
        }

        protected static void MinSizeChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            var self = d as ScrollViewerMaxSizeBehavior;
            if (null == self)
            {
                return;
            }
            self.Update();
        }

        protected override void OnAttached()
        {
            base.OnAttached();
            this.AssociatedObject.SizeChanged += this.ParentSizeChanged;
            this.Update();
        }

        protected override void OnDetaching()
        {
            this.AssociatedObject.SizeChanged -= this.ParentSizeChanged;
            base.OnDetaching();
        }

        protected void ParentSizeChanged(Object sender, SizeChangedEventArgs e)
        {
            this.Update();
        }

        private void Update()
        {
            if (null == this.AssociatedObject)
            {
                return;
            }
            var content = this.AssociatedObject.Content as FrameworkElement;

            if ((0 >= this.AssociatedObject.ActualHeight)
                || (0 >= this.AssociatedObject.ActualWidth))
            {
                // The attached ScrollViewer was probably not laid out yet, or has zero size.
                this.AssociatedObject.VerticalScrollBarVisibility = ScrollBarVisibility.Disabled;
                this.AssociatedObject.HorizontalScrollBarVisibility = ScrollBarVisibility.Disabled;
                return;
            }

            int minHeight = this.MinContentHeight;
            int minWidth = this.MinContentWidth;

            if ((minHeight <= 0) || (minWidth <= 0))
            {
                // Probably our attached properties were not initialized. By default we disable the scrolling completely,
                // to prevent exceptions from infinitely-sized objects within us.
                this.AssociatedObject.VerticalScrollBarVisibility = ScrollBarVisibility.Disabled;
                this.AssociatedObject.HorizontalScrollBarVisibility = ScrollBarVisibility.Disabled;
                return;
            }

            this.AssociatedObject.SizeChanged -= this.ParentSizeChanged;

            if (this.AssociatedObject.ActualHeight < minHeight)
            {
                this.AssociatedObject.VerticalScrollBarVisibility = ScrollBarVisibility.Auto;
                if (null != content)
                {
                    content.MaxHeight = minHeight - (content.Margin.Bottom + content.Margin.Top);
                }
            }
            else
            {
                this.AssociatedObject.VerticalScrollBarVisibility = ScrollBarVisibility.Disabled;
                if (null != content)
                {
                    content.MaxHeight = Double.PositiveInfinity;
                }
            }

            if (this.AssociatedObject.ActualWidth < minWidth)
            {
                this.AssociatedObject.HorizontalScrollBarVisibility = ScrollBarVisibility.Auto;
                if (null != content)
                {
                    content.MaxWidth = minWidth - (content.Margin.Left + content.Margin.Right);
                }
            }
            else
            {
                this.AssociatedObject.HorizontalScrollBarVisibility = ScrollBarVisibility.Disabled;
                if (null != content)
                {
                    content.MaxWidth = Double.PositiveInfinity;
                }
            }

            this.AssociatedObject.SizeChanged += this.ParentSizeChanged;
        }
    }

An here's how to use it in a XAML file (assuming the above class was defined in a namespace known as "custom" within the XML namespace):
Somewhere at the top: xmlns:Interactivity="clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity"
Then:

    <ScrollViewer Style="{StaticResource AppHost_ScrollViewer}">
        <Interactivity:Interaction.Behaviors>
            <custom:ScrollViewerMaxSizeBehavior MinContentWidth="600"
                                                MinContentHeight="500"/>
        </Interactivity:Interaction.Behaviors>

        <!-- content -->

    </ScrollViewer>
```

### Almost, but not quite

Continuing the 100-pixel example - what happens if we have some _statically sized_ content (no infinite sizes) that requires more than what we defined as a minimum? For example, what if instead of a dynamically-sizable grid we have content that requires 200 pixels, and the window size is 100 pixels? In this case, the previous solution is bad: it will not allow scrolling to see the full 200 pixels. So our dual-behavior needs to know, somehow, if the content within it can expand at all an infinite size (and therefore requires the dual behavior defined above to prevent explosion to infinite size). Because if the content has a _finite_ size, we simply want the ScrollViewer to behave as usual and allow scrolling to see that maximum size.

### Conclusion - what's missing in WPF (and possibly other UI frameworks)

What I'd expect is that there would be some property on UI controls specifying whether or not this element can expand infinitely (in height, width, or both). If any node in the UI's element containment tree has this "infinite size" property set, then the ScrollViewer that contains this tree must act using the dual behavior and must have a minimum size defined (smaller size means we allow scrolling and set the content exactly to the minimum size; bigger-than-minimum size means we don't allow scrolling and give the content whatever space we have and no more). If the "infinite size" property, propagated to the ScrollViewer from the contained tree is _not_ set, the ScrollViewer acts like a regular ScrollViewer - allowing the content to grow to whatever size it needs, and showing scroll bars if needed.

For now, since there is no such feature in WPF, I'll be using the aforementioned ScrollViewer behavior with appropriately defined minimum sizes for those screens that need them - hard coded, ugly, but works.
