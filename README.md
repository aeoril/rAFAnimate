## rAFAnimate

### requestAnimationFrame animation engine
rAF based abstraction that handles timing and control of one or more independent
multiple animations.

### API

#### rAFAnimate( animate, options, context )

    var analogClockRAFed = rAFAnimate(

      // animate function - same as below, must be reentrant
      analogClock,

      // options object
      {
        msPerTick:  1000.0,   // 1 second per tick for second hand
        run:        Infinity, // Once started, run continuously,
        secondHand: Infinity, // Move second hand continuously
        minuteHand: Infinity, // Move minute hand continuously
        hourHand:   Infinity  // Move hour hand continuously
      },

      // context (optional)
      analogClockContext      // 2D Canvas context for Analog Clock
    );

    var ticksPerSecond = 60; // same as rAF frames per second on most systems
    var msPerSecond = 1000.0;
    var msPerTick = msPerSecond / ticksPerSecond; // 16.67, same as frame ms
                                                  // on most systems
    var ticksPerDay = msPerTick * msPerSecond * 60.0 * 60.0 * 24.0;

    var analogClockSweepRAFed = rAFAnimate(

      // animate function - same as above, must be reentrant
      analogClock,

      // options object
      {
        msPerTick:  msPerTick,   // Sweep second hand continuously
        run:        ticksPerDay, // Once started, run for 24 hours,
        dial:       Infinity,    // Once started, show dial continuously
        secondHand: Infinity,    // Move second hand continuously
        minuteHand: Infinity,    // Move minute hand continuously
        hourHand:   Infinity     // Move hour hand continuously
      },

      // context (optional)
      analogClockSweepContext    // 2D Canvas context for Analog Sweep Clock
    );

    analogClockRAFed()      // Start analog clock
    analogClockSweepRAFed() // Start analog sweep clock

    analogClockRAFed( { secondHand: 0 } );        // Hide second hand
    analogClockRAFed( { secondHand: 'toggle' } ); // toggle second hand back on
    analogClockRAFed( { run: 0 } );               // Turn off animation, reset times
    analogClockRAFed( { render: true } );         // Show reset clock
    analogClockRAFed( { dial: 'immediate' } );    // Show only the dial

`rAFAnimate` ties the animate function and its control options to an engine
instance.

Calling it initially stores the callback, sets up the internal timing related
variables to initialize time when the animation first runs, and defines all the
possible options that control the animation along with their initial values.

It then returns a function that is used to control the animation, including
starting and stopping it and changing any of the control options.

##### animate argument
`animate` is a callback.  It is the animation function that is controlled using
the engine instance returned by this function.

##### options argument
`options` is an object whose properties control the various option various
portions of the animation. Every possible option except one of the default
options must be included explicitly to declare the set of control options
for the engine instance returned.  Options cannot be added later.

##### How options work
Depending on the value of options initially set (or changed by calling the
engine function with the options to change as an argument), the engine will
determine if the `animate` function needs to be called.

If so, `animate` will be called with an `options` argument of its own.  However,
the options passed to `animate` will normally be `true` if a given option is
to be executed or `false` if it is not to be executed.

The `animate` function normally contains a series of `if` conditions that cause
various pieces of the animation, including the render portion, to execute
depending on the `options` passed to it.

The animation can loop continuously, run for a given number of ticks, or fire
only when the engine function is explicitly called.

###### Default options
There are three default options with default initial values.  They may be passed
explicitly to set different values than the defaults:

- `msPerTick` - controls the number of milliseconds per tick.  Defaults to
`1000.0 / 60`, or 60 ticks per second (16.67 ms)
- `run` - number of ticks to run.  Defaults to `0`
- `render` - whether or not to force an initial render.  Defaults to `false`

###### About ticks
The engine is 'tick' based.  That is, all options except `render` and `msPerTick`
are set to values of number of ticks to run.  The tick time is controlled by the
msPerTick option.

Each time the engine loops, the number of ticks which have passed since the last
loop is automatically calculated based on the rAF timestamp.  If one or more
ticks has passed, every option that still has a non-zero number of ticks left
to run has the number of ticks passed subtracted from it, and the tick count
is passed to the `animate` function callback in its `options` argument.

###### About `render`
The `render` option is special.  It can only be `true` or `false`.  It is not
normally passed in explicitly, but calculated.

If any option other than `run` or `msPerTick` is non-zero, `render` will
automatically be set to `true` in the
