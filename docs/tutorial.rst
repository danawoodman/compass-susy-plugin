=============
Susy Tutorial
=============

Installation:
=============

Susy runs on `Compass <http://www.compass-style.org/>`_ and `Sass <http://www.sass-lang.com>`_, which run in the command line, so you will have to learn both. However, you can start by installing Susy, and the others will tag along without any help at all. To install, you will need `RubyGems <http://www.rubygems.org/>`_. Eventually this will get much simpler, but for now this is the way things are. In your terminal, enter::

  gem install compass-susy-plugin

Some OSX/linux users will need to add ``sudo`` to the beginning of that line. Then you can create a project::

  compass create myproject -r susy -u susy
  cd myproject
  compass compile

Defining grids:
===============

Let’s make sure we’re talking the same language here. As far as myself and Susy are concerned, a "grid" is a set number of "columns" and intervening "gutters" inside a "container" of any width and flexibility. The container can have optional "side-gutters" that pad the grid from the surrounding "document".

It looks something like this:

.. image:: http://susy-media.oddbird.net/media/adminfiles/grid-system.png
   :width: 461
   :alt: labled susy grid demo

"Grid Elements" are the actual content objects that we want to align to our grid. Grid elements can span any number of columns and gutters, and can "nest" inside each other. Let’s start to build one!

Building a grid in CSS:
=======================

At this point it will be assumed that you know at least the basics of HTML and CSS.

Our grid will be a simple HTML page - a simplified version of the one you are looking at *right now*. It will include a header for branding, a side-bar for navigation, a main section for content, a secondary sidebar for asides, and a footer for the site info. Here’s some HTML5 to get us started::

  <!DOCTYPE html>
  <html>

  <head>
  <meta charset="utf-8" />
  <title>Susy Tutorial</title>
  <link rel="stylesheet" href="stylesheets/screen.css" media="screen" />
  </head>

  <body>
  <div id="page">

  <h1>Susy: un-obtrusive grids for designers</h1>

  <nav>
  <ul>
  <li><a href="">Tutorial</a></li>
  <li><a href="">Source</a></li>
  <li><a href="">Compass</a></li>
  <li><a href="">Sass</a></li>
  </ul>
  </nav>

  <div role="main">
  <article>
  <h2>The Main Content</h2>
  <p>This is where all the main content of our page will live.</p>
  </article>
  <aside>
  <h2>note:</h2>
  <p>we can add side-notes here</p>
  </aside>
  </div>

  <footer>
  <p>this site is built using susy!</p>
  </footer>

  </div>
  </body>
  </html>

Now, if we are going to turn that HTML into a strong grid, we’ll have to do some planning and a bit of math before we write our CSS. While Susy will write most of this CSS for us, it is important to start with an understanding of what Susy is doing and why. If you already know, feel free to skip ahead.

To show you how simple this will become, our final code will look like this::

  // Imports --------------------------------------------------------------*/

  @import "base";

  /* Layout --------------------------------------------------------------*/

  #page {
    @include container;
    @include susy-grid-background;
  }

  /* Header --------------------------------------------------------------*/

  h1 {
    @include full;
    @include prefix(3);
  }

  /* Nav --------------------------------------------------------------*/

  nav {
    @include columns(3);
    @include alpha;
  }

  /* Content --------------------------------------------------------------*/

  div[role="main"] {
    @include columns(9);
    @include omega;
    article {
      @include columns(6,9);
    }
    aside {
      @include columns(3,9);
      @include omega(9);
    }
  }

  /* Footer --------------------------------------------------------------*/

  footer {
    @include full;
    @include pad(3,3);
  }

Defining the container:
-----------------------

Let’s say we want a 12-column grid, where each column is 4em wide and there are 1em gutters between columns. Let’s also add 1em side-gutters to each side of that for padding against the edges of the browser window::

  12*4em [columns] + 11*1em [gutters] + 2*1em [side-gutters] = 61em

So our container needs a width of 61em::

  #page {
    width: 61em;
  }

But let’s make our grid responsive to small browser sizes as well, so we never activate the horizontal scroll bar::

  #page {
    width: 61em;
    max-width: 100%;
  }

We’ll also want to center it in the document and factor for float clearing, IE hasLayout, and other possible issues, expanding our simple CSS out to::

  #page {
    *zoom: 1;                   /* hasLayout */
    margin: auto;               /* centering */
    width: 61em;                /* grid container size */
    max-width: 100%;            /* responsive layout */
  }

  #page:after {                 /* float clearing */
    content: "\0020";
    display: block;
    height: 0;
    clear: both;
    overflow: hidden;
    visibility: hidden;
  }

Laying out our elements:
------------------------

And we're finally ready to lay out our elements in relation to that grid. Let's start with the page header::

  <h1>Susy: un-obtrusive grids for designers</h1>

We're going to give it a grid-row all it's own, and then push the text in 3 columns from the left to align with our main content. Full-width is, of course, the default for block-level elements, so this should be easy. All we need to do for the width is account for the side-gutters. For that, we will need a margin of 1em on each side.

That would be simple enough on it's own, but we want this grid to be super-flexible. Hell, why not build the coolest grid possible, right? If we build it with percentages rather than ems (even though the full width is determined in ems), we gain several advantages:

* We can change the outer width to anything we want without touching the math inside. That means we can switch from elastic to flexible to fluid, at any widths we want, by changing one single property on the container. This is a huge gain when you are designing in the browser. Or when you want to adjust for new display standards a year down the road. Or when you want to be responsive to user feedback. Or...
* We can use that same feature dynamically to handle smaller browser windows. At smaller sizes we can automatically become fluid (using our max-width declaration above) so that we never get a side-scroll bar.

The only problem with this is the math. It can get painful. We won't need to worry about that in development though, because Susy will take care of it. For now, however, let's see what is involved. We'll start by calculating the side gutter as a fraction of the full page::

  1em [side-gutter] / 61em [context] = 1.639%

The math is similar for pushing the text in from the left by three columns. We're spanning 3 columns and 3 gutters, still in a context of 61ems::

  (3*4em [columns] + 3*1em [gutters]) / 61em [context] = 24.59%

All this math is based on the same formula::

  target / context = multiplier

And so::

  h1 {
    margin-right: 1.639%;       /* right side gutter */
    margin-left: 1.639%;        /* left side gutter */
    padding-left: 24.59%;       /* 3-column 'prefix' */
  }

Spiffy. On to the navigation, then?::

  <nav>
  <ul>
  <li><a href="">Tutorial</a></li>
  <li><a href="">Source</a></li>
  <li><a href="">Compass</a></li>
  <li><a href="">Sass</a></li>
  </ul>
  </nav>

For the navigation, we'll need to account for the left side-gutter, a span of three columns (and the two intervening gutters), and a gutter to the right. Once we get all our math right, we will simply float it to the left so other elements can stack up next to it::

  (3*4em [columns] + 2*1em [gutters]) / 61em [context] = 22.951% [width]
  1em [gutter or side-gutter] / 61em [context] = 1.639% [left and right margins]

Both our left side-gutter and our right inside gutter are 1em at this point, which simplifies things for us a bit. Let's turn that into CSS::

  nav {
    display: inline;            /* fix an IE float bug */
    float: left;                /* float left (obviously) */
    width: 22.951%;             /* span 3 columns */
    margin-right: 1.639%;       /* right inside gutter */
    margin-left: 1.639%;        /* left side gutter */
    text-align: right;          /* right-align our text */
  }

Now to align our main content::

  <div role="main">
  <article>
  <h2>The Main Content</h2>
  <p>This is where all the main content of our page will live.</p>
  </article>
  <aside>
  <h2>note:</h2>
  <p>we can add side-notes here</p>
  </aside>
  </div>

We'll be giving it a span of the 9 remaining columns, with no left gutter needed, and a right side gutter (in this case the same as an internal gutter, but could be different)::

  (9*4em [columns] + 8*1em [gutters]) / 61em [context] = 72.131% [width]
  1em [side-gutter] / 61em = 1.639% [right margin]

Giving us::

  div[role="main"] {
    display: inline;            /* IE fix */
    float: right;               /* float the last element in a row right */
    width: 72.131%;             /* span 9 columns */
    margin-right: 1.639%;       /* right side gutter */
    #margin-left: -1em;         /* hack for IE6-7 sub-pixel rounding */
  }

Inside that div our math changes a bit. We are no longer in a context of 61em, we no longer have to worry about side-gutters, and the final elements in a row no longer get any gutter attached to the right-hand side. Let's stack up our article and aside to span 6 and 3 columns respectively::

  (6*4em + 5*1em) / (9*4em + 8*1em) = 65.909%
  (3*4em + 2*1em) / (9*4em + 8*1em) = 31.818%
  1em / (9*4em + 8*1em) = 2.273%

And the CSS::

  article {
    display: inline;            /* IE fix */
    float: left;                /* float left */
    width: 65.909%;             /* span 6 of 9 columns */
    margin-right: 2.273%;       /* right inside gutter */
  }

  aside {
    display: inline;            /* IE fix */
    float: right;               /* float the last element in a row right */
    width: 31.818%;             /* span 3 of 9 columns */
    margin-right: 0;            /* no gutter */
    #margin-left: -1em;         /* hack for IE6-7 sub-pixel rounding */
  }

All we have left is the footer, which is back in the 61em context and will be treated much like the header. The only difference is that we want to push it in 3 columns from both sides, keep the font size, and push it around less vertically. We also want it to clear all our floats::

  footer {
    clear: both;                /* footer clears all previous floats */
    margin-right: 1.639%;       /* right side gutter */
    margin-left: 1.639%;        /* left side gutter */
    padding-left: 24.59%;       /* 3-column 'prefix' */
    padding-right: 24.59%;      /* 3-column 'suffix' */
  }

Done! Here's your final CSS::

  /* Susy --------------------------------------------------------------*/

  #page {
    *zoom: 1;                   /* hasLayout */
    margin: auto;               /* centering */
    width: 61em;                /* grid container size */
    max-width: 100%;            /* responsive layout */
  }

  #page:after {                 /* float clearing */
    content: "\0020";
    display: block;
    height: 0;
    clear: both;
    overflow: hidden;
    visibility: hidden;
  }

  /* Header -----------------------------------------*/

  h1 {
    margin-right: 1.639%;       /* right side gutter */
    margin-left: 1.639%;        /* left side gutter */
    padding-left: 24.59%;       /* 3-column 'prefix' */
  }

  /* Nav ---------------------------------------------*/

  nav {
    display: inline;            /* fix an IE float bug */
    float: left;                /* float left (obviously) */
    width: 22.951%;             /* span 3 columns */
    margin-right: 1.639%;       /* right side gutter */
    margin-left: 1.639%;        /* left inside gutter */
    text-align: right;          /* right-align our text */
  }

  /* Content ------------------------------------------*/

  div[role="main"] {
    display: inline;            /* IE fix */
    float: right;               /* float the last element in a row right */
    width: 72.131%;             /* span 9 columns */
    margin-right: 1.639%;       /* right side gutter */
    #margin-left: -1em;         /* hack for IE6-7 sub-pixel rounding */
  }

  article {
    display: inline;            /* IE fix */
    float: left;                /* float left */
    width: 65.909%;             /* span 6 of 9 columns */
    margin-right: 2.273%;       /* right inside gutter */
  }

  aside {
    display: inline;            /* IE fix */
    float: right;               /* float the last element in a row right */
    width: 31.818%;             /* span 3 of 9 columns */
    margin-right: 0;            /* no gutter */
    #margin-left: -1em;         /* hack for IE6-7 sub-pixel rounding */
  }

  /* Footer -------------------------------------------*/

  footer {
    clear: both;                /* footer clears all previous floats */
    margin-right: 1.639%;       /* right side gutter */
    margin-left: 1.639%;        /* left side gutter */
    padding-left: 24.59%;       /* 3-column 'prefix' */
    padding-right: 24.59%;      /* 3-column 'suffix' */
  }

Now imagine building a complex grid with all that math and repeated code. Many of you may not even need to imagine: you've done it on a daily basis. Now let's look at how Susy can simplify all of that for you.

Building a grid with Susy:
==========================

Every single line of CSS that we have written so far can be handled more simply and dynamically with Susy. Susy uses abstraction and math functions to make sure you can build and adjust your grid using just a few variables and commands, rather than doing all the math and writing all the code yourself.

For this we'll assume you already know the basics of Sass (either syntax) and Compass. You can handle the compiling of files, etc. I'll just explain how the Susy part works.

Defining the grid:
------------------

We have to start by telling Susy about the grid that we want to build. Susy starts us out with a set of variables to do that. You can find them in the ``base`` partial (file beginning with ``_``) in your susy project sass directory::

  // Grid --------------------------------------------------------------

  $total-cols: 12
  $col-width: 4em
  $gutter-width: 1em
  $side-gutter-width: $gutter-width

``$total-cols`` represents the number of columns in our grid, ``$col-width`` is the width of each column, ``$gutter-width`` is the width of space between columns, and ``$side-gutter-width`` is the space on either side of the page.

These variables can and should be edited to fit any grid you would like to build. For a fixed grid you can simply change your grid units to px. For a fluid grid you can change them to percentages, assuming they all add up to 100% or less.

But we won't do that now. For now we want an elastic grid, and the default one is exactly to our specifications. That's lucky. Coincidence or fate? We'll never know.

Now we just need to build that. If you open your ``screen`` sass file you will see::

  // Imports --------------------------------------------------------------

  @import "base";

  /* Layout -------------------------------------------------------------- */

  .container {
    @include container;
    @include susy-grid-background; }

The ``.container`` element has the ``container`` mixin included, which handles sizing, centering, a clear-fix and has-layout. It also has the ``susy-grid-background`` mixin set up to show us our grid. All you need to change to match your own markup is the ``.container`` selector, and you are ready to go with a Susy grid already in place. Since our demo uses ``#page`` as the container, we will make that one simple change.

Laying out our elements:
------------------------

Let's take it from the top again, starting with that ``h1`` banner. We want it to span the full width of the grid container, minus the side-gutters, and then we want to pad the left by 3 columns, and give some vertical space. No problem.

There is one more term we need to establish. In order to properly apply or remove gutters and side-gutters at the right moments, Susy needs to know whether a given element lives in a "root" or "nested" context.

In Susy, the context is the default full-span of the block, or the space that is available for it to expand into naturally. That is normally the width of the parent. Susy context is given in terms of columns-spanned. If the parent spans N columns, the context is N. If the parent spans N columns plus the side gutters, there is no context (I call this the 'root'). If the parent isn’t aligned to the grid, you’re on your own.

If "context" isn't making sense to you, skip over to the `Context Demo <http://susy.oddbird.net/demo/context/>`_ for a more thorough explanation.

Our ``h1`` is in the root context. Left to it's own devices it will span the full container including the side gutters. Keep that in mind.

Susy has a simple mixin for handling elements that span their full context, and another to add a padding prefix spanning any number of columns::

  full([context])
  prefix(span, [context])

However, with Susy, we **never** pass the context when it is "root". So::

  h1 {
    @include full;
    @include prefix(3);
  }

Looks good. On the navigation: 3 columns floated left at the root context. We have a few more mixins we can use::

  columns(span, [context])
  alpha

The ``alpha`` mixin is only needed at the root context on the left-most ``columns`` element of any row. This adds on the necessary side-gutter. It doesn't take any arguments, because it is only needed in this one specific situation and always has the same effect::

  nav {
    @include columns(3);
    @include alpha;
  }

All set there, let's take care of the content: 9 columns floated right and then subdivided into a main 6-column article and a 3-columns aside. The only change here is that we'll use ``omega`` instead of ``alpha``::

  omega([context])

The difference is that we will need omega in any context, so it gets that argument passed to it (when the context is other-than-root). That is because Susy applies internal gutters to the right margins of their preceding columns. We only need ``alpha`` to take care of left side-gutters at the root, but we need ``omega`` to take care of both adding the root side-gutter and removing the final gutters later on. You can see that below. Let's do this thing::

  div[role="main"] {
    @include columns(9);
    @include omega;
    article {
      @include columns(6,9);
    }
    aside {
      @include columns(3,9);
      @include omega(9);
    }
  }

There you see everything you need to know. The div element is in a root context and so no context is passed to the mixins. But then the div *becomes* the context at 9-columns wide, and that is passed to all nested grid mixins as the second argument. That's not too hard, is it?

The footer is back in the root context, at the full width but padded in from both sides. That brings us to two new mixins we can use::

  suffix(span, [context])
  pad(prefix, suffix, [context])

``suffix`` works just like prefix did. It may be worth noting that both are subtractive when applied to a ``full`` element, because full elements have no set width applied. Where a full-width element would normally expand to all 12 columns, the added padding makes the content-box narrower rather than pushing out the borders. So 3 columns of padding leave you only with 9 columns of content. But, given the standard css box-model of padding adding to set widths, they will become additive when applied to ``columns`` elements. Assigning 3 columns to the width, and another 3 to the padding will make for a 6-column element.

``pad`` is simply a shortcut for adding both ``prefix`` and ``suffix`` at the same time. Let's put it together::

  footer {
    @include full;
    @include pad(3,3);
  }

And we're done. No math. Just columns and contexts, alphas and omegas. That's it. Susy does the rest. Here's our full code, about 1/3rd the length of our CSS and much more clear::

  // Imports --------------------------------------------------------------*/

  @import "base";

  /* Layout --------------------------------------------------------------*/

  #page {
    @include container;
    @include susy-grid-background;
  }

  /* Header --------------------------------------------------------------*/

  h1 {
    @include full;
    @include prefix(3);
  }

  /* Nav --------------------------------------------------------------*/

  nav {
    @include columns(3);
    @include alpha;
  }

  /* Content --------------------------------------------------------------*/

  div[role="main"] {
    @include columns(9);
    @include omega;
    article {
      @include columns(6,9);
    }
    aside {
      @include columns(3,9);
      @include omega(9);
    }
  }

  /* Footer --------------------------------------------------------------*/

  footer {
    @include full;
    @include pad(3,3);
  }

Moving Forward
==============

Susy is full of more flexibility and features under the surface. You can get straight to the numbers without any properties attached using the ``columns()`` and ``gutter()`` and ``side-gutter()`` functions to do your own math. You can change a setting to remove all IE hackery. And so on and on.

Susy is simply a set of functions and mixins that do math for you. That is all. There is nothing all-in-one or magical about these things, and they will break if not applied with some finesse. You won't find leakier abstractions. While we try to fill the gaps any way we can, Susy can't write your HTML and doesn't know your design. That isn't a bug, that's the way things are.

Whether you are a beginner or an expert at CSS, Susy can help you get a site off the ground more quickly and easily. Either way, you should be checking the output CSS and comparing it to your desired outcome. As always with Sass, browsers don't care what abstractions you used, they only care what CSS is in that output file. To debug means to read the CSS. As an expert you can use that knowledge to adjust your use of Susy for optimized output. For beginners, you can start to learn the tricks of the CSS trade by seeing how Susy works, and eventually you'll be an expert.