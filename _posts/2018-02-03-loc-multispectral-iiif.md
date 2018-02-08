---
layout: single
title:  "LOC Multispectral IIIF"
date:   2018-01-12 12:27:22 +0000
categories: iiif
---

{% include openseadragon.html image="http://localhost:8182/iiif/2/BlockBooks_MultispectralImaging%2FBlockBooks_HSI_FullSpectrumColor_Images%2FBB-AoM_595_00_PSC.jp2" %}


To extract bands:

```
gdal_translate -b 0 BB-ASJ_570_2l_00_PSC.tif band0.tiff
```

View separate layers:

http://localhost:8182/iiif/2/BlockBooks_MultispectralImaging%2FBlockBooks_HSI_Flattened%2FBB-AoM_595_00%2FBB-AoM_595_00_018_F.tif/full/512,/0/default.jpg

Notes:

 * Full tiffs too slow
 * Static tiles no longer works in Mirador doesn't respect info.json e.g:
   * thumbnail requests
   * layers
 * Full jpgs also too slow
 * going to try jpeg2000 with  open jpeg

Docker with open jpeg:

docker run -p 8182:8182 --name imageapi -v /tmp/full_jpg:/var/lib/cantaloupe/images/ --rm pulibrary/cantaloupe:latest

using openjpeg:
 * converting to tiff doesn't work due to colour space
 * converting from jpg isn't supported
 * works to convert jpg to png to jp2 but not good results.
