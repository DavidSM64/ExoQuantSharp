# ExoQuantSharp (ExoQuant v0.7)

ExoQuantSharp is a C# port of Dennis Ranke's ExoQuant library: https://github.com/exoticorn/exoquant

ExoQuant is a high-quality, easy to use color quantization library. This is for you if you need one or more of the following:

* Very high-quality color reduction
* Reduction of images including alpha
* Creation of a shared palette for more than one image (or mipmap level)
* Dithering of the reduced image with very little noise

Other versions:\
Javascript - https://github.com/DavidSM64/ExoQuantJS \
VB.NET - https://github.com/DavidSM64/ExoQuantVB

## Usage:

First, of course, you need to import the library:

    using ExoQuantSharp;

Then for each image or texture to convert follow the following steps:

### Step 1: Initialise and set options.

First you need to create an Exoquant object:

    ExoQuant exq = new ExoQuant();

Then you can set the following options:

#### Option: Alpha is no transparency

Use this option if you don't use the alpha channel of your image/texture as transparency or if the color is already premultiplied by the alpha. To set this option just call the method `NoTransparency()`:

    exq.NoTransparency();

### Step 2: Feed the image data

Now you need to feed the image data to the quantizer. The image data needs to be 32 bits per pixel. The first byte of each pixel needs to be the red channel, the last byte needs to be alpha.

To feed the image data you have to call `Feed`, which can be called more than once to create a shared palette for more than one image, for example for a texture with several mipmap levels:

    exq.Feed(imageData); // 'imageData' is byte[]

### Step 3: Color reduction

    exq.Quantize(numColors);
    exq.QuantizeHq(numColors); // High Quality, recommended option.
    exq.QuantizeEx(numColors, highQuality); // 'highQuality' is a bool

### Step 4: Retrieve the palette

    exq.GetPalette(out byte[] rgba32Palette, numColors);

### Step 5: Map the image to the palette

    exq.MapImage(numPixels, imageData, out byte[] indexData);
    exq.MapImageOrdered(width, height, imageData, out byte[] indexData);

## Converting images to N64 CI format

The main reason I use this library is to reduce the number of colors of an image to use as a color-index (CI) texture for Nintendo 64 games. 

4-bit CI4 has a limit of 16 RGBA5551 colors and 8-bit CI8 has a limit of 256 RGBA5551 colors.

    /**
     * width = Width of the Texture.
     * height = Height of the Texture.
     * ciBitDepth = Should either be 4 for CI4, or 8 for CI8
     * numOfColors = Size of the color palette.
     * rgba32Data = input RGBA32 data
     * out ciData = output CI data
     * out rgba16Palette = output palette for CI data
     */
    void ConvertRGBA32ToCI(int width, int height, int ciBitDepth, int numOfColors, byte[] rgba32Data, 
      out byte[] ciData, out byte[] rgba16Palette)
    {
        if (ciBitDepth != 4 && ciBitDepth != 8)
            throw new Exception("Invalid CI type: CI" + ciBitDepth);

        if (ciBitDepth == 8) // CI8
        {
            if (numOfColors > 256)
                numOfColors = 256;
        }
        else // CI4
        {
            if (numOfColors > 16)
                numOfColors = 16;
        }

        // Use ExoQuant to reduce the number of colors.
        ExoQuant exq = new ExoQuant();
        exq.Feed(rgba32Data);
        exq.QuantizeHq(numOfColors);
        exq.GetPalette(out byte[] rgba32Palette, numOfColors);
        exq.MapImageOrdered(width, height, rgba32Data, out byte[] ci8Data);
        
        rgba16Palette = new byte[numOfColors * 2];

        // Convert RGBA32 palette to a RGBA16 palette
        for (int i = 0; i < numOfColors; i++)
        {
            byte red = (byte)((rgba32Palette[i * 4 + 0] / 8) & 0x1F);
            byte green = (byte)((rgba32Palette[i * 4 + 1] / 8) & 0x1F);
            byte blue = (byte)((rgba32Palette[i * 4 + 2] / 8) & 0x1F);
            byte alpha = (byte)(rgba32Palette[i * 4 + 3] > 0 ? 1 : 0); // 1 bit alpha

            rgba16Palette[i * 2 + 0] = (byte)((red << 3) | (green >> 2));
            rgba16Palette[i * 2 + 1] = (byte)(((green & 3) << 6) | (blue << 1) | alpha);
        }

        if (ciBitDepth == 4)
        {
            // Convert CI8 image to CI4 image
            ciData = new byte[rgba32Data.Length / 8];
            for (int i = 0; i < ciData.Length; i++)
            {
                ciData[i] = (byte)((ci8Data[i * 2 + 0] << 4) | ci8Data[i * 2 + 1]);
            }
        }
        else
            ciData = ci8Data;
    }

## Licence

ExoQuantSharp (ExoQuant v0.7)

Copyright (c) 2019 David Benepe\
Copyright (c) 2004 Dennis Ranke

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
