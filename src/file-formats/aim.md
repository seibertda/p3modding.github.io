# AIM

AIM is a proprietary archive file format for images and textures used in many games developed by Ascaron.

The filename extension used by AIM files is `.aim`.

Despite sharing the same file extension, there are two incompatible versions of AIM files.
They can be distinguished by their respective `magic numbers`:

```bash
DLL      | Magic number                  | ASCII      | Games
---------+-------------------------------+------------+---------------------------
AIM.dll  | 41 49 4D 00                   | AIM        | Patrician 3, Port Royale
AIM20.dll| 41 49 4D 52 45 53 32 2E 30 30 | AIMRES2.00 | Port Royale, Port Royale 2
```

AIM files can be compressed and include a header containing metadata, followed by one or more data sections.
These data sections may hold raw pixel data, uncompressed or compressed image formats, as well as tiled and mipmapped textures.

## AIM.dll
Patrician 3 uses only the first version of the AIM format via `AIM.dll`, which provides an interface for creating and manipulating AIM files.
All AIM files of Patrician 3 are compressed. The compression algorithm needs to be reverse-engineered.

### Data structures
The enumeration `AIM_EXPORT_FORMAT` defines the supported output formats for the `AIM_SAVE_BITMAP` function:
```C
typedef enum {
	BMP_8       = 1,       //  8-bit BMP
	BMP_24      = 2,       // 24-bit BMP
	TGA_MAP_8   = 3,       //  8-bit TGA
	TGA_RGBA_16 = 4,       // 16-bit TGA
	TGA_RGB_24  = 5,       // 24-bit TGA
	TGA_RGBA_32 = 6,       // 32-bit TGA
	JPG_LOW     = 100,     // low quality JPG
	JPG_MID     = 150,
	JPG_HIGH    = 200      // high quality JPG
} AIM_EXPORT_FORMAT;
```

The enumeration `AIM_PIXEL_ENCODING` defines the supported pixel encodings of a AIM file:
```C
typedef enum {
	COMPACT = 0,
	PALETTE = 1,
	UNKNOWN = 2,
	NOALPHA = 3,
	SIMPLE  = 4,
} AIM_PIXEL_ENCODING;
```

The following data structure, `AIM_IMAGE`, is required to call the exported functions of `AIM.dll`:
```C
typedef struct {
    uint8_t*           buf_ptr;            // pointer to image data
    uint32_t*          palette_ptr;        // pointer to color palette, if palette is used
    uint32_t           width1;             // image width
    uint32_t           height;             // image height
    AIM_PIXEL_ENCODING pixel_encoding;     
    uint32_t           bytes_per_pixel2;
    uint32_t           width2;
    uint32_t           field_1c;
    uint32_t           field_20;
    uint32_t           field_24;
    uint32_t           field_28;
    uint32_t           field_2c;
    uint32_t           field_30;
    uint32_t           field_34;
    uint32_t           field_38;
    uint32_t           field_3c;
    uint32_t           field_40;
    uint32_t           field_44;
    uint32_t           field_48;
    uint32_t           field_4c;
    uint32_t           field_50;
    uint32_t           field_54;
    uint32_t*          inner_ptr;
} AIM_IMAGE;
```

### Exported functions
The following function signatures have been verified through testing:
```C
// initializes an AIM_IMAGE instance
void AIM_INIT(AIM_IMAGE* image);

// loads the specified '.aim' image and decompresses it
int AIM_CONVERT_FILE(AIM_IMAGE* image, const char* input_file);

// exports the loaded '.aim' image into the specified file format
int AIM_SAVE_BITMAP(AIM_IMAGE* image, AIM_EXPORT_FORMAT format, const char* output_file);

// frees the AIM_IMAGE instance
void AIM_FREE(AIM_IMAGE* image);
```

### Implementations
There is currently no independent implementation that does not rely on `AIM.dll` and fully supports reading and writing of `.aim` files.

[aimcli](https://github.com/P3Modding/p3-lib/tree/master/aimcli) supports `.aim` to `.bmp` conversion with `PALETTE`, `NOALPHA` and `SIMPLE` pixel encodings.
