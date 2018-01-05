16.4.1 Assignment: Sepia Filter

In this assignment, we will create a program to perform a sepia filter on an image. A sepia filter makes an image with vivid colors look like an old, aged photograph. Most graphical editors include a sepia filter.

The filter itself is not hard to code. It recalculates the red, green, and blue components of each pixel based on the old values of red, green, and blue. Mathematically, if we think about a pixel as a three- dimensional vector, the transformation is nothing but a multiplication of a vector by matrix.

Let the new pixel value be \(B G R\)T\(whereTsuperscript stands for transposition\).B,G,andRstand for blue, green, and red levels. In vector form the transformation can be described as follows:

pic here pic here

In scalar form, we can rewrite it as

pic here pic here

In the assignment given in section 13.10 we coded a program to rotate the image. If you thought out its architecture well, it will be easy to reuse most of its code.

We will have to use saturation arithmetic. It means, that all operations such as addition and multiplication are limited to a fixed range between a minimum and maximum value. Our typical machine arithmetic is modular: if the result is greater than the maximal value, we will come from the different side of the range. For example, forunsigned char: 200 + 100 = 300 mod 256 = 44. Saturation arithmetic implies that for the same range between 0 and 255 included 200 + 100 = 255 since it is the maximal value in range.

C does not implement such arithmetic, so we will have to check for overflows manually. SSE contains instructions that convert floating point values to single byte integers with saturation.

Performing the transformation in C is easy. It demands direct encoding of the matrix to vector multiplication and taking saturation into account. Listing16-32shows the code.

Listing 16-32.image\_sepia\_c\_example.c

```
#include <inttypes.h>
struct pixel { uint8_t b, g, r; };

struct image {
    uint32_t width, height;
    struct pixel* array;
};

static unsigned char sat( uint64_t x) {
    if (x < 256) return x; return 255;
}
static void sepia_one( struct pixel* const pixel ) {
    static const float c[3][3] =  {
    { .393f, .769f, .189f },
    { .349f, .686f, .168f },
    { .272f, .543f, .131f } };
struct pixel const old = *pixel;

pixel->r = sat(
        old.r * c[0][0] + old.g * c[0][1] + old.b * c[0][2]
        );
pixel->g = sat(
        old.r * c[1][0] + old.g * c[1][1] + old.b  * c[1][2]
        );
pixel->b = sat(
        old.r * c[2][0] + old.g * c[2][1] + old.b * c[2][2]
        );
}

void sepia_c_inplace( struct image* img ) {
    uint32_t x,y;
    for( y = 0; y < img->height; y++ )
        for( x = 0; x < img->width; x++ )
            sepia_one( pixel_of( *img, x, y ) );
}
```

Note that usinguint8\_torunsigned charis very important.

In this assignment you have to

•Implement in a separate file a routine to apply a filter to a big part of image \(except for the last pixels maybe\). It will operate on chunks of multiple pixels at a time using SSE instructions.

The last few pixels that did not fill the last chunk can be processed one by one using the C code provided

in Listing16-32.

* Make sure that both C and assembly versions produce similar results.

* Compile two programs; the first should use a naive C approach and the second one should use SSE instructions.

* Compare the time of execution of C and SSE using a huge image as an input \(preferably hundreds of megabytes\).

* Repeat the comparison multiple times and calculate the average values for SSE and C.

To make a noticeable difference, we have to have as many operations in parallel as we can. Each pixel consists of 3 bytes; after converting its components into floats it will occupy 12 bytes. Eachxmmregister is 16 bytes wide. If we want to be effective we will have use the last 4 bytes as well. To achieve that we use a frame of 48 bytes, which corresponds to threexmmregisters, to 12-pixel components, and to 4 pixels.

Let the subscript denote the index of a pixel. The image then looks as follows:

pic here

We would like to compute the first four components. Three of them correspond to the first pixel, the fourth one corresponds to the second one.

To perform necessary transformations it is useful to first put the following values into the registers:

pic here

We will storethematrix coefficients in eitherxmmregisters or memory, but it is important to store thecolumns, not the rows.

To demonstrate the algorithm, we will use the following start values:

pic hee

We usemulpsto multiply these packed values withxmm0...xmm2.

pic here

The next step is to add them usingaddpsinstructions.  
 The similar actions should be performed with two other two 16-byte-wide parts of the frame, containing

pic here

This technique using transposed coefficients matrix allows us to cope without horizontal addition instructions such ashaddps. It is described in detail in \[19\].

To measure time, usegetrusage\(RUSAGE\_SELF, &r\)\(readman getrusagepages first\). It fills a structrof typestruct rusagewhose fieldr.ru\_utimecontains a field of typestruct timeval. It contains, in turn, a pair of values for seconds spent and millise conds spent. By comparing these values before transformation and after it we can deduce the time spent on transformation.

Listing16-33shows an example of single time measurement.

Listing 16-33.execution\_time.c

```
#include <sys/time.h>
#include <sys/resource.h>
#include <stdio.h>
#include <unistd.h>
#include <stdint.h>

int main(void) {
    struct rusage r;
    struct timeval start;
    struct timeval end;

    getrusage(RUSAGE_SELF, &r );
    start = r.ru_utime;

    for( uint64_t i = 0; i < 100000000; i++ );

    getrusage(RUSAGE_SELF, &r );
    end = r.ru_utime;

    long res = ((end.tv_sec - start.tv_sec) * 1000000L) +
        end.tv_usec - start.tv_usec;

    printf( "Time elapsed in microseconds: %ld\n", res );
    return 0;
}
```

Use a table to perform a fast conversion fromunsigned charintofloat.

float const byte\_to\_float\[\] = {0.0f, 1.0f, 2.0f, ..., 255.0f };

---

■Question 335 read about methods of calculating the confidence interval and calculate the 95% confidence interval for a reasonably high number of measurements.

---



