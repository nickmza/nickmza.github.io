---
title: eKYC with Azure Face API & Cognitive Services
tags: azure
ogimage: fr2.png
excerpt: >-
  I wanted to see how Azure's Face API and Cognitive Services for Vision would
  stand-up against commercial off-the-shelf offerings. Here's what happened...
date: 2023-07-01 08:04:56
---


I wanted to see how Azure's Face API and Cognitive Services for Vision would stand-up against commercial off-the-shelf offerings. Here's what happened...

# Requirements
The requirement is to scan the Mauritian National Identity Card and extract the various text fields. In addition we'll need to scan the barcode found on the back of the card and compare it with the ID Number on the front. Lastly we need to compare the photo on the card with a selfie provided by the customer.

# Set-up
I started by building a simple app using Node Express & Bootstrap that would allow me to upload the ID Card and display the extracted details.

<img src="fr1.png"/>

# Identifying the Face
Azure provides a dedicated [Face API](https://azure.microsoft.com/en-gb/products/cognitive-services/face) for this purpose. Simply call the ['detect' endpoint](https://learn.microsoft.com/en-us/rest/api/faceapi/face/detect-with-stream?tabs=HTTP) passing the image data as an octet-stream. I set the returnFaceId to true and returnFaceLandmarks to false. Here are the results:

```
        {
            "faceId": "d6e2d95e-976c-4709-8f79-f6d93299408e",
            "faceRectangle": {
                "top": 696,
                "left": 200,
                "width": 413,
                "height": 413
            }
        }
```

The faceId is a reference to the underlying biometric data now residing in Azure and will be used later when we verify the selfie. The faceRectangle is exactly that - the coordinates of the face in the source image.

> I'm not posting the code here as it's trivial. If you're interested let me know in the comments and I'll share it. 

# Identifying Card Details
To extract the card details I used the Cognitive Services [Computer Vision Service](https://azure.microsoft.com/en-us/products/cognitive-services/vision-services). I called the ['analyse' endpoint](https://centraluseuap.dev.cognitive.microsoft.com/docs/services/unified-vision-apis-public-preview-2023-02-01-preview/operations/61d65934cd35050c20f73ab6) again passing the image data as an octet-stream. I set the 'features' parameter to 'read' as I want to extract text data from the image. 

Here are the results:

```
{
        "readResult": {
            "stringIndexType": "TextElements",
            "content": "REPUBLIC OF MAURITIUS\nNATIONAL IDENTITY CARD\...",
            "pages": [
                {
                    "height": 1491.0,
                    "width": 2284.0,
                    "angle": 0.8425,
                    "pageNumber": 1,
                    "words": [
                        {
                            "content": "REPUBLIC",
                            "boundingBox": [],
                            "confidence": 0.994,
                            "span": {
                                "offset": 0,
                                "length": 8
                            }
                        },
                        ...
                    ],
                    "spans": [
                        {
                            "offset": 0,
                            "length": 232
                        }
                    ],
                    "lines": [
                        {
                            "content": "REPUBLIC OF MAURITIUS",
                            "boundingBox": [],
                            "spans": [
                                {
                                    "offset": 0,
                                    "length": 21
                                }
                            ]
                        },
                    ]
                }
            ],
            "styles": [
                {
                    "isHandwritten": true,
                    "spans": [
                        {
                            "offset": 199,
                            "length": 8
                        }
                    ],
                    "confidence": 0.9
                }
            ],
            "modelVersion": "2022-04-30"
        },
        "modelVersion": "2023-02-01-preview",
        "metadata": {
            "width": 2284,
            "height": 1491
        }
    }
```
The results are separated into individual words, lines of text as well as handwritten information. Each data element is presented with the extracted text as well as the coordinates of the text in the image. 

# Displaying the Results

Here are the results. I uploaded the card images, ran the API's and rendered the results. In my testing across a number of cards the overall accuracy was extremely high. There were some issues though which I will describe later.

<img src="fr2.png"/>

## Matching fields
To figure out which fields were which I needed to map data fields to the labels. Since I knew the labels I am expecting, for example 'First Name', and I know the data for this field is immediately below it I calculated the distance from each label field to each non-label field and then selected the closest one. I added a cutoff for cases where there was no field. For example here there is no value for 'Surname at Birth' so the nearest field would have been invalid.

## Coordinate translation
I wanted to render the extracted data on the image. In addition to the aesthetic value (I am aware that I am stretching this term here when used in reference to my UI) having this visual feedback is essential for debugging. You really do need to see what is being extracted and where on the image. 

A challenge though is that the coordinates returned from the API are based on the dimensions of the source image. When the image is rendered onto the webpage, unless it is rendered to the original size, the coordinates will be invalid. To correct this you need to translate the coordinates between the source image and the destination image. I use the following approach:

```
var translatedX = (sourceX / sourceWidth) * translatedWidth;
var translatedY = (sourceY / sourceHeight) * translatedHeight;
```

# Verifying the Selfie
The last step is to verify the selfie. Once I have the image I use the 'detect' endpoint described earlier to get the faceId for the selfie. All that remains is to pass both faceId's to the ['verify' endpoint](https://learn.microsoft.com/en-us/rest/api/faceapi/face/verify-face-to-face?tabs=HTTP). Here's the result:

```
{
  "isIdentical": true,
  "confidence": 0.9
}
```
Essentially there is a 90% chance that the two faces belong to the same person or the face belongs to the person.

# Image processing Gotchas

Putting this together I ran into a few issues:

## Selfie Capture
If you are designing a flow to capture a selfie you need to ensure that your UX gives you the best possible chance to capture a high-quality selfie. As image quality drops the accuracy of the processing drops dramatically. Fortunately Microsoft has provided some excellent [guidance and sample applications](https://learn.microsoft.com/en-us/azure/cognitive-services/computer-vision/enrollment-overview) to assist.

## Image Quality
Blurry or badly scaled images throw off the face identification as well as text extraction. You will need to account for this in designing your process.

## Image Sizes 
The Azure API's have limits on the size of file you can upload. You will need to resize the image as part of your process.

## Item Padding
In our dataset there were several images where the card had been placed on a flatbed scanner and copied as an A4 page. This means that we had a lot of dead space around the card. Simply scaling the image caused problems as the scaled card lost too much data in the process to be useful. We will need to develop a mechanism to identity where on the A4 page the card is sitting and then crop the source image to those dimensions.

# Conclusion
All-in-all a great experience working with these services. We've got a basic solution running and it did not take a huge amount of time. There are still a few features, such as liveness checking and image tampering, I'd still want to play with but for now I've got a good feel for the effort and challenges involved in going this route.