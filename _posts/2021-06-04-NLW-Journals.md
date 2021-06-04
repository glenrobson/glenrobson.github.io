---
layout: single
title:  "NLW Journals Writeup"
date:   2021-06-04 12:27:22 +0000
categories: iiif
---
[NLW Journals](https://journals.library.wales) write up from 2017. I often find that I link to this article but the original isn't always available so I have copied it here for posterity. The original is available on the [NLW Dev Site](https://dev.llgc.org.uk/wiki/index.php?title=IIIF_Journals).

# Welsh Journals Technical Writeup 
Authors: 
 * Glen Robson - Head of Systems, 
 * Dan Field - Head of Development, 
 * Dylan Jones - Senior Web Developer, 
 * Kim Botticelli - Senior Web Developer

This document describes how the National Library of Wales (NLW) has built the new Journals website https://journals.library.wales. The new website brings together two collections of Welsh Journals and also provides access to content that hasn’t previously been available. The amount of content in the new website includes:

* Total titles 479
* Total new titles 395 
* Total number of pages 1,226,816
* Total number of new pages 669,223

The development was done on top of the NLW IIIF infrastructure including harvesting using Sitemaps and indexing the IIIF Manifests and Annotations lists. 

## Background
The NLW has worked on two Journal projects. The first project was funded by JISC digitising 50 modern Welsh Journals. The second project was funded by the Welsh Government and the European Regional Development fund to digitise over one million pages of Journal content. The content of the Journals ranges from academic and scientific publications to literary and popular magazines. Further details on the collection can be seen at:

[https://www.llgc.org.uk/index.php?id=7594](https://www.llgc.org.uk/index.php?id=7594)

The JISC-funded modern Welsh Journals project was undertaken around 2008 and included content that was under copyright so Authors and Publications had to be identified and rights status established. As part of the digitisation workflow articles were identified by noting the article start and end page. The project website was launched around 2009/10. 

The second project learned a number of lessons from the previous project particularly in what metadata was captured and the workflow involved in making the items available. For this project Journals were chosen which were out of copyright and article metadata wasn’t captured as part of the digitisation process. 36 titles were shared with Europeana and were made available on a temporary website while development was undertaken to merge the two datasets. The project included digitising 425 titles and around 1 million pages. 

## IIIF interface to the data 
The Journal pages were scanned to TIFF files and a number of derivatives were created to support the previous website including a 650px image, 50% image and a PFF zoomable image. The OCR was stored as ALTO and also in AbbyyFineReader xml.  For the set of Journals that were shared with Europeana the access file was JPEG2000 and the NLW had undertaken a migration to replace the previous images with a single JPEG 2000. After delivery of the Europeana project the NLW developed a PFF to IIIF image server so converting the images not covered by this project was no longer necessary. Now both JP2 and PFF images appear in the website as IIIF images and the differing source formats is no longer an issue. Using the IIIF image standard has also made the 650px and 50% images redundant.

The metadata for the Journals is stored in the NLW fedora repository as METS documents and these were converted to IIIF Manifests and Collections as can be seen in the diagram below. The IIIF standard promotes using a seeAlso link to point to structured metadata and we converted our MODS metadata to EDM. The list of Journals which are part of the project are listed in a sitemap and this is what we used for indexing. 

![Technical infrastructure diagram](http://dev.llgc.org.uk/iiif/IIIF_Journals_Technical_ARC.jpg)

### Sitemap 

An example of the sitemap we used is below and includes live links to IIIF collections for Public Domain Titles:

```
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"  xmlns:iiif="http://iiif.io/api/presentation/2/">
    <url>
        <iiif:manifest>http://dams.llgc.org.uk/iiif/2.0/2000001/manifest.json</iiif:manifest>
    </url>
    <url>
        <iiif:manifest">http://dams.llgc.org.uk/iiif/2.0/2035242/manifest.json</iiif:manifest>
    </url>
    ….
    <url>
        <iiif:manifest>http://dams.llgc.org.uk/iiif/2.0/2007313/manifest.json</iiif:manifest>
    </url>
    <url>
        <iiif:manifest>http://dams.llgc.org.uk/iiif/2.0/2007768/manifest.json</iiif:manifest>
    </url>
</urlset> 
```

There wasn’t time to add last modification dates into the sitemap but this would have been very useful to enable the indexer to pickup when a manifest changed. This process of re-indexing was managed through an issue tracking system but became complicated to manage at times.

### Europeana Data Model (EDM) Metadata 

An example EDM record for a title can be seen below:

[http://dams.llgc.org.uk/behaviour/llgc-id:2000001/fedora-sdef:rdf/toRDF](http://dams.llgc.org.uk/behaviour/llgc-id:2000001/fedora-sdef:rdf/toRDF)

Issue level manifests also contain links to EDM but it was only the title level that was indexed for this project. The EDM record provided the following information in the SOLR record:

* Publication title - dc:title
* Publisher details - dc:publisher (Welsh and English)
* Language of Journal - dc:language
* Description of Journal - dc:description (Welsh and English)

Frequency of publication was added later to the EDM.

### Articles

The original Welsh Journals project contained Article information and this has been encoded using IIIF Ranges and an example can be seen at:

[http://dams.llgc.org.uk/iiif/2.0/1097087/manifest.json](http://dams.llgc.org.uk/iiif/2.0/1097087/manifest.json)

The ranges contain an article title, author and type although only the title is shown through the Universal Viewer. The Universal Viewer displays these ranges in the index panel and this manifest can be seen in isolation at:

[https://viewer.library.wales/1097087](https://viewer.library.wales/1097087) 

When viewing this manifest in the Journals website the Universal Viewer shows the current issue within its context of running Journals issues:

[https://journals.library.wales/view/1093205/1097087](https://journals.library.wales/view/1093205/1097087)

In the index section you can view the issues either by label (from the Journal Title IIIF Collection) or by navDate. 

### Missing Pages 
During the processing of the second Journals project the NLW developed a tool which would allow the re-arrangement of images and also the adding of missing pages where the original pages weren’t available for digitisation. These images are noted in the METS document and in the generated IIIF manifests these appear as canvases without a IIIF Image service. See the following as an example which is a slightly extreme example picked up during testing. We believe that there was only one image (page 16) available for this issue hence the first 16 pages are marked as missing. The more common case is where a single image is missing in a sequence:

[https://viewer.library.wales/2886549](https://viewer.library.wales/2886549) ([http://dams.llgc.org.uk/iiif/2.0/2886549/manifest.json](http://dams.llgc.org.uk/iiif/2.0/2886549/manifest.json))

### Broken Images
During the validation of images we undertook during development we found 6 images which required attention. Some of these could be regenerated from the original tiff file but some images will require further work after the project has completed. One jp2 in particular passes jhove validation and jpylyzer analysis but still doesn’t work using IIP image and will have to be reported. To create a valid manifest we treated these errors similar to above and marked them as missing. See page 35 of the following:

[https://viewer.library.wales/2161205](https://viewer.library.wales/2161205) ([http://dams.llgc.org.uk/iiif/2.0/2161205/manifest.json](http://dams.llgc.org.uk/iiif/2.0/2161205/manifest.json))

### Rights protected Images
In the original Welsh Journals project there are a number of pages which are protected due to copyright reasons. Some images have been blanked but others are protected by the Fedora rights management process. These images present a 401 when requested. We looked into how this would work in the Universal Viewer and if it was a use case for the IIIF Authentication API. The following manifest responds to 401s in two different ways:

[https://viewer.library.wales/1272050](https://viewer.library.wales/1272050) ([http://dams.llgc.org.uk/iiif/2.0/1272050/manifest.json](http://dams.llgc.org.uk/iiif/2.0/1272050/manifest.json))

Pages 162 and 163 return a 401 if the info.json is requested with a customer error message in Welsh and English. If you request these images using CURL you get the following response:

```
HTTP/1.1 401 Nid yw'r hawliau gennym i arddangos y deunydd hwn. - We do not have the rights to display this material.
```

This works in the Universal Viewer as it displays the message in a information box. The possibly more correct way of doing this is in page 166 to 177 where the info.json displays a rights service with a description saying the content is unavailable. Unfortunately currently this isn’t working in the UV and more experimentation is needed to see what needs to be in the info.json to make this use case work. The experimental info.json for page 166 is:

[https://damsssl.llgc.org.uk/iiif/2.0/image/1272222/info.json](https://damsssl.llgc.org.uk/iiif/2.0/image/1272222/info.json)

### Validation
One of the things we found most useful during the development of the website was to have a validation script that would go through the sitemap and validate all of the IIIF collections and manifests and identify missing fields that were considered ‘mandatory’ for the project.  The validator for this project checked the following:

* All collections and manifests were valid JSON and existed in the cache
* All Journal titles and Manifests had valid navDates - there were a handful of issues which didn’t have dates and even after investigation dates couldn’t be assigned. 
* All Journal titles had descriptions and frequency information (not all missing frequencies could be fixed by the end of the project) in the EDM
* All Issues had a search api pointer
* All images responded to at least an info.json request

This validator was run regularly and on a daily basis towards the end of the project. The navDates were created by converting textual string dates e.g. 'Nov 1852' or 'Spring 1924' into ISO dates and so the validation picked out problems with this conversion. It also picked out mistakes in the data like 30th Feb 2001.

## Indexing 
In order to deliver site content in a timely manner, we have indexed the IIIF manifests, EDM and Sitemap to a Solr search index. This allows our front-end web developers to converse with a simple HTTP search API and receive JSON content for inclusion in the website. 

We developed an indexing script which firstly iterated over the Sitemap in order to collect all of the top-level manifest documents for the project. Each manifest was then processed and a page-level document was indexed to solr containing the full text contained on the journal page along with associated metadata for faceting e.g Issue Date, Publication Title, page Order within issue (used to land on correct page within the Universal Viewer). We also indexed a single ‘publication’ document which was used for the publication information page.

Page-level documents needed to store the location of each word on the page in pixel co-ordinates. This is stored as ALTO XML but converted on the fly to use IIIF Annotations. Storing the annotations as separate records in SOLR would have meant a lot of redundant information being stored and it wouldn’t be possible to make use of SOLRs ranking algorithms which can show results containing multiple word matches on a page higher. We devised a cut down version of an annotation in JSON for storing the string and coordinates and stored this as a string within solr. This could be extracted in the IIIF search API implementation in order to highlight search terms on the page within the Universal Viewer. 

Each word would be marked up as [x,y,width,height, word] e.g. ["671","178","21","14","cymdeithas"] and each word was stored in an array of all words on the page, which made up the Json object. 

We considered a number of alternatives to compressing the annotations into SOLR including:

* Reading the Annotation List at query time 
We were concerned this might lead to performance problems. We currently have 100 results per page and this could lead to 100 annotation list requests per user per search. It would also take time for search implementation to processes the JSON and find the annotations for that result.
* Storing the annotations in a separate annotation store
This would also lead to similar amount of requests to the above and we would have to develop a similar method of matching the initial query with annotations. We were also concerned about keeping another system alongside SOLR up to date.
* Indexing the annotation as annotations (like the SimpleAnnotationServer does)
This would produce a SOLR instance that was large and contained repeated information for example over 1 million copies of the same motivation. We would also have to work on developing something that would match searches for phrases as these would cross annotations assuming we encoded them at word level. If we encoded them at another level e.g. line or paragraph the highlight box in the Universal Viewer would be bigger than it needs to be.  
 
An example Solr Page document is included below with comments after the -- symbol. 

```
 "Doctype":"page",
 "page_coords":"[[\"580\",\"299\",\"163\",\"84\",\"YR\"],[\"773\",\"295\",\"31\",\"84\",\"\"],[\"834\",\"295\",\"861\",\"88\",\"HYFFORDDWR.\"],[\"347\",\"524\",\"104\",\"42\",\"Cyp.\"],[\"473\",\"524\",\"111\",\"54\",\"III.]\"],[\"1000\",\"520\",\"129\",\"48\",\"MAI,\"]]", -- encoded page annotations
 "page_text":" YR  HYFFORDDWR. Cyp. III.] MAI", -- the above text without coordinate information
 "Id":"2005214", -- cutdown page identifier 
 "Publicationpid":"llgc-id:2004553", -- cut down identifier of title
 "publicationtitle":"Yr hyfforddwr", -- title from EDM dc:title
 "publicationtitle_facet":"Yr hyfforddwr", -- SOLR processed copy of above for faceting
 "publisherdetails_en":"Hughes and Co.", -- from EDM dc:publisher[@xml:lang=en]
 "publisherdetails_cy":"Hughes and Co.", -- from EDM dc:publisher[@xml:lang=en]
 "Publicationlanguage":["cy"], -- from EDM dc:language
 "publicationdetails_en":"The monthly Welsh language religious periodical of the Campbellite Baptists in Wales. The periodical's main contents were religious articles and poetry. The periodical was edited by the poet and man of letters, John Edwards (Meiriadog, 1813-1906). Associated titles: Yr Hyfforddiadyr (1855); Y Llusern (1858).", -- from EDM dc:description[@xml:lang=en]
 "publicationdetails_cy":"Cylchgrawn crefyddol, Cymraeg ei iaith, y Bedyddwyr Campbellaidd yng Nghymru. Prif gynnwys y cylchgrawn oedd erthyglau crefyddol a barddoniaeth. Golygwyd y cylchgrawn gan y bardd a llenor, John Edwards (Meiriadog, 1813-1906). Teitlau cysylltiol: Yr Hyfforddiadyr (1855); Y Llusern (1858).", -- from EDM dc:description[@xml:lang=cy]
 "Issuepid":"llgc-id:2005213", -- cut down issue identifier
 "issuelabel":"Cyf. III rhif. 5 - Mai 1854", -- from Issue manifest label
 "Issueorder":31, -- Important for linking into the Universal Viewer index of issue within the Title (from IIIF Collection)
 "isodate":"1854-05-01T00:00:00Z", -- From Issue Manifest
 "Issuedate_decade":1850, -- Created from Nav date for facets and front page graphic. 
 "Issuedate_year":1854, -- Created from Nav date for facet
 "Issuedate_month":5, -- Created from Nav date for facet
 "Issuedate_day":1, -- Created from Nav date for facet
 "Issuedate_weekday":1, -- Created from Nav date for facet
 "issuerightsurl":"https://creativecommons.org/publicdomain/mark/1.0/", -- from license field in IIIF manifest
 "Pagepid":"llgc-id:2005214", -- cutdown Page identifier
 "Pagelabel":"[65]", -- from Issue manifest, canvas label
 "Pageorder":2, -- Important for linking into the Universal Viewer, index of canvas in sequence from issue manifest.
 "Project":"scif", -- internal project identifier. If it has rangers this would be ccymod. 
 "timestamp":"2017-04-07T12:15:48.112Z"} -- indexing timestamp to for debugging purposes. 
```

Due to the iterative development process found in an Agile project like this, we needed to implement caching on the indexing server to reduce stress on the IIIF server. Certain content like the Annotation Lists was unlikely to change between iterations so a gziped cache of the JSON content was stored locally with the indexer. A MD5 sum of the URL was used to name the file on the filesystem so it could be easily matched for future cache requests. Gzipping kept the file sizes down and the entire cache of the project is only 12GB in size. 

In order to access the metadata, as IIIF does not promote the storage of structured metadata within the manifest, we instead opted to provide access to the metadata in an EDM file linked from the manifests. This enabled us to use many existing vocabularies such as Dublin Core, RDF, DCTerms to further mark up the data.

## Website 
The website is written in PHP and is built on top of SOLR. The majority of the searching and the website front page is generated directly from queries into SOLR. The IIIF manifests and collections come in when you start viewing an item using the Universal Viewer. The Universal Viewer reads the IIIF Collection and Manifests to produce an index showing all of the issues of a Journal and Articles if present. The metadata on the right hand panel is generated from the current issue which is shown in the central image panel. 

### Supporting IIIF search API 
We have developed an implementation of the IIIF search API which uses the same SOLR instance as the website and allows searching within the Universal Viewer. It is also used when navigating between the results page into an item. This was a complicated development due to the way we had decided to compress the annotations in SOLR.

One of the issues is it is very difficult to predict the number of results per IIIF search results ‘page’ (page as in results pagination). To give a working example if you search for a common word like ‘the’, chances are you will get multiple results per page. The IIIF search implementation will search SOLR for this issue and get x pages returned which have at least one occurrence of ‘the’. When the annotations are decompressed from SOLR this will lead to x * the number of occurrences of ‘the’ on each page.  It is possible to support paging of the returned SOLR records but this won’t tally with the actual amount of results per page. In the current implementation it cuts off at 100 results but we have considered whether it would be allowable in the Search API specification and supported in the Universal Viewer to have varying amounts of results per page. This would mean we couldn’t support the startIndex field in the results. The other alternative is to implement some kind of paging cache outside of SOLR which keeps track of the results to a query. 

The second problem we came across was linking between the results in the website e.g:

[https://journals.library.wales/search?query=%22Pryse+Family%22&range%5Bmin%5D=1735&range%5Bmax%5D=2007](https://journals.library.wales/search?query=%22Pryse+Family%22&range%5Bmin%5D=1735&range%5Bmax%5D=2007)

and if you click on the first result you get to the viewer which doesn’t highlight the words on the page. The reason for this is that you can pass complicated query parameters to SOLR including; OR, AND, NOT and speech marks among others (see https://wiki.apache.org/solr/SolrQuerySyntax for full details) which result in hits in the SOLR search but in the IIIF search implementation this needs to be matched with the compressed annotations. The common features of OR, AND, NOT are supported but currently speech mark phrases are unsupported.

### Developments with Universal Viewer 

Throughout our project we have used a standard URI structure for our document viewers of /[publicationpid]/[issuepid]/[pagenumber], for example:

* [http://newspapers.library.wales/view/3600141/3600147/64](http://newspapers.library.wales/view/3600141/3600147/64)
* [https://journals.library.wales/view/1050541/1051974/8](https://journals.library.wales/view/1050541/1051974/8)

To continue following these standards while using the UniversalViewer we had to address the problem of detecting changes in the front-end such as users navigating through issues or pages and update the URI to reflect this. JavaScript was used to process the hash parameters and to detect when either the m (issue) value or cv (page) value, and to replace the URI segments with the updated values. The issuepid was retrieved from the manifest using the publicationpid.

Originally published at 23:59, 11 April 2017 (BST) on the [NLW Dev Site](https://dev.llgc.org.uk/wiki/index.php?title=IIIF_Journals).

