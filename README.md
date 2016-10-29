# arduino-tone-library
## Description
This is an Arduino Library to produce square-wave of the specified frequency (and 50% duty cycle) on any Arduino pin. It is an updated fork of [an abandoned Google Code project](https://code.google.com/archive/p/rogue-code/wikis/ToneLibraryDocumentation.wiki).

A duration can optionally be specified, otherwise the wave continues until `stop()` is called.

The pin can be connected to a piezo buzzer or other speaker to play tones.

## Arduino Core Version

A simplified version of the Tone library has been incorporated into the Arduino core since 0018. It only provides a single tone (since only one timer is used). You can find the core documentation [here](https://www.arduino.cc/en/Reference/Tone).

## Warning

Do not connect the pin directly to some sort of audio input. The voltage is considerably higher than [standard line level voltages](https://en.wikipedia.org/wiki/Line_level), and can damage sound card inputs, etc. You could use a voltage divider to bring the voltage down, but you have been warned.

You MUST have a resistor in line with the speaker, or you WILL damage your controller.

## Hardware Connections/Requirements
 * Just connect the digital pin to a speaker (with a resistor - say 1K - in line), and the other side of the speaker to ground (GND).
 * You can use a potentiometer to control the volume. Use a 10K Ohm potentiometer (variable resistor) in line with the 1K Ohm resistor connected to the speaker.
 * Using this library will affect usage of the PWM outputs, so be aware.
 * Also, although it's the last timer to be allocated, timer 0 (which is used for millis() among other things) will be affected if used.

## Library Usage
### Instantiation/Creation
```
Tone tone1;

void setup(void) {
  tone1.begin(13);
}
```
### Methods
 * `begin(pin)` - prepares a pin for playing a tone.
 * `isPlaying()` - returns true if tone is playing, false if not.
 * `play(frequency[,duration])` - play a tone.
   * frequency is in Hertz, and the duration is in milliseconds.
   * duration is optional. If duration is not given, tone will play continuously until stop() is called.
   * `play()` is non-blocking. Once called, play() will return immediately. If duration is given, the tone will play for that amount of time, and then stop automatically.
 * `stop()` - stop playing a tone.
### Constants
[Tone.h](../blob/master/Tone.h) includes a list of available constants for notes and their frequencies in Hz. For example, `NOTE_A4`.

### Quick Example
Play a 440 Hz - musical note of 4th octave A - on pin 13:
```
include <Tone.h>

Tone tone1;

void setup() {
  tone1.begin(13);
  tone1.play(NOTE_A4);
}

void loop() { }
```

## Ugly Details
The library uses the hardware timers on the microcontroller to generate square-wave tones in the audible range.

You can output the tones on any pin (arbitrary). The number of tones that can be played simultaneously depends on the number of hardware timers (with CTC capability) available on the microcontroller.

 * ATmega8: 2 (timers 2, and 1)
 * ATmega168/328: 3 (timers 2, 1, and 0)
 * ATmega1280/2560: 6 (timers 2, 3, 4, 5, 1, 0)

The timer order given above is the order in which the timers are allocated. Timer 0 is a sensitive timer on the Arduino since it provides `millis()` and PWM functionality.

The range of frequencies that can be produced depends on the microcontroller clock frequency and the timer which is being used:

|MCU clock|8 bit timer F<sub>low</sub>|16 bit timer F<sub>low</sub>|F<sub>high</sub>|
|:------------|:------------------------------|:-------------------------------|:-------------------|
|8 MHz |16 Hz |1 Hz (1/16 Hz) |4 MHz |
|16 MHz |31 Hz |1 Hz (1/8 Hz) |8 MHz |

Although F<sub>high</sub> can go as high as 8 MHz, the human hearing range is typically as high as 20 kHz.

Tone accuracy is dependent on the timer prescalar. Frequency quantization occurs as the frequencies increase per prescalar.

If you used a 16 bit timer (e.g. timer 1, or timers 3,4,5 on a Mega), you could generate "tones" down to 1/8 Hz (one cycle every 8 seconds), although the library only accepts integers for frequency.

After all is said and done, because `play()` only accepts unsigned integers for frequency, the maximum frequency that can be produced is 65535 Hz - which, after rounding, results in a 65573.77 Hz "tone" on a 16 MHz part. Even if play accepted larger values for frequency, you couldn't achieve better than around 80KHz with the Tone library because the pin toggling is done in software. Each toggle, in software, requires AT LEAST 50+ cycles.
