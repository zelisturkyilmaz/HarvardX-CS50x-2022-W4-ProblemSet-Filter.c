# CS50's Introduction to Computer Science
## HarvardX - CS50x
### Week 5 Problem Set - Filter
<hr>


### Assignment and Requirements:
Write helper.c file that defines and applies filters to BMPs, per the below. *Other files in the project are provided by CS50 team*


```
$ ./filter -r IMAGE.bmp REFLECTED.bmp
```
where ```IMAGE.bmp``` is the name of an image file, ```-r``` is the filter mode and ```REFLECTED.bmp``` is the name given to an output image file, now reflected.

### Filters:

**Grayscale**

Grayscale filter takes an image and convert it to black-and-white. As long as the red, green, and blue values are all equal, the result will be varying shades of gray along the black-white spectrum, with higher values meaning lighter shades (closer to white) and lower values meaning darker shades (closer to black).

If we apply that to each pixel in the image, the result will be an image converted to grayscale.

```C
// Convert image to grayscale
void grayscale(int height, int width, RGBTRIPLE image[height][width])
{
    float gray_value;

    for (int row = 0; row < height; row++)
    {
        for (int col = 0; col < width; col++)
        {
            gray_value = round((image[row][col].rgbtBlue + image[row][col].rgbtGreen + image[row][col].rgbtRed) / 3.00);

            image[row][col].rgbtBlue = gray_value;
            image[row][col].rgbtGreen = gray_value;
            image[row][col].rgbtRed = gray_value;
        }
    }
    return;
}
```

where RGBTRIPLE is a struct defined as:

```C
typedef struct
{
    BYTE  rgbtBlue;
    BYTE  rgbtGreen;
    BYTE  rgbtRed;
} __attribute__((__packed__))
RGBTRIPLE;
```

**Reflection**

By reflecting an image, pixels on the left side of the image should end up on the right, and vice versa.

```C
// Reflect image horizontally
void reflect(int height, int width, RGBTRIPLE image[height][width])
{
    for (int row = 0; row < height; row++)
    {
        for (int col = 0; col < width / 2; col++)
        {
            RGBTRIPLE temp = image[row][col];
            image[row][col] = image[row][width - (col + 1)];
            image[row][width - (col + 1)] = temp;
        }
    }
    return;
}
```

**Blur**

For this problem “box blur” method will be used, which works by taking each pixel and, for each color value, giving it a new value by averaging the color values of neighboring pixels.

```C
// Blur image
void blur(int height, int width, RGBTRIPLE image[height][width])
{
    RGBTRIPLE copy[height][width];
    for (int i = 0; i < height; i++)
    {
        for (int j = 0; j < width; j++)
        {
            copy[i][j] = image[i][j];
        }
    }
    for (int i = 0; i < height; i++)
    {
        for (int j = 0; j < width; j++)
        {
            float sumOfRed = 0;
            float sumOfGreen = 0;
            float sumOfBlue = 0;
            int count = 0;

            for (int x = -1; x < 2; x++)
            {
                for (int y = -1; y < 2; y++)
                {
                    if (i + x < 0 || i + x >= height)
                    {
                        continue;
                    }
                    if (j + y < 0 || j + y >= width)
                    {
                        continue;
                    }
                    sumOfRed += copy[i + x][j + y].rgbtRed;
                    sumOfGreen += copy[i + x][j + y].rgbtGreen;
                    sumOfBlue += copy[i + x][j + y].rgbtBlue;
                    count++;
                }
            }

            image[i][j].rgbtRed = round(sumOfRed / count);
            image[i][j].rgbtGreen = round(sumOfGreen / count);
            image[i][j].rgbtBlue = round(sumOfBlue / count);
        }
    }

    return;
}
```

**Edges**

Detect edges in an image, lines in the image that create a boundary between one object and another, by applying [the Sobel operator](https://en.wikipedia.org/wiki/Sobel_operator) to the image.

```C
// Detect edges
void edges(int height, int width, RGBTRIPLE image[height][width])
{
    RGBTRIPLE copy[height][width];
    for (int i = 0; i < height; i++)
    {
        for (int j = 0; j < width; j++)
        {
            copy[i][j] = image[i][j];
        }
    }

    int Gx[3][3] = {{-1, 0, 1}, {-2, 0, 2}, {-1, 0, 1}};
    int Gy[3][3] = {{-1, -2, -1}, {0, 0, 0}, {1, 2, 1}};

    for (int i = 0; i < height; i++)
    {
        for (int j = 0; j < width; j++)
        {
            float Gx_red;
            float Gx_blue;
            float Gx_green;
            float Gy_red;
            float Gy_blue;
            float Gy_green;
            Gx_red = Gx_blue = Gx_green = Gy_red = Gy_blue = Gy_green = 0;


            for (int k = -1; k < 2; k++)
            {
                for (int l = -1; l < 2; l++)
                {
                    if (i + k < 0 || i + k >= height)
                    {
                        continue;
                    }
                    if (j + l < 0 || j + l >= width)
                    {
                        continue;
                    }
                    Gx_red += copy[i + k][j + l].rgbtRed * Gx[k + 1][l + 1];
                    Gx_green += copy[i + k][j + l].rgbtGreen * Gx[k + 1][l + 1];
                    Gx_blue += copy[i + k][j + l].rgbtBlue * Gx[k + 1][l + 1];
                    Gy_red += copy[i + k][j + l].rgbtRed * Gy[k + 1][l + 1];
                    Gy_green += copy[i + k][j + l].rgbtGreen * Gy[k + 1][l + 1];
                    Gy_blue += copy[i + k][j + l].rgbtBlue * Gy[k + 1][l + 1];
                }
            }
            int red = round(sqrt(Gx_red * Gx_red + Gy_red * Gy_red));
            int green = round(sqrt(Gx_green * Gx_green + Gy_green * Gy_green));
            int blue = round(sqrt(Gx_blue * Gx_blue + Gy_blue * Gy_blue));
            if (red > 255)
            {
                red = 255;
            }
            if (green > 255)
            {
                green = 255;
            }
            if (blue > 255)
            {
                blue = 255;
            }
            image[i][j].rgbtRed = red;
            image[i][j].rgbtGreen = green;
            image[i][j].rgbtBlue = blue;
        }
    }
    return;
}
```

#### Compiling And Execution:

Before execution of the program, it must be compiled with a compiler, translating it from source code into machine code.\
Execute the command below in the Command Line to do that:

```C
make filter
```

Execute the program by executing the below:
```C
./filter -filtermode images/yard.bmp out.bmp
```
which takes the image at ```images/yard.bmp```, and generates a new image called ```out.bmp``` after running the pixels through ```-filtermode``` function
