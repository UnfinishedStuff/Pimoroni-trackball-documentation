# Trackball-breakout documentation

Home made documentation for [Pimoroni's Trackball Breakout](https://shop.pimoroni.com/products/trackball-breakout).  It's a neat breakout, but Pimoroni don't have a huge amount of documentation at the moment.

Begin by importing the library:

`from trackball import Trackball`

Then create an instance of the TrackBall object which can be used to control the breakout:

`trackball = TrackBall(interrupt_pin=4)`

The `interrupt_pin` isn't well described, but usually these things are to allow the peripheral to tell the main board (in this case the Pi) that something has changed or is ready to be read.  I don't imagine that this can be changed if you're using one of the Breakout Garden (p)HATs, to change this from the default Pin 4 you'd have to wire it manually.

After this begin using the library's functions:

# Function reference

### set_rgbw(r,g,b,w)

The breakout has four LEDs: starting frop the top left of the breakout and moving clockwise these are red, blue, white and green.  These are set up nicely to illuminate the trackball.  The brightness of these LEDs is controlled by passing a value from 0 to 255 for each colour as a parameter to the function.  

### set_red(r)
### set_green(g)
### set_blue(b)
### set_white(w)

These are a subset of `set_rgbw(r,g,b,w)`, and simply set a single LED to a value between 0 and 255.

### read()

This function returns the values which describe the trackball itself.  It returns a tuple of five integers plus a boolean.  In order that they're returned, the integers describe the up, down, left and right velocities of the trackball plus a "1" if the ball is pressed in or a "0" if not.  The boolean describes the stwitch state with "True" if pressed or False if not.  

I'm not entirely sure how the trackball velocity is calculated, but I've never been able to get it to rise above 26 even when tring to spin it as fast as I can, so maybe assuming 30 as an upper limit is a good idea.

The difference between the integer button state and the boolean state seems to be that the integer is only `1` at the moment the button is pressed and `0` at all other times (even if held down), whereas the boolean is `True` continuously whilst the button is held down.

### change_address(new_address)

This is an interesting function: the Pi doesn't communicate directly with the trackball, there's actually a small microcontroller on the breakout which acts as a go-between.  By default the microcontroller uses i2c address `0x0A`, or `0x0B` if you cut a trace on the board.  However, you can actually use this function to tell the microcontroller to use an entirely different address.  I've been able to change it to values from 0x09 to 0x77 inclusive.

This could be a helpful way to avoid address collisions with other devices.  However, there's a catch:  the device is locked by the traces on the board to load with address `0x0A` or `0x0B` if a trace is cut.  The address can only be changed _after_ the device has already loaded, so at least one of those addresses must be free to begin with.  There's another hitch:  the `change_address` function sets the new address and then waits for the device to send a signal on the interrupt pin that it is ready. You must use the `enable_interrupt()` function to permit this behaviour _before_ changing the address.  When the Pi sends the `change_address(new_address)` command it will wait for the microcontroller to send it a signal over the interrupt pin before continuing.  After that you can delete the old `trackball` object and create a new one with the new address.

I'm not _entirely_ sure why this exists when the initial address is hard wired.  It's quite neat though.

###  enable_interrupt(interrupt=True)

This enables the microcontroller to use the interrupt pin.  You'll need to have defined an interrupt pin when the trackball instance was created.  This is important for the `change_address(new_address)` function.

### get_interrupt()

This returns a boolean which describes the state of the interrupt pin.
