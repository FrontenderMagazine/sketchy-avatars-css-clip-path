
*The following is a guest post by [Ryan Scherf][1]. Ryan found a neat way to
give avatars kind of rough, uneven, varied edges. Kinda like they were cut out 
with scissors by someone who wasn't very good at using scissors. What's nice is 
it's naturally a progressive enhancement technique and it can be done through 
just CSS.*

With a creative and fun brand like [Quirky][2], we are always thinking about
ways to bring that vibe to the web. Throughout the site, there is a "hand drawn
" look to some elements. Without the use of lots of images, it's very difficult 
to get that hand-drawn vibe. With some light trigonometry and very basic 
knowledge of CSS
' `clip-path`, we're able to do this with relative ease and good performance.<
figure id="post-188243 media-188243" class="align-none
">

![][3]<figcaption>What we're building. Notice the uneven and varied edges on
each.</figcaption></figure>
### Why not use image masks?

For instance, a mask defined in SVG:

    img {
      mask: url(mask.svg) top left / cover;
    }

The `mask` property can reference external SVG or SVG defined in the document
by ID.

But what if you wanted a unique shape for every single avatar displayed, not
the same shape? You could programmatically generate lots of different SVG shapes
to apply. But we can achieve the same thing and get that mathematical generation
through generating`clip-path`s with (S)CSS.

### What's the browser support?

The browser support for `clip-path`, when used with a shape value like 
`polygon()`, is Chrome 24+, Safari 7+, Opera 25+, iOS 7.1+, Android 4.4+.
Firefox supports`clip-path` only with the path defined in SVG (we'll cover that
). No support in IE yet.

You'll need to use `-webkit-clip-path`, as that's the only way it's supported
right now, but probably best to drop`clip-path` on there too. If IE or Firefox
start supporting it this way, it'll likely be unprefixed.

### Clipping paths in a nutshell

There are a few different shape values you can use for CSS clipping but in our
case, the`polygon` shape is best as it gives us the most amount of points and
flexibility to create our hand-drawn effect.

You give `polygon()` a list of X, Y point values, like: 
`<x0> <y0>, <x1> <y1>, ... <xn> <yn>`. That
will draw a path around your points**in order** and crop any of the content **
outside** of the newly created shape.

    /* 
      This will create a Hexagon, with the first 
      point being the top tip of the shape 
    */
    
    .hexagon {
      clip-path: polygon(50% 0, 100% 25%, 100% 75%, 50% 100%, 0 75%, 0 25%);
    }

Here is that simple example in action:

See the Pen [Simple Hexagon][4]

### Not-so-scary math

Our hexagon is pretty cool, but it doesn't achieve a real sketchy effect quite
yet. It's quite rigid - too few lines. The best way to think of a hand-drawn 
shape is a series of small lines connecting two dots. The more dots we have, the
more short lines we create. In fact, with enough points, we could make a
`polygon` shape so smooth it mimics a `circle`.

Here is an example of using 200 points:

See the Pen [Circle via Polygon][5]

#### Where do the points come from?

Here's where a little bit of math comes in. Perhaps you took trigonometry in
high school? One of the fundamental ideas you learn in that class is regarding 
the**Unit Circle**. Basically, there is a set formula (given pi) that can
generate any number of points around a circle.<figure>

![][6]<figcaption>The unit circle (via [Wikipedia][7])</figcaption></figure>
If we were to connect our segments, we'd get a shape that looked like:<figure
>

![][8]<figcaption>Connect the dots!</figcaption></figure>
Still a little rigid, but looking a little more hand-drawn as well.

#### More Points!

We know how to make hexagons and circles with the `clip-path: polygon()`, so
how do we make it look hand-drawn?

*   Adjust the number of points (the more there are, the lower the segment
    lengths
    )
*   Add some X and Y variance (so the segments aren't uniform)

Let's bring that in SCSS and create a function to do the dirty work for us. We'
ll be using:

The most relevant math is:

    /* 
      To generate an arbitrary points on 
      the unit circle at angle t 
    */
      
    $x: cos(t);
    $y: sin(t);

And putting that in the right syntax looks like:

    $w: 160px    // Avatar width
    $n: 60;      // Number of points on the circle
    
    @function sketchAvatar() {
      $points: ();
    
      @for $i from 0 through $n {
        $points: append($points, ($w / 2) * (1 + cos((2 * pi() * $i / $n))) ($w / 2) * (1 + sin((2 * pi() * $i / $n))), comma);
      } 
      
      @return $points;
    }

This is a little hairy. What is happening is we start at the top middle of our
shape, and generate list of sets of points around the circle for 60 evenly 
spaced points.

### Bringing it altogether with variances

The above code still produces fairly bland and uniform polygons, so we'll have
to add in variance. All we need to do is adjust the points in any direction to 
give that offset feel we're looking for. The`$lower` and `$upper` variance
numbers can be just about anything depending on the look you're going for.

    $w:     120px;   // Overall width
    
    @function sketchAvatar() {
      $n: 	  60;     // Number of points
      $lower: -80;    // Lower variance
      $upper: 80;     // Upper variance
    
      $points: ();
    
      @for $i from 0 through $n {
        $points: append($points, ($w / 2) * (1 + cos((2 * pi() * $i / $n))) + (rand($lower, $upper) / 100) ($w / 2) * (1 + sin((2 * pi() * $i / $n))), comma);
      } 
      
      @return $points;
    }

We did it! Sketchy, unique avatars with CSS `clip-path: polygon()`:

See & fork the Pen [Sketchy Avatars][9]

### Making it work in Firefox

Chris here! I thought since Firefox doesn't support this done this way, but
does support the SVG syntax, we could maybe kinda polyfill it.

    .avatar {
      clip-path: polygon( ... ) /* Firefox: nope */
      clip-path: url(#clip); /* Firefox: yep */
    }

So for each avatar, I...

1.  Output the polygon points in the content property of a pseudo element (of
    an element that has a valid pseudo element like the parent div) in CSS
   
2.  Extracted that value with JavaScript
3.  Reformat the points to match the SVG format (e.g. no "px")
4.  Injected a new `<svg>` on the path with a `<clipPath>` ready to
    go
   

    $(".user").each(function(i) {
     
      var path = window.getComputedStyle(this, ':after').getPropertyValue('content');
      
      // clean house
      svgPolygonPoints = 
        path
          .replace(/px/g, "")
          .replace(/polygon/, "")
          .replace(/\(/, "")
          .replace(/\)/, "")
          .replace(/\;/g, "")
          .replace(/"/g, "")
          .replace(/\'/g, "");
        
      // To get this to actually work, create a <div> instead with this inside, see below.
      var svg = $("<svg width='0' height='0'>")
        .append("<defs><clipPath id='clip-" + (i+1) +"'><polygon points='" + svgPolygonPoints +"' /></clipPath></defs>");
      
      $("body").append(svg);
        
    });

It doesn't work! haha. Even if you force a repaint on the avatars, it just
doesn't like the injected SVG for some reason. Check out 
[Amelia's solution][10]

It's basically like:

    .user:nth-child(1)  {
      clip-path: polygon(120.04px 60px ...);
    }

becomes:

    <svg width="0" height="0">
      <defs>
        <clippath id="clip-1">
          <polygon points="120.04 60, ... "></polygon>
        </clippath>
      </defs>
    </svg>

[Twitter][11] [Facebook][12] [Google+][13] </article>

 [1]: http://twitter.com/ryanscherf
 [2]: http://quirky.com
 [3]: img/sketchy-avatars.png
 [4]: http://codepen.io/rscherf/pen/haoEk
 [5]: http://codepen.io/rscherf/pen/zpuBg
 [6]: img/unit-cricle.png
 [7]: http://en.wikipedia.org/wiki/Unit_circle
 [8]: img/path.png
 [9]: http://codepen.io/rscherf/pen/pzgKt
 [10]: http://css-tricks.com/sketchy-avatars-css-clip-path/#comment-1586734

 [11]: https://twitter.com/intent/tweet?text=Sketchy%20Avatars%20with%20CSS%20clip-path&url=http://css-tricks.com/sketchy-avatars-css-clip-path/&via=real_css_tricks

 [12]: https://www.facebook.com/sharer/sharer.php?u=http://css-tricks.com/sketchy-avatars-css-clip-path/

 [13]: https://plus.google.com/share?url=http://css-tricks.com/sketchy-avatars-css-clip-path/