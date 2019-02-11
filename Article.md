# Xamarin.Forms + SkiaSharp: Awesome cross-platform animations in your mobile app

It gives me great pleasure to share with you about some of my favorite topics in software development: computer graphics and mobile applications. I've been in love with graphics since I was a kid, I loved drawing on paper and when my parents got me my first home computer, an amazing ZX Spectrum, I became completely fascinated by computer graphics. 

SkiaSharp is a .NET open-source wrapper library of the Skia graphics engine, and in combination with Xamarin.Forms, which is a great cross-platform open-source mobile app development framework, you can give your mobile apps a new life.


##  Skia, the star performer

**Skia** is a high performance and open-source 2D graphics engine written in C++. It's owned and backed by Google and as a proof of its solid performance, it's been used in several mainstream products, such as Google Chrome, Chrome OS, Android or Mozilla FireFox, just to name a few. The Skia repository can be find here: [https://github.com/google/skia](https://github.com/google/skia)

Skia has native library versions for Android, iOS, Mac, Windows, and ChromeOS.

As you'd expect from a 2D graphics engine, Skia has API to draw graphics primitives such as text, geometries and images. It has an immediate rendering mode API, so for every call you do with the Skia API, it's going to be quickly drawn on the screen. 


## A *sharp* Skia

**SkiaSharp** is a .NET wrapper library over the Google's Skia, developed and maintained by Xamarin engineers and, of course, it's completely open-source under the MIT license. You can check-out its source code repository on Github: [https://github.com/mono/SkiaSharp](https://github.com/mono/SkiaSharp)

On non-Windows platforms, SkiaSharp runs on top of Xamarin and, as with any .NET library, the easiest way to add SkiaSharp to your projects is by NuGet: [https://www.nuget.org/packages/SkiaSharp](https://www.nuget.org/packages/SkiaSharp)

### SkiaSharp canvas view

In order to actually see on the screen what you draw with SkiaSharp API, SkiaSharp provides a canvas control on every platform it supports. This is done by SkiaSharp.Views, an extension library sitting on top of SkiaSharp: [https://www.nuget.org/packages/SkiaSharp.Views](https://www.nuget.org/packages/SkiaSharp.Views). 

Here are few examples of the canvas native control which SkiaSharp provides on some platforms:

```csharp
// Xamarin.iOS
public class SKCanvasView : UIView, IComponent
{
     public event EventHandler<SKPaintSurfaceEventArgs> PaintSurface;
}

// Xamarin.Android
public class SKSurfaceView : SurfaceView, ISurfaceHolderCallback
{
     public event EventHandler<SKPaintSurfaceEventArgs> PaintSurface;
}

// Mac
public class SKCanvasView : NSView
{
    public event EventHandler<SKPaintSurfaceEventArgs> PaintSurface;
}

// UWP
public partial class SKXamlCanvas : Canvas
{
    public event EventHandler<SKPaintSurfaceEventArgs> PaintSurface;
}

// WPF
public class SKElement : FrameworkElement
{
    public event EventHandler<SKPaintSurfaceEventArgs> PaintSurface;
}
```
As you know, Xamarin.Forms has its own cross-platform API and controls wrapping the native APIs and controls. Therefore, for Xamarin.Forms apps, SkiaSharp has a dedicated library called SkiaSharp.Views.Forms: [https://www.nuget.org/packages/SkiaSharp.Views.Forms](https://www.nuget.org/packages/SkiaSharp.Views.Forms) 

SkiaSharp.Views.Forms provides a Xamarin.Forms control for the canvas, the `SkCanvasView`:  

```csharp
// Xamarin.Forms
public class SKCanvasView : View, ISKCanvasViewController
{
     public event EventHandler<SKPaintSurfaceEventArgs> PaintSurface;
}
```
Under the hood, the Xamarin.Forms renderers for `SKCanvasView`  use the same native views which share same implementation as in `SkiaSharp.Views`

### Drawing on the SkiaSharp canvas view

As you can see above, on all platforms, the `SkCanvasView` has a `PaintSurface` event.  It fires the event when the canvas needs to be painted, either because the view was resized or because you called `InvalidateSurface()` method on it:

```
_skCanvasView.InvalidateSurface();
```

On the event handler, you get an instance of `SkCanvas` which you can use to draw with the SkiaSharp API:

```csharp
...
     _skCanvasView.PaintSurface += SkCanvasViewRequiredPainting;
...

 void SkCanvasViewRequiredPainting(object sender, SKPaintSurfaceEventArgs e)
{
    SKSurface skSurface = e.Surface;
    SKCanvas skCanvas = skSurface.Canvas;
    
    skCanvas.Clear();

    var paint = new SKPaint()
    {
        Style = SKPaintStyle.Stroke,
        Color = Color.Blue.ToSKColor(),
        StrokeWidth = 10
    };

    // Draw on the skCanvas instance using SkiaSharp API
    skCanvas.DrawRect(x, y, width, height, paint);
}
```

I think it's impressive to see how the combination of Xamarin, Xamarin.Forms and SkiaSharp brings .NET and Skia together on so many different platforms and provide a cross-platform .NET API for mobile app development and Skia!

This article talks about using Xamarin.Forms and SkiaSharp, so what you will read about next uses  the `SkCanvasView` from the SkiaSharp.Views.Forms library.

## Implementing an animated highlight in a mobile app with Xamarin.Forms and SkiaSharp

As an example, I'm going to show how to build a highlight which moves between inputs and a button on a sign-up form, creating a captivating animation effect. The app is built with Xamarin.Forms, Skia.Sharp and C# which runs seamlessly on Android and iOS. Here's a screen capture of the final application running in the iOS simulator: [https://www.youtube.com/watch?v=BBZWcWjJO_g](https://www.youtube.com/watch?v=BBZWcWjJO_g)

You can check-out the complete source code of the app in my repository on GitHub: [https://github.com/andreinitescu/AnimatedHighlightApp](https://github.com/andreinitescu/AnimatedHighlightApp)
 
The implementation is in C# and it's 100% shared across the platforms, there's no custom platform specific code involved, no Xamarin.Forms renderers and no effects were used. I haven't tested it on other platforms supported by Xamarin.Forms and SkiaSharp, but I'm sure the code  runs very well on those too.

Credit goes to the following sources and their authors for animation design idea:

 - [https://codepen.io/ainalem/pen/EQXjOR](https://codepen.io/ainalem/pen/EQXjOR)
   
 - [https://dribbble.com/shots/2580453-Health-App-Login](https://dribbble.com/shots/2580453-Health-App-Login)

### Implementing the highlight

The highlight effect is created by a combination of drawing a geometric path with SkiaSharp API and animating the visible part of the path using Xamarin.Forms animation API.  Here are the main steps I followed in my implementation:

1. Create the sign-up form layout
2. Build and draw `SkPath` on `SkCanvasView` based on the position of form elements in the container layout
3. Making certain part of `SkPath` visible using dash effect
4. Animating the highlight between elements

### Create the sign-up form layout

The sign-up form has three `Entry` elements to enter username, password and confirm password, and a `Button` to submit.  I'm using a `StackLayout` as the container for the form elements, but any other container would work:

```xml
<StackLayout x:Name="_formLayout" ...>
    <Label Text="Username"
           Style="{StaticResource FormLabelStyle}" />
    <Entry Style="{StaticResource FormEntryStyle}"
           Focused="EntryFocused"/>

    <Label Text="Password"
           Margin="0, 15, 0, 0"
           Style="{StaticResource FormLabelStyle}"/>
    <Entry IsPassword="True"
           Style="{StaticResource FormEntryStyle}"
           Focused="EntryFocused"/>

    <Label Text="Confirm Password"
           Margin="0, 15, 0, 0"
           Style="{StaticResource FormLabelStyle}"/>
    <Entry IsPassword="True"
           Style="{StaticResource FormEntryStyle}"
           Focused="EntryFocused"/>

    <Button Text="Sign-Up"
            Margin="0, 40, 0, 0"
            Style="{StaticResource FormSubmitBtnStyle}"
            Clicked="ButtonClicked"/>
</StackLayout>
```

### Build and draw SkPath on SkCanvasView based on the position of form elements in the container layout

The actual highlight line is a geometric path drawn using SkiaSharp API. Using the position of every Xamarin.Forms element on the form layout, I'm creating a `SkPath` connecting all the form elements and then draw the created `SkPath` on the `SKCanvasView` view. Here's how the complete `SkPath` looks like:

![Complete SkPath which connects form elements](complete_path.png)

To make the implementation easier, it's important that the `SKCanvasView` has the same top-left screen coordinates as the `StackLayout` form layout. This is in order to be easier to compute the translation between the position of the Xamarin.Forms element within the `StackLayout` to the SkiaSharp position used to draw the `SkPath` on the `SKCanvasView`. Here's the XAML which shows the `SkCanvasView` and the form layout wrapped in a `Grid`:

```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto"/>
    </Grid.ColumnDefinitions>
    <skiaSharp:SKCanvasView x:Name="_skCanvasView"
                            PaintSurface="SkCanvasViewRequiredPainting"
                            SizeChanged="SkCanvasViewSizeChanged" />
    <StackLayout x:Name="_formLayout" ...>
         ...
    </StackLayout>
</Grid>
```
The creation of the `SkPath` based on the position of Xamarin.Forms elements is implemented in the `CreatePathHighlightInfo` method [here](https://github.com/andreinitescu/AnimatedHighlightApp/blob/master/AnimatedHighlightApp/AnimatedHighlightApp/MainPage.xaml.cs#L129).


### Making certain part of SkPath visible using dash path effect

When an element receives focus, I make visible only the part of the `SkPath` which is meant to represent the focused element highlight:

![Visile part of the path when username Entry is focused](dash_path.png)

 This is accomplished by creating a dash path effect (`SkPathEffect`) and paint with it the `SkPath`:

```csharp
_skPaint.PathEffect = SKPathEffect.CreateDash(strokeDash.Intervals, strokeDash.Phase);
skCanvas.DrawPath(skPath, paint);
```
As you can see, the `CreatesDash` API  takes an **interval array** and a **phase**. 

The `intervals` is an array of float values that indicate the **length of the "on" interval** and the **length of "off" interval**. For example an intervals array with elements 10, 5 set on a line path, creates the effect of seeing 10 pixels followed by a gap of 5 pixels (assuming the stroke width is 1 pixel), and this succession repeats along the path:

![Dash intervals](dash_intervals.png)

The rule for this intervals array is that the array must contain least two values and it must be of an even number of values, and the values are interpreted as a succession of "on" and "off" intervals.

The `phase` value represents the **offset** used to draw the beginning of the dash. For example, if we have an interval of 10, 5 and phase is 5,  we will see something like the following:

![Dash intervals with phase](dash_intervals_phase.png)

To highlight the first `Entry` for example, the "on" interval is the width of the `Entry`, the "off" interval is the remaining length of the path, and the phase is zero.

There's more to know about how the stroke width and cap influence the path which you can read about in the excellent Xamarin documentation for Skia.Sharp [here](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/graphics/skiasharp/paths/dots).

As part of creating the path for the highlight, beside building the actual `SkPath`, I also build an array of dashes representing the dash path intervals corresponding to highlighting every  `Entry` and `Button` elements:

![Dash frames](dash_frames.png)

```csharp
class HighlightPath
{
    readonly Dictionary<int, StrokeDash> _strokeDashList = new Dictionary<int, StrokeDash>();
    ...
}

class StrokeDash
{
    public float[] Intervals { get; set; }
    public float Phase { get; set; }
    ...
}
```
I'm using the position of the element on the form layout as a key to know how get the dash based on the focused element. 

In my form, I have three `Entry` elements and one  `Button`. The dash list will contain 4 entries representing the dash values which makes the path visible when every element has focus. Here is a screenshot with the dash values from debugger:

![Dash list](dash_frame_list.png)


### Animating the highlight between elements

In order to make the highlight appear like it's moving between the form elements, I animate the dash values (intervals and phase), from current dash values to the precalculated dash values corresponding to the element which must show highlight.

I started with creating my own `StrokeDashAnimation` class which encapsulates animating the stroke dash `intervals` and `phase` values:

```csharp
class StrokeDashAnimation
{
    StrokeDash _currStrokeDash;

    public StrokeDash From { get; }
    public StrokeDash To { get; }
    public TimeSpan Duration { get; }
    public Easing Easing { get; }

    public StrokeDashAnimation(StrokeDash from, StrokeDash to, TimeSpan duration)
    {
        From = from;
        To = to;
        Duration = duration;
    }
   ...
}
```
I'm using `StrokeDash` class to encapsulate current dash value, which has the intervals and phase properties updated separately by every animation.

If you haven't worked with animations in Xamarin.Forms, the framework has a very simple but very powerful support for creating animations based on animating a `double` value. 

The way the `Animation` works is very simple: you give it a start `double` value, a `double` end value and an easing type. `Animation` uses easing to compute the interpolated value between the start and the end values. Once started using the `Commit` method, an `Animation` instance will call your callback for every computed value, starting with the given start value until end value is reached.

`Animation` can hold other `Animation` instances and when you start the parent animation it starts its child animations. 

You can read more about Xamarin.Forms animation and its capabilities here: [https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/animation/](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/animation/)

In my implementation, I create an animation holding three inner animations for every stroke dash property: interval "on", interval "off" and phase:

```csharp
class StrokeDashAnimation
{
    StrokeDash _currStrokeDash;

    public StrokeDash From { get; }
    public StrokeDash To { get; }
    public TimeSpan Duration { get; }
    public Easing Easing { get; }

    public StrokeDashAnimation(StrokeDash from, StrokeDash to, TimeSpan duration)
    {
        From = from;
        To = to;
        Duration = duration;
    }

    public void Start(Action<StrokeDash> onValueCallback)
    {
        _currStrokeDash = From;

        var anim = new Animation((v) => onValueCallback(_currStrokeDash));

        anim.Add(0, 1, new Animation(
            callback: v => _currStrokeDash.Phase = (float)v,
            start: From.Phase,
            end: To.Phase,
            easing: Easing));

        anim.Add(0, 1, new Animation(
            callback: v => _currStrokeDash.Intervals[0] = (float)v,
            start: From.Intervals[0],
            end: To.Intervals[0],
            easing: Easing));

        anim.Add(0, 1, new Animation(
            callback: v => _currStrokeDash.Intervals[1] = (float)v,
            start: From.Intervals[1],
            end: To.Intervals[1],
            easing: Easing));

        anim.Commit(
            owner: Application.Current.MainPage,
            name: "highlightAnimation",
            length: (uint)Duration.TotalMilliseconds);
    }
}
```
When the focus changes to an `Entry` or `Button` is clicked, I start the animation by animating from the current dash values to the precalculated dash values:

```csharp
void DrawDash(SKCanvasView skCanvasView, StrokeDash fromDash, StrokeDash toDash)
{
    if (fromDash != null)
    {
        var anim = new StrokeDashAnimation(
            from: fromDash,
            to: toDash,
            duration: _highlightSettings.AnimationDuration);

        anim.Start((strokeDashToDraw) => RequestDraw(skCanvasView, strokeDashToDraw));
    }
    else
        RequestDraw(skCanvasView, toDash);
}

void RequestDraw(SKCanvasView skCanvasView, StrokeDash strokeDashToDraw)
{
    _highlightState.StrokeDash = strokeDashToDraw;
    skCanvasView.InvalidateSurface();
}
```
For every new computed stroke dash value, I invalidate the `SkCanvasView` surface in order to make it fire its `PaintSurface` event. On the paint event handler, I draw the path with the new dash values kept by `_highlightState.StrokeDash`: 

```csharp
public void Draw(SKCanvasView skCanvasView, SKCanvas skCanvas)
{
    skCanvas.Clear();

    if (_highlightState == null)
        return;

    if (_skPaint == null)
        _skPaint = CreateHighlightSkPaint(skCanvasView, _highlightSettings, _highlightState.HighlightPath);

    StrokeDash strokeDash = _highlightState.StrokeDash;
    // Comment the next line to see whole path without dash effect
    _skPaint.PathEffect = SKPathEffect.CreateDash(strokeDash.Intervals, strokeDash.Phase);
    skCanvas.DrawPath(_highlightState.HighlightPath.Path, _skPaint);
}
```

## Closing words

I hope you can see the potential of combining the two awesome cross-platform APIs, Xamarin.Forms and SkiaSharp. Both frameworks have easy to use but powerful cross-platform APIs, and would not exist without the giant on which shoulders they are standing: Xamarin.

I hope you enjoyed reading this article, if you have any questions or feedback, feel free to reach out to me on [Twitter](https://twitter.com/nitescua) or on [ my blog](https://andreinitescu.blogspot.com).


