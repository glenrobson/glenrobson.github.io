---
layout: single
title:  "IIIF from Scratch"
date:   2018-01-12 12:27:22 +0000
categories: iiif
---

I've been trying to learn Welsh for a number of years but unfortunately I'm not a natural linguist. So when one of our neighbours gave our Children an old Welsh book to read I set about using it as a challenge and translating it. Being prone to technical procrastination rather than getting on with the translation I've had a look at how I can turn this book into a IIIF resource. I hope this will be useful for people looking to make their images available as IIIF but don't have the time or resources to invest in a comprehensive IIIF stack.

## Stage 1 - Digitisation

So the quality of the digitisation isn't going to win any prizes but I used a HP Envy 5546 combination printer and Scanner:

<figure>
	<a href="https://support.hp.com/us-en/product/hp-envy-5540-all-in-one-printer-series/5447939/model/5447940/drivers"><img src="/assets/images/HP-Envy-5546.jpg"></a>
	<figcaption><a href="https://support.hp.com/us-en/product/hp-envy-5540-all-in-one-printer-series/5447939/model/5447940/drivers" title="HP Envy 5546">HP Envy 5546</a>.</figcaption>
</figure>


The provided 'HP Easy Scan' software has a nice simple interface for scanning. For scanning a book like this you can click 'return' to capture another scan allowing you to faff with the book on the scanner trying to ensure it doesn't fall off the table. As its a graphical book I started off with the 'Photos, Graphics, Etc.' setting but this was a bit too clever for its own good and cropped the most graphical part of the page. Once switching to the 'Documents with Color' it scanned the page as expected.

I exported the scanned images (14 pages) to the following formats:

 * jp2 - I won't use this in this blog post but in future I plan to write a post about using these images with one if the more feature full image servers. See [IIIF Awesome for a list][awesome-image].
 * tiff

The images are named page001 to page014.

## Stage 2 - Making the images through IIIF

For this project I'm looking for a no server option that ideally will be hosted through this blog. The fully featured IIIF image servers require a service to be running to do the required conversions on a single source image. An alternative is to chop up the source image into tiles and store them on disk. This allows you to get all of the zoomable features of IIIF without running a server. The limitation is that you won't get all of the functionality a IIIF image server can support, particularly custom regions of an image. I'll demonstrate this limitation later.

To chop up the images I'm going to use [Simeon's tile generator][static-tiles]. Following the install instructions I do the following:

```
hostname:website gmr$ cd /tmp/
hostname:tmp gmr$ git clone git://github.com/zimeon/iiif.git
Cloning into 'iiif'...
remote: Counting objects: 5514, done.
remote: Total 5514 (delta 0), reused 0 (delta 0), pack-reused 5514
Receiving objects: 100% (5514/5514), 32.25 MiB | 1.71 MiB/s, done.
Resolving deltas: 100% (1825/1825), done.
iPhone-Glen:tmp gmr$ cd iiif/

hostname:iiif gmr$ ls
CHANGES.rst		MANIFEST.in		demo-static		iiif_static.py		pypi_upload.md		testimages
CONTRIBUTING.md		README			etc			iiif_testserver.py	run_tests.sh		testpages
INSTALLATION.md		README.rst		iiif			iiif_testserver.wsgi	run_validate.sh		tests
LICENSE.txt		demo-auth		iiif_cgi.py		iiif_testservertest.py	setup.py

hostname:iiif gmr$ sudo pip install 'Pillow<4.0.0'
Password:
The directory '/Users/gmr/Library/Caches/pip/http' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
The directory '/Users/gmr/Library/Caches/pip' or its parent directory is not owned by the current user and caching wheels has been disabled. check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
Collecting Pillow<4.0.0
  Downloading Pillow-3.4.2-cp27-cp27m-macosx_10_6_intel.macosx_10_9_intel.macosx_10_9_x86_64.macosx_10_10_intel.macosx_10_10_x86_64.whl (3.4MB)
    100% |████████████████████████████████| 3.5MB 372kB/s
Installing collected packages: Pillow
Successfully installed Pillow-3.4.2

hostname:iiif gmr$
```

The next stage is to feed in the tiff images and give an output directory for the IIIF tiles. Trying the first page:

```
hostname:iiif gmr$ mkdir -p ../welsh-book/page001
hostname:iiif gmr$ ./iiif_static.py --write-html ../welsh-book/page001 -d ../welsh-book/page001 --api-version=2.1 ~/Documents/images/welsh_book/tiff/page001.tif
iiif_static.py: source file: /Users/gmr/Documents/images/welsh_book/tiff/page001.tif
iiif.static: ../welsh-book/page001 / page001/0,0,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/0,512,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/0,1024,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/0,1536,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/0,2048,512,286/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/512,0,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/512,512,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/512,1024,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/512,1536,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/512,2048,512,286/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1024,0,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1024,512,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1024,1024,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1024,1536,512,512/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1024,2048,512,286/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1536,0,160,512/160,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1536,512,160,512/160,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1536,1024,160,512/160,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1536,1536,160,512/160,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1536,2048,160,286/160,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/0,0,1024,1024/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/0,1024,1024,1024/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/0,2048,1024,286/512,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1024,0,672,1024/336,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1024,1024,672,1024/336,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/1024,2048,672,286/336,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/0,0,1696,2048/424,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/0,2048,1696,286/424,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/212,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/212,292 -> page001/full/212,
iiif.static: ../welsh-book/page001 / page001/full/106,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/106,146 -> page001/full/106,
iiif.static: ../welsh-book/page001 / page001/full/53,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/53,73 -> page001/full/53,
iiif.static: ../welsh-book/page001 / page001/full/27,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/27,36 -> page001/full/27,
iiif.static: ../welsh-book/page001 / page001/full/13,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/13,18 -> page001/full/13,
iiif.static: ../welsh-book/page001 / page001/full/7,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/7,9 -> page001/full/7,
iiif.static: ../welsh-book/page001 / page001/full/3,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/3,5 -> page001/full/3,
iiif.static: ../welsh-book/page001 / page001/full/2,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/2,2 -> page001/full/2,
iiif.static: ../welsh-book/page001 / page001/full/1,/0/default.jpg
iiif.static: ../welsh-book/page001 / page001/full/1,1 -> page001/full/1,
iiif.static: ../welsh-book/page001 / page001/info.json
iiif.static: Writing HTML to ../welsh-book/page001
iiif.static: ../welsh-book/page001 / page001.html
```

I then had a look at the webpage `page001.html` to see if it had worked. Unfortunately I noticed that it couldn't access the OpenSeadragon script in `/tmp/welsh-book/page001/openseadragon200/openseadragon.min.js` so I symlinked the OpenSeadragon javascript file:

```
hostname:iiif gmr$ ln -s /tmp/iiif/iiif/third_party/openseadragon200 ../welsh-book/page001/openseadragon200
```

and hit refresh and got a working zoomable image:

![Working IIIF Tiles shown in OpenSeaDragon](/assets/images/iiif-tiles-page001.png)

To process all of the images I can run the following:

```
./iiif_static.py -d ../welsh-book --api-version=2.1 ~/Documents/images/welsh_book/tiff/*.tif
```

and this generates 14 directories in `/tmp/welsh_book`. I've then copied the 14 directories into [this github project][iiif-welshbook] which is the code behind this blog. Now I've moved the images I have to update the `@id` in all of the info.json files so that it points to the correct place. Note if you forget to do this then OpenSeaDragon won't be able to open the image.

I've written a program [updateImageId.py][updateImage] that can be run as follows:

```
./standalone-iiif/updateImageId.py json_file 'https://glenrobson.github.io/iiif/welsh_book/
```

and this should prepend the correct prefix to make it work on this site. To run this on allow of the images I run:

```
find ../gdmr-digital/ -name "info.json" -exec ./standalone-iiif/updateImageId.py {} 'https://glenrobson.github.io/iiif/welsh_book/' \;
```

Note if you make a mistake then have a look at the updateImageId.py README to see how you can replace a prefix.

It should now be possible to view the static image using Openseadragon:

{% include openseadragon.html image="/iiif/welsh_book/page002" %}


[awesome-image]: https://github.com/IIIF/awesome-iiif#image-servers
[static-tiles]: https://github.com/zimeon/iiif/tree/master/demo-static
[iiif-welshbook]: https://github.com/glenrobson/glenrobson.github.io/tree/serverless_iiif/iiif/welsh_book
[updateImage]: https://github.com/glenrobson/iiif_stuff/blob/master/standalone-iiif/updateImageId.py
