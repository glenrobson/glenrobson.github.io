---
layout: single
title:  "IIIF Tribunal Image Classifier"
date:   2020-09-08 12:27:22 +0000
categories: iiif,AI
---

As [mentioned previously](/iiif,ai/2020/09/08/AI-ideas.html), I am following the [Fast AI course](https://course.fast.ai/) and as part of the second lesson you are encouraged to develop an Image Classifier. An Image Classifier is a type of program which can look at an image and decide which bucket the image should go in. In the first lesson the example Image Classifier let you know if an image was a picture of a dog or a cat and in this weeks lesson the Image Classifier distinguished between pictures of Brown, Grizzly and Teddy Bears. 

In the previous post I discussed a couple of ideas but decided to go with the Tribunal records identification as this seemed the easiest to get started with. As a reminder the [Cardiganshire War Tribunal records](https://www.library.wales/discover/digital-gallery/archives/cardiganshire-great-war-tribunal-appeals-records#?c=&m=&s=&cv=&xywh=-2068%2C-1%2C7731%2C5641) are a fascinating collection of records covering the communication between the community in Ceredigion and the Military Tribunals who decided if applicants were allowed to avoid military conscription during WW1. The records were part of a National Library of Wales (NLW) crowdsourcing project working with volunteers to both transcribe the records but also to retain a link between the transcription and the field on the form. The completed transcriptions were made available by the NLW and are available on [Paul McCann's Github Page](https://github.com/NLW-paulm/Welsh-Tribunal-annotations) as IIIF Annotations. 

The archive contains many different forms and supporting correspondence. My plan is to create a classifier which will identify the form type. This is now mostly an academic exercise as all of the pages have been identified and transcribed but if in theory someone else was to digitise a set of tribunal records they could use this tool to identify which type of documents they have.  I can test this theory using the sample tribunal record copies in [Manchester Archive](https://www.flickr.com/photos/manchesterarchiveplus/albums/72157632619308865/) and the [National Archives](https://discovery.nationalarchives.gov.uk/details/r/C14091136).

## Preparing the data

To fit the data structure discussed in the course there should be a directory per category (or bucket) containing images in that set. So there should be 9 directories containing images for the following document types:

 * Tag: Beige R186/187, page 1
 * Tag: Beige R186/187, page 2
 * Tag: Beige R41/42: Page 1
 * Tag: Beige R41/42: Page 2
 * Tag: Blue R52/53, page 1
 * Tag: Blue R52/53, page 2
 * Tag: Pink R43/R44: Page 1
 * Tag: Pink R43/R44: Page 2
 * Tag: Unknown document type

The dataset from Paul, contains Annotation Lists for each of the different districts that make up the county of Cardiganshire (now called Ceredigion). The districts are:

 * Aberaeron Borough District
 * Aberaeron Rural District
 * Aberystwyth Borough District
 * Aberystwyth Rural District
 * Cardigan Borough District 
 * Cardigan Rural District
 * Lampeter Rural District
 * Lampeter Urban District
 * Llandysyl rural District
 * Newquay District
 * Tregaron District
 
Inside these files there are lists of annotations and there is one annotation per page which looks as follows:

```
{
    "@type": "oa:Annotation",
    "motivation": "sc:painting",
    "on": "https://damsssl.llgc.org.uk/iiif/2.0/4001851/canvas/4001854.json#xywh=0,0,3497,4413",
    "resource": {
        "@type": "cnt:ContentAsText",
        "chars": "Description: Letter from Fred Burris & Sons from Horse Shoes and Mule Shoes for war department of British Government.
                 <p>Transcription: We understand from Mr John Rees, 16 Albert St, that exemption has been refused him.   We however, 
                 suggest that there must be some error here, particularly as this man is badged No. L 16876, and consequently can only
                 be taken with the permission of the Ministry of Munitions, \nWe think we have previously explained the matter and are 
                 at a loss to understand what has happened. The only thing, therefore, is for us to communicate with the War Office and
                 inform them of the details,  \n<p>Name of Tribunal: Aberaeron Borough/Urban District<p>Number of Case: Appeal Form No 4
                 <p>Tag: Unknown document type",
        "format": "text/plain"
    }
},
```

The first job is to find the page `Tag` and link it to the image it references. The `Tag` is buried in the `resource.chars` part of the annotation and to pick it out we need to split the string and find `<p>Tag: ` and then this will be followed by the Tag. I used the following python method to do this conversion:

```
def string2json(line):
    data = {}
    for field in line.split('<p>'):
       # print (field)
        name, value = field.split(':')[0], ':'.join(field.split(':')[1:])
       # print("Name: {}, value: {}".format(name,value))
        data[name] = value.strip()

    return data 
```

This will return a hash table with all of the fields in the string separated so `data['Tag']` will return the page tag. 

The second part of the task is to identify the image this annotation points to. To start we need to find the canvas which is in the `on` field:

```
"on": "https://damsssl.llgc.org.uk/iiif/2.0/4001851/canvas/4001854.json#xywh=0,0,3497,4413",
```

If this was a set of images I didn't know then I would find the manifest, look for this canvas id and then find the reference to the IIIF Image. Luckily I can take a short cut as I know the last number (`4001854`) before the .json is actually the Image Identifier. I can then use the following to get access to the Image information:

http://dams.llgc.org.uk/iiif/2.0/image/**4001854**/info.json

One decision that was required was the size of the image to request. The course recommends 

_"We don't have a lot of data for our problem (150 pictures of each sort of bear at most), so to train our model, we'll use RandomResizedCrop with an image size of 224 px, which is fairly standard for image classification, and default aug_transform"_

(From ['Training Your Model, and Using It to Clean your Data'](https://github.com/fastai/fastbook/blob/master/02_production.ipynb))

so I can request a IIIF Image at this size by running:

http://dams.llgc.org.uk/iiif/2.0/image/4001854/full/**224,**/0/default.jpg

![Example Tribunal Image](https://damsssl.llgc.org.uk/iiif/2.0/image/4001854/full/224,/0/default.jpg)

The full code for the program to extract the `Tags` and Images from the Annotation Lists is called [typeImage.py](https://github.com/glenrobson/Welsh-Tribunal-annotations/blob/master/scripts/typeImage.py) and can be found on [GitHub](https://github.com/glenrobson/Welsh-Tribunal-annotations/blob/master/scripts/typeImage.py). Once this program has run you will have a directory for each Tag containing the following number of images:

* Beige_R186-187_page_1:      138 images
* Beige_R186-187_page_2:      137 images
* Beige_R41-42_Page_1:     1,331 images
* Beige_R41-42_Page_2:     1,350 images
* Blue_R52-53_page_1:      701 images
* Blue_R52-53_page_2:      690 images
* Pink_R43-R44_Page_1:     1,618 images
* Pink_R43-R44_Page_2:     1,609 images
* Unknown_document_type:      971 images

To access it in the next steps I needed to make it public so I zipped it up and put it here: [TribunalTypeImages.zip](https://iiif.gdmrdigital.com/ww1-tribunal/TribunalTypeImages.zip).

## Training the model

The next part of the task was to take this zip file of images and train a model that can identify the different forms. The course recommends doing this by creating a Jupiter Notebook. I haven't quite got my heard around how these notebooks fit into or compares with virtual machines, Docker or Vagrance but they allow you to both run code and mix in Markdown descriptions to comment on the process. Using the Google Colab software you can run these Jupiter notepads on their virtual machine infrastructure for free. One of the main advantages of this approach is that Google Colab provides Virtual Machines with performant graphics cards which appear essential for training models. This was mentioned in the first lesson and I had a go at running the Jupiter notebook locally but even though my machine isn't slow, it took a surprisingly long time to train the basic example models. Even though I was sorely tempted, lesson one advised against spending _(wasting)_ time building your own computer with a powerful graphics card and just use the free or low cost options they link to in the course. 

The Jupiter notebook is embedded in this blog as a read only version with the saved output when I ran it.

You should be able to open a version you can run yourself by clicking the Open in Colab button below. In the Colab version, when you see a block of code you should be able to run it by moving your mouse cursor to the top left part of the code block. This should then show a play button. You should run the code blocks in order and wait for them to finish before going onto the next one.  

<script src="https://gist.github.com/glenrobson/969a6a25b4822c7f37945bf85087b7e9.js"></script>

## Results and reflection

I was really pleased to see the results of the Image Classification. It worked on most of the images and for the ones that it didn't there seemed to be clear reasons why not. It would be interesting to look further into the top losses to see if this could be used to identify data that needs correcting. The only real downside to this project is the lack of applicability to real world problems. Its very specific to this collection but I wonder if the techniques could be applied to identify different types of objects in a collection. One thought I've had is whether you could train a classifier to identify types of images e.g. Maps, Newspapers, Manuscripts etc. This maybe something I try next although finding the source data will more challenging.  
