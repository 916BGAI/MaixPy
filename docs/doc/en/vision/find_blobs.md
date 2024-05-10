---
title: MaixPy Find Blobs
update:
  - date: 2024-04-03
    author: neucrack
    version: 1.0.0
    content: Initial documentation
  - date: 2024-04-03
    author: lxowalle
    version: 1.0.1
    content: Added detailed usage for finding blobs
---
Before reading this article, make sure you know how to develop with MaixCAM. For details, please read [Quick Start](../README.md).

## Introduction

This article will introduce how to use MaixPy to find color blobs and how to use the default application of MaixCam to find color blobs.

In vision applications, finding color blobs is a very common requirement, such as robots finding color blobs, automated production lines finding color blobs, etc., which requires identifying specific color areas in the image and obtaining information such as the position and size of these areas.

## Using MaixPy to Find Blobs

The `maix.image.Image` module in MaixPy provides the `find_blobs` method, which can conveniently find color blobs.

### How to Find Blobs

A simple example to find color blobs and draw bounding boxes:

```python
from maix import image, camera, display

cam = camera.Camera(320, 240)
disp = display.Display()

# Select the corresponding configuration based on the color of the blob
thresholds = [[0, 80, 40, 80, 10, 80]]      # red
# thresholds = [[0, 80, -120, -10, 0, 30]]    # green
# thresholds = [[0, 80, 30, 100, -120, -60]]  # blue

while 1:
    img = cam.read()
    blobs = img.find_blobs(thresholds, pixels_threshold=500)
    for blob in blobs:
        img.draw_rect(blob[0], blob[1], blob[2], blob[3], image.COLOR_GREEN)
    disp.show(img)
```

Steps:

1. Import the image, camera, and display modules

   ```python
   from maix import image, camera, display
   ```

2. Initialize the camera and display

   ```python
   cam = camera.Camera(320, 240)	# Initialize the camera with an output resolution of 320x240 in RGB format
   disp = display.Display()
   ```

3. Get the image from the camera and display it

   ```python
   while 1:
       img = cam.read()
       disp.show(img)
   ```

4. Call the `find_blobs` method to find color blobs in the camera image and draw them on the screen

   ```python
   blobs = img.find_blobs(thresholds, pixels_threshold=500)
   for blob in blobs:
       img.draw_rect(blob[0], blob[1], blob[2], blob[3], image.COLOR_GREEN)
   ```

   - `img` is the camera image obtained through `cam.read()`. When initialized with `cam = camera.Camera(320, 240)`, the `img` object is an RGB image with a resolution of 320x240.
   - `img.find_blobs` is used to find color blobs. `thresholds` is a list of color thresholds, where each element is a color threshold. Multiple thresholds can be passed in to find multiple colors simultaneously. Each color threshold is in the format `[L_MIN, L_MAX, A_MIN, A_MAX, B_MIN, B_MAX]`, where `L`, `A`, and `B` are the three channels in the LAB color space. The `L` channel represents brightness, the `A` channel represents the red-green component, and the `B` channel represents the blue-yellow component. `pixels_threshold` is a pixel count threshold used to filter out unwanted small blobs.
   - `img.draw_rect` is used to draw bounding boxes around the color blobs. `blob[0]`, `blob[1]`, `blob[1]`, and `blob[1]` represent the x-coordinate of the top-left corner of the blob, the y-coordinate of the top-left corner of the blob, the width of the blob, and the height of the blob, respectively.

### Common Parameter Explanations

Here are explanations of commonly used parameters. If you cannot find parameters that can implement your application, you may need to consider using other algorithms or extending the required functionality based on the current algorithm's results.

| Parameter        | Description                                                  | Example                                                      |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| thresholds       | Thresholds based on the LAB color space, thresholds=[[l_min, l_max, a_min, a_max, b_min, b_max]], representing:<br/>Brightness range [l_min, l_max]<br/>Green to red component range [a_min, a_max]<br/>Blue to yellow component range [b_min, b_max]<br/>Multiple thresholds can be set simultaneously | Set two thresholds to detect red and green<br/>```img.find_blobs(thresholds=[[0, 80, 40, 80, 10, 80], [0, 80, -120, -10, 0, 30]])```<br/>Red threshold is [0, 80, 40, 80, 10, 80]<br/>Green threshold is [0, 80, -120, -10, 0, 30] |
| invert           | Enable threshold inversion, when enabled, the passed thresholds are inverted. Default is False. | Enable threshold inversion<br/>```img.find_blobs(invert=True)``` |
| roi              | Set the rectangular region for the algorithm to compute, roi=[x, y, w, h], where x and y represent the coordinates of the top-left corner of the rectangle, and w and h represent the width and height of the rectangle, respectively. The default is the entire image. | Compute the region at (50, 50) with a width and height of 100<br/>```img.find_blobs(roi=[50, 50, 100, 100])``` |
| area_threshold   | Filter out blobs with a pixel area smaller than area_threshold, in units of pixels. The default is 10. This parameter can be used to filter out some useless small blobs. | Filter out blobs with an area smaller than 1000<br/>```img.find_blobs(area_threshold=1000)``` |
| pixels_threshold | Filter out blobs with fewer valid pixels than pixels_threshold. The default is 10. This parameter can be used to filter out some useless small blobs. | Filter out blobs with fewer than 1000 valid pixels<br/>```img.find_blobs(pixels_threshold=1000)``` |

This article introduces commonly used methods. For more APIs, please see the [image](../../../api/maix/image.md) section of the API documentation.

## Using the Find Blobs App

To quickly verify the find blobs functionality, you can first use the find blobs application provided by MaixCam to experience the effect of finding color blobs.

### Usage
Open the device, select the `Find Blobs` app, then select the color to be recognized from the bottom options or customize a color, and you can recognize the corresponding color. At the same time, the serial port will also output the recognized coordinates and color information.

<video src="/static/video/find_blobs.mp4" controls="controls" width="100%" height="auto"></video>

### Detailed Explanation

The app interface is as follows:

![](../../../static/image/find_blobs_app.jpg)

#### Using Default Configuration

The find blobs app provides four default configurations: `red`, `green`, `blue`, and `user`. `red`, `green`, and `blue` are used to `find red, green, and blue color blobs`, respectively, while `user` is mainly provided for `user-defined color blob finding`. The method for customizing configurations is described below. For a quick experience, you can switch to the corresponding configuration by `clicking` the `buttons` at the bottom of the interface.

#### Finding Custom Color Blobs

The app provides two ways to find custom color blobs: using adaptive LAB thresholds and manually setting LAB thresholds.

##### 1. Finding Color Blobs with Adaptive LAB Thresholds

Steps:

1. `Click` the `options icon` in the bottom-left corner to enter configuration mode.
2. Point the `camera` at the `object` you need to `find`, `click` on the `target object` on the screen, and the `left side` will display a `rectangular frame` of the object's color and show the LAB values of that color.
3. Click on the appearing `rectangular frame`, and the system will `automatically set` the LAB thresholds. At this point, the image will outline the edges of the object.

##### 2. Manually Setting LAB Thresholds to Find Color Blobs

Manual setting allows for more precise targeting of the desired color blobs.

Steps:

1. `Click` the `options icon` in the bottom-left corner to enter configuration mode.
2. Point the `camera` at the `object` you need to `find`, `click` on the `target object` on the screen, and the `left side` will display a `rectangular frame` of the object's color and show the `LAB values` of that color.
3. Click on the bottom options `L Min`, `L Max`, `A Min`, `A Max`, `B Min`, `B Max`. After clicking, a slider will appear on the right side to set the value for that option. These values correspond to the minimum and maximum values of the L, A, and B channels in the LAB color format, respectively.
4. Referring to the `LAB values` of the object color calculated in step 2, adjust `L Min`, `L Max`, `A Min`, `A Max`, `B Min`, `B Max` to appropriate values to identify the corresponding color blobs. For example, if `LAB = (20, 50, 80)`, since `L=20`, to accommodate a certain range, set `L Min=10` and `L Max=30`. Similarly, since `A=50`, set `A Min=40` and `A Max=60`. Since `B=80`, set `B Min=70` and `B Max=90`.

#### Getting Detection Data via Serial Protocol

The find blobs app supports reporting information about detected color blobs via the serial port (default baud rate is 115200).

Since only one report message is sent, we can illustrate the content of the report message with an example.

For instance, if the report message is:

```
shellCopy code

AA CA AC BB 14 00 00 00 E1 08 EE 00 37 00 15 01 F7 FF 4E 01 19 00 27 01 5A 00 A7 20
```

- `AA CA AC BB`: Protocol header, content is fixed
- `14 00 00 00`: Data length, the total length excluding the protocol header and data length
- `E1`: Flag, used to identify the serial message flag
- `08`: Command type, for the find blobs app application, this value is fixed at 0x08
- `EE 00 37 00 15 01 F7 FF 4E 01 19 00 27 01 5A 00`: Coordinates of the four vertices of the found color blob, with each value represented by 2 bytes in little-endian format. `EE 00` and `37 00` represent the first vertex coordinate as (238, 55), `15 01` and `F7 FF` represent the second vertex coordinate as (277, -9), `4E 01` and `19 00` represent the third vertex coordinate as (334, 25), `27 01` and `5A 00` represent the fourth vertex coordinate as (295, 90).
- `A7 20`: CRC checksum value, used to verify if the frame data has errors during transmission.

## About the LAB Color Space

The LAB color space, like the RGB color space, is a way to represent colors. LAB can represent all colors visible to the human eye. If you need to learn more about LAB, you can search for relevant articles online, which will provide more details. However, for you, it should be sufficient to understand why LAB is advantageous for MaixPy.

Advantages of LAB for MaixPy:

1. The color gamut of the LAB color space is larger than that of RGB, so it can completely replace RGB.
2. In the LAB color space, since the L channel is the brightness channel, we often set it to a relatively large range (commonly [0, 80]), and when coding, we mainly focus on the A and B channels. This can save a lot of time spent struggling with how to select color thresholds.
3. The color perception in the LAB color space is more uniform and easier to debug with code. For example, if you only need to find red color blobs, you can fix the values of the L and B channels and only adjust the value of the A channel (in cases where high color accuracy is not required). For RGB channels, you generally need to adjust all three R, G, and B channels simultaneously to find suitable thresholds.

