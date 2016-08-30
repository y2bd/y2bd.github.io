---
layout: post
title: "Creating a Parallax Scrolling Effect on Windows Phone"
date: 2014-03-18 19:24
categories: [windows phone, csharp, programming, quill]
---

<div style='position:relative;padding-bottom:167%'><iframe src='https://gfycat.com/ifr/SimpleLiveAnnelid' frameborder='0' scrolling='no' width='100%' height='100%' style='position:absolute;top:0;left:0;' allowfullscreen></iframe></div>

_tl;dr: [Here’s a commented sample project if you’re in a hurry](https://github.com/y2bd/WPParallaxScrolling)_

Windows Phone 8’s scrollviews have a property where if you scroll past the content area, the content is squished. Nokia’s [App Social](http://www.nokia.com/us-en/apps/app-social/) has a cool but subtle effect that takes advantage of this. If you’re looking at a recommendation list within the app and try pulling the list down, you’ll notice that the header image doesn’t scroll with the rest of the list. Instead, it scrolls slower than everything else, as well as expands to fill the space made by the squished scrollview.

If you’ve ever played with an iOS device, you should know this effect well, as it’s present in so many apps on that platform. My favorite example is probably within [Day One](http://dayoneapp.com/), an awesome diary app, where within your diary entries you can set a header photo that behaves similarly.

<!--more-->

{% marginnote margin1 %}
You're going to need a header image to test with. Might I recommend [mine](/assets/posts/parallax/chitanda.jpg)?
{% endmarginnote %}

Creating this effect on Windows Phone is certainly tricky for a variety of reasons—in fact, if you play around with App Social’s implementation, you might even catch a few glitches—but with a bit of work you can hack together something like the above.

Let’s get to work! Open up VS and create a new blank WP8 project. This might work in WP7 (or even WP8.1!) but I haven’t tested it.

Here’s the XAML we’re going to be using:

```xml
<Grid x:Name="LayoutRoot"
      Background="Transparent">

    <Grid x:Name="ImageContainer"
          VerticalAlignment="Top"
          CacheMode="BitmapCache"
          RenderTransformOrigin="0.5,0.5">
        <Grid.RenderTransform>
            <CompositeTransform x:Name="ImageTransform" />
        </Grid.RenderTransform>

        <Image Source="/Assets/chitanda.jpg"
               Stretch="None"
               HorizontalAlignment="Center"
               VerticalAlignment="Top"
               x:Name="Image" />
    </Grid>

    <ScrollViewer x:Name="Scroller"
                  ManipulationMode="Control"
                  Background="#33000000">
        <Grid x:Name="ScrollGrid">
            <Grid.RowDefinitions>
                <RowDefinition Height="500" />
                <RowDefinition Height="*" />
            </Grid.RowDefinitions>

            <StackPanel x:Name="TitlePanel"
                        VerticalAlignment="Bottom"
                        Margin="0,0,0,24">
                <TextBlock Style="{StaticResource PhoneTextTitle1Style}"
                           Margin="24,0"
                           TextWrapping="Wrap"
                           Text="The Title" />
                <TextBlock Style="{StaticResource PhoneTextTitle2Style}"
                           Margin="24,0"
                           TextWrapping="Wrap"
                           Text="(with snarky subtitle)" />
            </StackPanel>

            <StackPanel x:Name="ContentPanel"
                        Grid.Row="1"
                        Background="{StaticResource PhoneChromeBrush}">
                <TextBlock HorizontalAlignment="Left"
                           TextWrapping="Wrap"
                           VerticalAlignment="Top"
                           Style="{StaticResource PhoneTextTitle3Style}"
                           Margin="24">
                  <Run Text="Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras nec erat massa. Phasellus accumsan ornare velit non vestibulum. Quisque nulla mi, condimentum sit amet sapien quis, blandit facilisis leo. Praesent pulvinar, justo at ornare aliquet, tellus massa varius metus, nec euismod dui metus a mauris. Sed eu erat et nulla varius pharetra. Suspendisse id mi nibh. Donec ac quam vel erat malesuada porttitor convallis condimentum ipsum. Donec risus lectus, auctor dictum mauris et, lacinia condimentum sem." />
                  <LineBreak />
                    <LineBreak />
                  <Run Text="Donec gravida purus non gravida volutpat. Suspendisse id libero non nunc dignissim molestie. Phasellus lobortis sit amet libero non volutpat. Duis viverra, augue ac bibendum eleifend, leo metus tincidunt neque, in malesuada dolor leo ac eros. Ut eget justo faucibus, vulputate ante ac, varius urna. Etiam blandit turpis ac leo tristique scelerisque. Phasellus ac aliquam eros. Curabitur condimentum mi ligula, et porttitor leo consectetur sit amet. " />
                    <LineBreak />
                    <LineBreak />
                    <Run Text=" Sed pellentesque sapien a diam interdum, vitae sagittis velit varius. Curabitur vehicula in velit ac porttitor. Integer nec nunc tellus. Nullam pulvinar magna eu mollis dignissim. Sed mattis at erat id euismod. Vestibulum mattis nunc nec sapien ornare, sit amet tristique diam volutpat. Nulla ut venenatis erat. " />
                </TextBlock>
            </StackPanel>
        </Grid>
    </ScrollViewer>
</Grid>
```

I also recommend dealing with the system tray in some manner, whether it’d be making it transparent or hiding it altogether:

```xml
<!-- hide the system tray -->
shell:SystemTray.IsVisible="False"

<!-- or make it transparent -->
shell:SystemTray.IsVisible="True"
shell:SystemTray.Opacity="0"
```

Let’s now jump into the code-behind of the page. First of all, you’ll notice from the XAML that the image is top-aligned to the top of the page. This isn’t what we want. Instead, we want the center of the image to be at center of the empty space. As far as I know, there isn’t an easy way to do this within XAML (without clipping the image) so let’s do it programmatically:

```csharp
public partial class MainPage : PhoneApplicationPage
{
  // this is the height of the empty space above the content
  // that we created in the XAML
  private const double EmptySpace = 500;

  // this will represent the vertical position of the image
  // we're storing it in a variable because we'll need it later
  private double _startPosition;

  public MainPage()
  {
    InitializeComponent();

    // we're putting our code in a Loaded event handler
    // because we have to wait until the image control is loaded
    // before getting its height
    Loaded += OnLoaded;
  }

  private void OnLoaded(object sender, RoutedEventArgs e)
  {
    // Here's the magic
    // To get the image where we want, we will shift it up by its height
    // Then we'll shift it down by the height of the empty space
    // Last, we'll divide that by two so we'll be directly between the top
    // of the page and the bottom of the empty space
    _startPosition = (-Image.ActualHeight + EmptySpace) / 2;

    // set the TranslateY of the CompositeTransform we created in the XAML
    // to our calculated start position
    ImageTransform.TranslateY = _startPosition;
  }
}
```

Perfect! Now Chitanda (or whatever you chose as your image) will be centered in the space rather than stuck to the top. This is very important because later on, we’ll be pulling the image down, so we don’t want it to end immediately at the top.

{% image /assets/posts/parallax/centering.png "Top-aligned versus centered" 578 460 fw %}

Now let’s get to the actual effect. Although it looks like one smooth effect, there’s actually two distinct parts. We need the image to scroll slowly when being hidden by content (the first part of the video above), and we also need to the image to “expand” when the scrollview is squished (the second part of the video). The first part is easier, so let’s do it first.

We need a way of detecting when the the scrollview is scrolled, and by how much. It turns out that the little scrollbar you see on the right is a good way of checking this. Now normally the scrollbar is buried deep within the ScrollViewer control, so we need some way of fishing it out. We’re going to use the [VisualTreeHelper](http://msdn.microsoft.com/en-us/library/system.windows.media.visualtreehelper%28v=vs.110%29.aspx) to do this:

```csharp
private void OnLoaded(object sender, RoutedEventArgs e)
{
  _startPosition = (-Image.ActualHeight + EmptySpace) / 2;

  ImageTransform.TranslateY = _startPosition;

  // extract the vertical scrollbar from our ScrollViewer named 'Scroller'
  var scrollbar =
    ((FrameworkElement) VisualTreeHelper.GetChild(Scroller, 0)).FindName("VerticalScrollBar") as ScrollBar;

  if (scrollbar != null)
  {
      // listen for scroll events
    scrollbar.ValueChanged += OnScrollbarValueChanged;
  }
}
```

Now that we’ve fetched the scrollbar, let’s see how we use it:

```csharp
private void OnScrollbarValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
{
  // if we're in positive scrolling territory
  // where positive means down
  if (e.NewValue > 0)
  {
    // we're going to scroll our Image along with the content
    // but only half as fast
    // try removing the /2.0 and see what happens
    ImageTransform.TranslateY = _startPosition - e.NewValue/2.0;

    // instead of using the scrollbar's value, you can also use the VerticalOffset
    // of the ScrollViewer itself
    // try it, see which one works better for you performance-wise
    // ImageTransform.TranslateY = _startPosition - Scroller.VerticalOffset/2.0;
  }
}
```

That’s it! Now if you scroll the content to read more of it, you’ll notice that instead of staying put, the header image now scrolls along, but slower than the content.

We’re not done yet though. If you try pulling down on the content, causing it to squish, you’ll see that nothing happens to the header image. You might think that it’s because of the `if (e.NewValue > 0)` check we did. But if you remove it, you’ll see that there’s no difference.

The problem is that the scrollbar’s value (as well as a ScrollViewer’s VerticalOffset) doesn’t respond to the squishing effect. It doesn’t become a negative value, which means that we can’t use for the second part of the parallax effect. This is where things get complicated.

The ScrollViewer control has a weird property called `Content`, which is a UIElement representing whatever you put inside of it. That UIElement, like all UIElements, has a RenderTransform property. It turns out that when you squish a ScrollViewer, it sets that RenderTransform to a new [CompositeTransform](http://msdn.microsoft.com/en-us/library/system.windows.media.compositetransform%28v=vs.95%29.aspx) instance. It only does _once a squish occurs_ though. That means that if you tried accessing `Scroller.Content.RenderTransform` in our OnLoaded function, you wouldn’t get a CompositeTransform object. You’d instead get a MatrixTransform, and we can’t do anything useful with that. We’re going to need to detect when it becomes a CompositeTransform, and the MouseMove event is perfect for that:

```csharp
private void OnLoaded(object sender, RoutedEventArgs e)
{
  _startPosition = (-Image.ActualHeight + EmptySpace) / 2;

  ImageTransform.TranslateY = _startPosition;

  var scrollbar =
    ((FrameworkElement) VisualTreeHelper.GetChild(Scroller, 0)).FindName("VerticalScrollBar") as ScrollBar;

  if (scrollbar != null)
  {
    scrollbar.ValueChanged += OnScrollbarValueChanged;
  }

  // when the mouse (the user tapping his screen) moves
  // within the ScrollViewer, there's a chance that
  // the ScrollViewer's Content's RenderTransform will be set to
  // a new CompositeTransform that we need to grab a hold of
  Scroller.MouseMove += ScrollerOnMouseMove;
}

private void ScrollerOnMouseMove(object sender, MouseEventArgs mouseEventArgs)
{
  // grab the content element
  var uiElement = Scroller.Content as UIElement;
  if (uiElement != null)
  {
    // try to grab its transform as a CompositeTransform
    var transform = uiElement.RenderTransform as CompositeTransform;

    // if it's actually a CompositeTransform
    if (transform != null)
    {
      // we're good, let's go to town!
    }
  }
}
```

Awesome, we got our hook! Now, we can use the TranslateY of the CompositeTransform and move our header image just like we did with the scrollbar. There’s a slight problem though.

This method, ScrollerOnMouseMove, will only be called when the “mouse” is moving within the ScrollViewer. There’s one case that it doesn’t handle: when the user squishes the content and then lets go, letting the content spring back into place. ScrollerOnMouseMove won’t be called then, so our image will never slide back into place.

We could use the MouseUp event for this to snap the image back into place, but that’ll be a harsh snapping instead of the smooth sliding that we want.

Instead, we’ll recall that the TranslateY property of a CompositeTransform is actually a dependency property, which means that it can be binded to:

```csharp

// this will be our backing property for the binding
private double _verticalOffset;
public double VerticalOffset
{
  get { return _verticalOffset; }
  set
  {
    _verticalOffset = value;

    onVerticalOffsetChanged();
  }
}

private void onVerticalOffsetChanged()
{
  // now let's do the same thing we did with the scrollbar

  // we only want to handle the squishes here, as the
  // scrollbar events handle the normal scrolling
  // so we'll only respond to the squishes, when the content
  // is being moved down
  if (VerticalOffset >= 0)
  {
    ImageTransform.TranslateY = _startPosition + VerticalOffset/2.0;
  }
}

private void ScrollerOnMouseMove(object sender, MouseEventArgs mouseEventArgs)
{
  // grab the content element
  var uiElement = Scroller.Content as UIElement;
  if (uiElement != null)
  {
    // try to grab its transform as a CompositeTransform
    var transform = uiElement.RenderTransform as CompositeTransform;

    // if it's actually a CompositeTransform
    if (transform != null)
    {
      // we're good, let's go to town!

      // let's set up the binding in a standard manner
      var binding = new Binding("VerticalOffset");

      // in a perfect world, we use a reverse OneWay binding, where
      // the DP we're binding to can set the backing property
      // as this doesn't exist in the world of Windows Phone
      // we're going to cheat and use TwoWay
      binding.Mode = BindingMode.TwoWay;
      binding.Source = this;

      BindingOperations.SetBinding(transform, CompositeTransform.TranslateYProperty, binding);

      // we're going to release the event handler
      // since we only need to bind once
      Scroller.MouseMove -= ScrollerOnMouseMove;
    }
  }
}
```

And now we’re done! For real this time! If you pull down on the content, squishing it, you’ll see your header image slowly slide down to fill the space. And if you release, the image will slide up with the content.

All is not perfect in the land of Windows Phone though.

I mentioned way back in the beginning that there was a peculiar line in our XAML that I’d discuss later. That line is `ManipulationMode="Control"`. See, by default the ManipulationMode of a ScrollViewer is set to “System”. What this amounts to is that scrolling events are optimized and reports are chunked to get the smooth scrolling Windows Phone is known for. Setting it to “Control” throws some of these optimizations out of the window, but it’s the only way we can get accurate values for the scroll offsets. If you try setting it back to “System”, you’ll see that our smooth parallax scrolling becomes chunky, and in the case of handling squishes fails completely.

You’ll have to settle for the (albeit slight) performance decrease if you want this effect—you’ve probably already noticed that sometimes the header image stutters. You’ll also have to settle for some sporadic bugs that I’ve noticed, including the ScrollViewer getting “stuck” in the squished state, or it jumping to the top or the bottom if you flick randomly.

If you’re okay with it though (or you find some way to further optimize this, please tell me if you do!), this is a really nice looking effect, and it honestly isn’t a ton of trouble to implement.

If you don’t feel like copy-pasting all of this code and organizing it yourself, you can download a fully-commented example here: [https://github.com/y2bd/WPParallaxScrolling](https://github.com/y2bd/WPParallaxScrolling)

Here are two articles I used when figuring this out:

*   [Pull-down-to-refresh a WP7 ListBox or ScrollViewer](http://blogs.msdn.com/b/jasongin/archive/2011/04/13/pull-down-to-refresh-a-wp7-listbox-or-scrollviewer.aspx)
*   [How to keep a UI element in view when scrolling a page in Windows Phone](http://developer.nokia.com/community/wiki/How_to_keep_a_UI_element_in_view_when_scrolling_a_page_in_Windows_Phone)
