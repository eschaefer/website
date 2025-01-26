---
title: 'Components of JPEG compression'
tags: [digital photos, compression, til]

comments: false
---

JPEG compression begins with a general conversion of color information (R, G, B) about an image to its luminance (Y) and chrominance (Cb, Cr). This is also known as an afine transformation in color space.

`[R G B] -> [Y Cb Cr]`

Interestingly, the eye is most sensitive to the green component in this transformation. It has the highest weighted value:

```
| Y  |     |  0.299       0.587       0.114 |   | R |     | 0 |
| Cb |  =  |- 0.1687    - 0.3313      0.5   | * | G |   + |128|
| Cr |     |  0.5       - 0.4187    - 0.0813|   | B |     |128|
```

There are many more little nuggets of trivia about how JPEG compression exploits other known strengths/weaknesses of the human eye [here](http://www.opennet.ru/docs/formats/jpeg.txt).
