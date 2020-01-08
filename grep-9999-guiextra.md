# GREP [XXXX] -- Upstreaming Enhanced GUI Controls

- Original Author: Michael Piscopo <ghostop14@gmail.com>
- Champion: Michael Piscopo <ghostop14@gmail.com>
- Status: Draft

History:
- 07-Jan-2020: Initial Draft

## Abstract

While GNU Radio comes with a great selection of GUI controls, there are a few Qt widgets 
available to modern GUI designers that GNU Radio doesn't have exposed through the UI. In 
addition, there are some controls that are used in common SDR receiver software such as 
a digital number control, Azimuth/Elevation displays, and visual LEDs that would greatly 
enhance the UI experience.  gr-guiextra is an OOT module that provides these additional 
controls, along with a port of the Fast Autocorrelator control in gr-baz that is only 
supported in GNU Radio 3.7.  This GREP is to suggest upstreaming the additional UI controls 
into the GNU Radio gr-qtgui core.

## Copyright / License

CC-BY-ND

## Motivation

In working with satellite communications and antenna array projects, it was necessary to 
create some new controls.  While those controls were being created it seemed like an 
opportunity to quickly create some useful solutions I had long-wanted in my GNU Radio 
flowgraphs. That gave birth to gr-guiextra.  After testing and operationally using them 
I'd like to make them available not only as an OOT but as part of the core GNU Radio 
package.

## Description

**Compass** - Provides an on-screen compass with 3 different choices for the needle (full, half, and -pi to pi with DoA uncertainty)

**Dial** - Think of this as your volume dial control.  Supports both variables and messages.

**Digital Number Control (Frequency Control)** - Very similar to SDR-based stand-alone applications, clicking the numbers can adjust frequency, or it can be set in read-only mode for display only.  Supports both controlling a variable and input / output messages.

**LED Indicator** - Just as it sounds.  On-screen LED.  You can choose the on/off colors and supports both variables and messages to control state.

**Linear Gauge (Progress Bar)** - Linear progress bar / linear gauge, either horizontal or vertical.  Colors can be chosen for the background and the bar, and it is controllable via variable or message.

**Dial Gauge** - Circular guage.  Colors can be chosen for the background and the bar, and it is controllable via variable or message.

**Toggle Button** - Push to hold down.  Push again to release.  Both variables and messages available.

**Toggle Button** - Modern toggle switch.  Both variables and messages available.

**Message-based Checkbox** - Like a standard checkbox, but both variables and messages available.

**Message Pushbutton** - Produce a message when the button is pushed.  (This is different than the toggle in that this bounces back to release like a traditional button when pressed and a message is only produced on press).

**Graphics Item** - Drop a graphic anywhere in your GNU Radio app screen.  You can also control the file based on input messages to change the graphic on the fly.  This can be tied to the toggle switch, button, etc. to control the image displayed based on other factors for a more dynamic display.  This control also supports the concept of overlays where additional images can be dynamically add/update/deleted from the main graphic which also allows for dynamic animation via message.  Overlay messages should be in the car portion of the message and can be a dictionary with the following keys: filename (full path), x, y, and an optional scalefactor.  A list of dictionaries can also be sent.  Overlays are keyed by filename so passing updates to x/y for an overlay will update it.  Setting any overlay file to x/y -1,-1 will remove it.

**App Background** - While stylesheets can be used to change an app, that can be cumbersome if all you want is to change the background color or display a background graphic.  This drop-in control lets you do either or both.

**Azimuth-Elevation Plot** - Similar to the Az/El plot in satellite trackers, this widget produces a similar screen that can be fed from gpredict-doppler's rotor block that outputs an az_el message.  This UI block is looking for a dictionary in the car part of a message with 'az' and 'el' float entries.

**Distance "Radar"** - Similar to the old-school signal trackers, this widget produces a radar-like screen and draws a "contact" circle at a distance of your choosing controlled by an inbound message.

**Fast Auto-Correlator GUI Sink** - A QT GUI version of the Fast Auto-correlator provided under gr-baz as a WX control

**Fast Auto-Correlator** - A fast auto-correlator block that provides correlated vectors as output that can either be used for additional processing, writing to files, or other UI controls.

