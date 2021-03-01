# uasyncfadingled
### Async Micropython driver for fading LED(s) at human-adapted brightness levels (for RP2 port).

A Micropython library to asynchronously drive one or more fading-
capable LED(s) at relative brightness levels adapted for (logarithmic)
human brightness perception. Provides a number of convenience methods
for dimming and fading that can be called synchronously, e.g. in a
callback handler. Additionally, almost any fading sequence can be
constructed and scheduled with simple instructions.

#### Usage
*Download the driver*  
It's really just a single small file (after all, this is for **Micro**python),
so you can just download it here and put it next to your own program file(s).

*Initialize the driver:*

    from uasyncfadingled import LED
    
    # create a driver on a single GPIO pin, e.g. GPIO 4:
    led = LED(4)
    # or drive several LED at different GPIO pins simultaneously,e.g. at GPIO 4,5,6:
    led = LED([4, 5, 6])

*Commands:*  
All brightness settings operate on a scale adapted for the logarithmic
nature of human brightness perception with values from 0-100%.
This has the side-effect that very low values (<10%) might not provide
enough current to the LED(s) to actually light them up.

    # self explanatory methods:
    led.turn_on()
    led.turn_off()
    led.set_to(brightness_level)
    
    # convenience method for stepping the brightness up/down
    led.change_brightness(percent_step)
    
    # the fading methods
    led.fade_to(target_brightness, [optional args])
    led.start_wave(brightness_range, wave_length)
    led.start_heartbeat(brightness_range, pulse)
    led.start_sequence(sequence, [optional args])
    led.abort_fading()

You can construct many different fading sequences with the method `start_sequence`. Usage
is in the docstring, and you can see in the code that `start_wave` and `start_heartbeat`
actually use this generalized method as well, so they can serve as examples.

*What's asynchronous about this?*
The actual commands are synchronous (and relatively cheap to execute, i.e. they can be
used in callback handlers. Behind the scenes, they schedule asynchronous update routines
that can be integrated into your main program's event loop. This means of course that
they require a started event loop into which the asynchronous updates can be scheduled.

*A simple example program:*

    import uasyncio
    from machine import Pin
    from uasyncfadingled import LED

    led = LED(4)

    def led_heartbeat_pattern():
      '''Synchronous callback handler'''
      led.start_wave(brightness_range=(20,100), wave_length=4_000)

    async def main():
      '''Asynchronous co-routine in your code'''
      # e.g. watching for GPIO 22 to change to high
      Pin(22).irq(trigger=Pin.IRQ_RISING, handler=led_heartbeat_pattern)
      while True:
        await uasyncio.sleep_ms(10)

    # provide the event loop, e.g.:
    uasyncio.run(main())

Have fun!
