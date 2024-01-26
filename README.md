 # PTZControl

## Table of Contents
1. [History](#history)
2. [Basic Code](#basic-code)
3. [Used Interfaces](#used-interfaces)
4. [Used Environment and Libraries](#used-environment-and-libraries)
5. [Behavior](#behavior)
6. [Supported Cameras](#supported-cameras)
7. [Guard Thread](#guard-thread)
8. [Logitech Motion Control](#logitech-motion-control)
9. [Standard Motion Control](#standard-motion-control)
10. [Hotkeys](#hotkeys)
11. [Command Line Options](#command-line-options)
12. [Registry Settings](#registry-settings)

## History
This small program is designed to control a Logitech Rally Camera. It quickly turned out that the operation with the remote control is possible but cumbersome and inaccurate.

## Basic Code
Finding code that shows how to control a PTZ camera was not easy, but after contacting Logitech, they provided a download link to the `Logitech Collaboration Software Reference Kit` (Logitech CSRK) which contained some Lync remote control code that gave some hints and usage for the Logitech camera interface.

## Used Interfaces
The program directly uses the camera media control via the Windows SDK, and for controlling the zoom functions and the preset functions of the PTZ 2 Pro, Logitech provided example code with defines from the Lync driver. Logitech internal interfaces are used for saving/getting presets, and for step-by-step Pan Tilt camera control. Standard controls from the Windows Driver API are used for zoom control and timer-controlled PT (Pan Tilt) control.

## Used Environment and Libraries
The program was developed using Visual Studio 2019 Community Edition with MFC and ATL as libraries. The EXE runs alone, without installing any other files or DLLs or any installation.

## Behavior
The program is always in the foreground, compact, and small for easy use over OBS programs. The current selected preset or home position is shown with a green background on the buttons. Recalling a preset is simply done by clicking on one of the number buttons, and presets are changed by pressing the M button followed by a number key.

The program remembers its last position on the screen and all settings are stored in the registry under `HKEY_CURRENT_USER\SOFTWARE\MRi-Software\PTZControl`.

## Supported Cameras
Currently, the Logitech PTZ 2 Pro, PTZ Pro, Logitech Rally cameras and ConferenceCam CC3000e Camera are automatically detected. For other cameras, you can try to force detection by specifying the name (or part of the name) of the cameras in the registry or on the command line.

Internally, all cameras that have one of the following tokens in the name are automatically used:

* PTZ Pro
* Logi Rally
* ConferenceCam

## Guard Thread
 Unfortunately, we've encountered issues where OBS or the USB bus hangs with a camera, causing the PTZControl program to become unresponsive due to blocked camera control commands. To address this, an internal guard thread has been implemented in the application, which can detect when it is no longer functioning correctly and terminate automatically. This feature prevents the need to use the task manager to forcefully close the application during a livestream, which can be time-consuming and stressful.

## Logitech Motion Control
Logitech cameras have their own interface for pan/tilt control, which moves the camera in the X and Y axes by a specified step value. This is a special feature offered by Logitech. When you click on a direction button once, a single step pulse is output. If you hold down a direction button, a pulse is sent to the camera repeatedly at a certain interval to change the direction.

## Standard Motion Control
 This control is the default when you first start the application. I've found that the Logitech motion control can be somewhat rough. However, it is also possible to control pan and tilt using motor commands for the X and Y directions through normal device control. This is done by turning the motor on for a specific time interval and then off again. Accordingly, you can adjust the timer interval for Motor on/off as needed. The default value is 70msec, but values between 70 and 100 or good values can be used. If you click on a direction button once, the motor will turn on and off again after the corresponding interval. If the direction button is kept pressed, the motor will remain switched on for the corresponding direction until the button is released. This control seems more effective and accurate to me, which is why it is the default setting. However, the disadvantage is that if the timer interval is too small, the camera may not react immediately when a button is clicked. Nevertheless, I use this setting with a 70msec timer because precision was more important to me, as our camera is installed relatively far away from the podium.

## Hotkeys
The program has several hotkeys that allow for control without the mouse when it has focus.
- Pan-Tilt control with Left, Right, Up, Down keys.
- Home position with Num-0, Home keys.
- Memory function with the M-key.
- Recall stored position with the numeric keys 1-8 or the numeric key pad keys Num-1 to Num-8.
- Open the setings dialog with Num-Divide or Num-Multiply
- Zoom in/out Page-Up/Down, Num+Plus, Num-Minus
- Select Camera 1: Alt+1, Alt+Num-1, Alt+Page-Up
- Select Camera 2: Alt+2, Alt+Num-2, Alt+Page-Down

## Command Line Options
- `-device:"name of device"`: specify a name component of a camera to be used for control. If you enter "\*" as the name, then any camera will be recognized.
- `-showdevices`: displays a message box after startup showing the name(s) of the detected cameras.
- `-noreset`: at startup, a detected camera is moved to the home position and the zoom is reset to maximum wide angle. If this option is specified, the camera position remains unchanged.
- `-noguard`: prevents the application from terminating itself in a controlled manner. This can be especially important in the event of a bug and for testing.

## Registry Settings
In the registry branch `HKEY_CURRENT_USER\SOFTWARE\MRi-Software\PTZControl\Options`, it is possible to preset the following options without using the command line.

**NoReset (DWORD value)**
*Value <>0:* Has the same function as -noreset on the command line. The current camera and zoom position is maintained when starting the program. Value = 0: When starting the program, you move to the home position and zoom to maximum wide angle. (Default)

**NoGuard (DWORD value)**
*Value <>0:* The guard thread that may automatically terminate the application is terminated.
*Value = 0:* The guard thread automatically terminates the application if a blocking of the USB bus is detected. (Default)
