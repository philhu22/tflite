
## Create a Training Dataset
### Collect the images
#### download
I did what most data scientist beginners do: use Google to search pictures in huge repositories like [unplash](https://unsplash.com/), [istockphotos](https://www.istockphoto.com/), [pixabay](https://pixabay.com/). I used the Chrome extension [Download All Images](https://download-all-images.mobilefirst.me) to quickly save the images in zip files. After unzipping, I moved all (543) images in a single folder and inspected them visually to remove images with no key or many keys. I tried to remove also duplicates. When done, my collection counted 292 images.
#### rename
Then, I changed the default names to something simpler to read: key(i), where i is the file index. On Windows, using File Explorer to batch rename files is usually the easiest way. To batch rename files, just select all the files you want to rename, press the F2 (alternatively, right-click and select rename), then enter the name you want on the first file. Press Enter to change the names for all other selected files. This method adds sequential numbers in parentheses beside each file name:
![keys1](/assets/images/keys1.png)
#### resize
The images used in the training video have a size of 800 x 600 pixels So, I resized all images to that dimensions. To do that, I used the [PowerToys Image Resizer utility for Windows](https://docs.microsoft.com/en-us/windows/powertoys/image-resizer) with the following options:

![keys2](/assets/images/keys2.png)

Finally, I have rotated the vertical images so that I finishd with 292 images with w=800 and h=600 pixels

### Data Annotation
#### Label Studio
I used the Community edition of  [Label Studio](https://labelstud.io/) from [Heartex](https://heartex.com/) to annotate the images. This tool has a Web interface and is very easy to use. A Docker image is available that makes the installation effortless. On Windows, to install and start Label Studio at http://localhost:8080, storing all labeling data in the current directory, run the following in a powershell terminal:

```bash
docker run -it -p 8080:8080 -v $pwd/:/label-studio/data heartexlabs/label-studio:latest
```

There is a great introduction to Label Studio on the [Youtube Heartex channel](https://www.youtube.com/watch?v=A0cob_f5BmM&list=PLDqcjLIsFtX3t1jqjW8BXW1EDTOczDpHH).

#### label the images
After importing the images, we must select the labeling interface. In our case, the template "Object Detection with Bounding Boxes" is used:

![keys3](/assets/images/keys3.png)

Next, we specify the labels: (regular) key and car key. We check the options to control the image zoom and rotation:

![keys4](/assets/images/keys4.png)

And finally, we go through all images and draw the bounding boxes around the keys:

![keys5](/assets/images/keys5.png) ![keys6](/assets/images/keys6.png)

#### export data
We will use the [Tensorflow Lite Model Maker](https://www.tensorflow.org/lite/guide/model_maker) to quickly train a model using transfer learning.  The Model Maker's object detection task can load data described with the [Pascal Visual Object Classes(VOC)](https://towardsdatascience.com/coco-data-format-for-object-detection-a4c5eaf518c5). So, we selected the Pascal VOC XML format to export our data:

![keys7](/assets/images/keys7.png)

The Export command downloads a zip file containing 2 folders with the images and the XML annotation files: 

![keys8](/assets/images/keys8.png)

A XML annotation file starts with these lines:

![keys9](/assets/images/keys9.png)

There are 2 problems:
1. The xml python parser does not expect the encoding line;
2. The filname is missing the file extension (.jpg)

We used Notepad++ to batch edit the xml files, remove the first line `<?xml version="1.0" encoding="utf-8"?>` and replace `</filename>` by `.jpg</filename>`

