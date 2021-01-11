# Tutorial for SFTDLIB

## Introduction

SFTDLIB is a Simple and Fast Text Drawing library for the Nintendo 3DS (using sf2dlib and ctrulib) It is then easy usable form the sf2dlib, which is a simple abstraction layer over citro3d with extra functions over citro3d to make displaying simple shapes and 2d sprites easy.

## Usage
### Preparation
You need clone and build the library:
1. Clone the library
2. In the directory sfillib/libsfil/ run:
    ```
    make
    make install
    ```

### LIBS in make file
* You need to add the LIBS variable the ```-lsftd -lsf2d -lcitro3d```.
* If  you get LD errors check the dependencies also added in the rigth order (for example: ```LIBS	:= -lsftd -lsf2d -lcitro3d -lctru -lm``` )

### Adding to include
* You should add it to the include section in your code: 
```c
#include <3ds.h>
#include <sf2d.h>
#include <sftd.h>
```

## Examining the sample file

Sample 3dsx available from [here](https://github.com/vargaviktor/sf2dlib/tree/master/sample) for test runs.

The code first starts with the standard includes, and also includes the font converted to .h with BIN2S.

```c
#include <stdio.h>
#include <string.h>
#include <math.h>
#include <3ds.h>
#include <sf2d.h>
#include <sftd.h>
#include "FreeSans_ttf.h"
```
In the main at first the sf2d initialization is done:
```c
int main()
{
	sf2d_init();
	sf2d_set_clear_color(RGBA8(0x40, 0x40, 0x40, 0xFF));
	sf2d_set_vblank_wait(0);
```
Then the SFTDLIB magic happens, after the SFTDLIB initialization:
```c
	// Font loading
	sftd_init();
	sftd_font *font = sftd_load_font_mem(FreeSans_ttf, FreeSans_ttf_size);
```
This loads the font files from the buffer into a texture, which can be displayed by the sf2d:\n

To load a file, you can respectly used the follwoing functions, where the filename is the font filename with path.

```c
	sftd_font *sftd_load_font_file(const char *filename);
```
The rest of the example demonstraes the funcitons.
The keyboard is scanned, and exits, it START pressed. (after freeing the resources)

```c
	const char *someText = "Font drawing on the top screen! Text wraps after 300 pixels... Lorem ipsum dolor sit amet, consetetur sadipscing elit.";
	int textWidth = 0;
	int textHeight = 0;

	while (aptMainLoop()) {

		hidScanInput();
		if (hidKeysDown() & KEY_START) break;

		sf2d_start_frame(GFX_TOP, GFX_LEFT);
```
In the first frame, on the top screen ```sftd_draw_textf``` draws a formated text to the 10,10 x,y position with the set color, and with 20 font size.

After that the ```sftd_calc_bounding_box``` calculates the text with and heigth for a 20 size font and with text wrap at 300. 		
Then the ```sftd_draw_text_wrap``` writes that long text to 10,40 with font size 20 and wrap at 300.

```c
			sftd_draw_textf(font, 10, 10, RGBA8(0, 255, 0, 255), 20, "FPS %f", sf2d_get_fps());
			sftd_calc_bounding_box(&textWidth, &textHeight, font, 20, 300, someText);
			sf2d_draw_rectangle(10, 40, textWidth, textHeight, RGBA8(0, 100, 0, 255));
			sftd_draw_text_wrap(font, 10, 40,  RGBA8(255, 255, 255, 255), 20, 300, someText);

		sf2d_end_frame();
```
The second screen then draws the same text in lot fo sizes and colors.
```c
		sf2d_start_frame(GFX_BOTTOM, GFX_LEFT);

			sftd_draw_text(font, 10, 10,  RGBA8(0,   255, 0,   255), 20, "Font drawing on the bot. screen!");
			sftd_draw_text(font, 10, 30,  RGBA8(255, 0,   0,   255), 10, "Font drawing on the bot. screen!");
			sftd_draw_text(font, 10, 50,  RGBA8(0,   255, 0,   255), 15, "Font drawing on the bot. screen!");
			sftd_draw_text(font, 10, 70,  RGBA8(0,   0,   255, 255), 5, "Font drawing on the bot. screen!");
			sftd_draw_text(font, 10, 90,  RGBA8(255, 255, 0,   255), 25, "Font drawing on the bot. screen!");
			sftd_draw_text(font, 10, 120, RGBA8(255, 0,   255, 255), 20, "Font drawing on the bot. screen!");
			sftd_draw_text(font, 10, 140, RGBA8(0,   255, 255, 255), 12, "Font drawing on the bot. screen!");
			sftd_draw_text(font, 10, 150,  RGBA8(255, 0,   0,   255), 28, "Font drawing on the bot. screen!");

		sf2d_end_frame();

		sf2d_swapbuffers();
	}

	sftd_free_font(font);
	sftd_fini();

	sf2d_fini();
	return 0;
}
```
The original documentation has missing complete list of its functions, you can look for details into the sftd.h.

A brief list of functions an parameters:

### Draw text functions
These functions draw wide text or normal text; formated or unformated:
* where w is before text (wtext) it outputs wide text
* where f is after text (textf) it outputs formated print of the text
* where wrap is after text or textf it ouputs warapped text, line wraps after the lineWidth number of characters. 
```c
	sftd_draw_text(sftd_font *font, int x, int y, unsigned int color, unsigned int size, const char *text);
	sftd_draw_textf(sftd_font *font, int x, int y, unsigned int color, unsigned int size, const char *text, ...);
	sftd_draw_wtext(sftd_font *font, int x, int y, unsigned int color, unsigned int size, const wchar_t *text);
	sftd_draw_wtextf(sftd_font *font, int x, int y, unsigned int color, unsigned int size, const wchar_t *text, ...);


	sftd_draw_text_wrap(sftd_font *font, int x, int y, unsigned int color, unsigned int size, unsigned int lineWidth, const char *text);
	sftd_draw_textf_wrap(sftd_font *font, int x, int y, unsigned int color, unsigned int size, unsigned int lineWidth, const char *text, ...);
```

### Information functions
These functions are returning size data about a text:
The ```get_(w)text_width``` function gives back the with of a (wide) text.

The ```sftd_calc_bounding_box``` calculates the size of a box, for a wrapped text, where the ```int *boundingWidth, int *boundingHeight``` will store the results, and the ```lineWidth``` is the number of characters before linewrap.

```c
sftd_get_text_width(sftd_font *font, unsigned int size, const char *text);
sftd_get_wtext_width(sftd_font *font, unsigned int size, const wchar_t *text);

sftd_calc_bounding_box(int *boundingWidth, int *boundingHeight, sftd_font *font, unsigned int size, unsigned int lineWidth, const char *text);
``` 




