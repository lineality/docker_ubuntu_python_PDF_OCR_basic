# docker_ubuntu_python_PDF_OCR_basic


# Overview
Here is sample minimal code for running keras-ocr (for img to text strings) and PyMuPDF (for PDF to img) in a flask docker image, using Flask as the micro-server for a basic output test (NOT a full api rest api setup).

#### For more about flask data endpoints see:
https://github.com/lineality/Minimal_Flask_Endpoint_API_for_Data_Science 

Follow each step and run each command in your terminal. Instructions that start with "$" are likely terminal commands. Other instructions will be text to copy into files.


## New Project Folder
It is recommended that you create a project-folder and then do your work in that folder("directory"). Run this line of code to create a project folder and then make that folder the "current working directory" where your terminal opera"my_project_folder" is arbitrary, you can name your project anything you like.
```
$ mkdir my_project_folder_keras; cd my_project_folder_OCR
```


## Check Docker Memory Use 
Check if docker is still using memory in any area:
```
$ sudo docker system df
```

## Remove Old Files (optional)
```
$ sudo docker system prune; sudo docker image prune; sudo docker volume prune; sudo docker container prune
```

## First Setup Bash Script
```
$ touch Dockerfile; touch requirements.txt; touch readme.txt; mkdir app; cd app; touch app.py; cd ..
```

## readme.md / readme.txt 
Having a readme or instructions file is helpful for documentation and clear communication. You can copy-paste the contents of this guide into that readme file. 
```
# Instructions and Guide
```

## requirements.txt
Add this text to your file. This specific minimal project has no required text, no required packages, but some docker applications require a 'requirements.txt' file even if that file is blank. (e.g. AWS-lambda-function docker-containers)
```
Flask
keras-ocr
PyMuPDF
Pillow
```

# Dockerfile
Add this text to your file. 
```
# Get docker base image
FROM ubuntu:20.04

# ?
RUN printf '#!/bin/sh\nexit 0' > /usr/sbin/policy-rc.d

# Update ubuntu linux software and packages
RUN apt-get update -y

# fix txdata glitch
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends tzdata

# Install and update pip and python-dev 
RUN apt-get install -y python3-pip python3-dev
RUN pip3 install --upgrade pip

# Install setuptools
RUN pip3 install setuptools
RUN pip3 install scikit-build
RUN pip3 install tensorflow

# Install opencv-python
# RUN apt-get install ffmpeg libsm6 libxext6 -y
# RUN pip3 install opencv-python
RUN apt-get install libgl1 -y
RUN apt-get install libgtk2.0-dev -y

# Import requirements.txt into docker
COPY ./requirements.txt /requirements.txt

# Set working directory
WORKDIR /

# Install python requirements
RUN pip3 install -r requirements.txt

# Copy all files in project directory into docker
COPY . /

# Specify the executable to run
ENTRYPOINT [ "python3" ]

# The function of this docker is to run the app.py
CMD [ "app/app.py" ]
```

# app.py
Add this text to your file. 
```
# Import python libraries and packages
import fitz
from flask import Flask
import keras_ocr
import numpy as np
from PIL import Image 
import requests

##########
# Get PDF
##########

# sample
url = 'https://github.com/lineality/docker_ubuntu_python_PDF_OCR_api/raw/main/cert1.pdf'

myfile = requests.get(url)

open("PDF_file.pdf", 'wb').write(myfile.content)


######################
# Convert PDF to .png
######################

# read PDF
doc = fitz.open("PDF_file.pdf") 


# iterate through doc pages
for page in doc:

    # get pixel map
    pix = page.get_pixmap()

    # save as image file: .png
    pix.save("image_file.png")


#################################
# PIL/Pillow image preprocessing 
#################################

img_path = "image_file.png"

# load and resize image file
"""
equivalent of this from Keras:

img = image.load_img(img_path, target_size=(224, 224))
"""
img = Image.open(img_path)
# img = img.resize((224, 224))

# image -> array
"""
equivalent of this from Keras:

img_array = image.img_to_array(img)
"""
img_array = np.asarray(img)

# removed extra pre-processing for now
"""
# already numpy
expanded_img_array = np.expand_dims(img_array, axis=0)

# already numpy
preprocessed_img = expanded_img_array / 255. 

# set: input_data = preprocessed image
input_data = preprocessed_img

# type cast to float32
input_data = input_data.astype('float32')
"""

# type cast to float32
input_data = img_array.astype('float32')

######################
# Set Up OCR Pipeline
######################

# pre-setup
pipeline = keras_ocr.pipeline.Pipeline()
images = [input_data]

# Terminal Print: data arrays
print(input_data)

# Variables to store data arrays
image0 = input_data

# store words in lists/arrays
text_1 = []

# build NLP/OCR pipeline
prediction_groups = pipeline.recognize(images)

# run prediction on image 1
predicted_image_1 = prediction_groups[0]
for text, box in predicted_image_1:
    # add text to list of words
    text_1.append( text )

# Flask "app" (note, in AWS maybe named differently)
app = Flask(__name__)

# main function
@app.route("/")
def index():
    return f"""
    <h1>Hello World!</h1>
    <h6>{image0}</h6>
    <h6>{text_1}</h6>
    <p> |O...O| </p>
    """

if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0')
```

## Build your Docker image
Add this text to your file. 

"project_abc" is an arbitrary project name, you can use any name you like.

The final period "." is not a typo:
```
sudo docker build -t project_abc .
```

#### Successful builds output these lines at end:
```
Successfully built ############
Successfully tagged ###:latest
```

## Run your Docker container
```
$ sudo docker run --name flaskapp -v$PWD/app:/app -p5000:5000 project_abc:latest
```

Note: If you are using windowsOS, use: "docker run --name flaskapp -v%cd%/app:/app -p5000:5000 project_abc:latest"


## See you results in a browser
Now the application can be accessed at 

http://localhost:5000 or  http://0.0.0.0:5000/

#### Successful run should print a display to a browser.
```
Hello, World... etc.
```

## Remove Old Files 
```
$ sudo docker system prune; sudo docker image prune; sudo docker volume prune; sudo docker container prune
```


## Look for Old Files  (optional)
Check if docker is still using memory in any area:
```
sudo docker system df

sudo docker container ls -a
```

### If a container is still running: Get container name; Then stop container; Then prune; Check
```
sudo docker container ls -a

sudo docker stop CONTAINER_NUMBER_HERE

sudo docker container prune

sudo docker system df
```




## Sources and Resources
#### use ubuntu, install docker
- https://docs.docker.com/engine/install/ubuntu/

#### Sources and Resources
- https://linuxize.com/post/how-to-list-docker-containers/ 
- https://linuxhandbook.com/remove-docker-images/ 
- https://docs.docker.com/engine/install/ubuntu/
- https://www.freecodecamp.org/news/how-to-remove-all-docker-images-a-docker-cleanup-guide/ 
- https://www.educba.com/docker-entrypoint/
- https://stackoverflow.com/questions/46247032/how-to-solve-invoke-rc-d-policy-rc-d-denied-execution-of-start-when-building 
- https://linuxhint.com/docker-entrypoint-beginner-guide/ 
- https://www.gcptutorials.com/post/extract-text-from-images-using-keras-ocr-in-python 
- https://www.codethebest.com/modulenotfounderror-no-module-named-skbuild-solved/ 
- https://stackoverflow.com/questions/55313610/importerror-libgl-so-1-cannot-open-shared-object-file-no-such-file-or-directo 
- https://www.tensorflow.org/install/
- https://hub.docker.com/r/tensorflow/tensorflow/ 
- https://www.gcptutorials.com/post/extract-text-from-images-using-keras-ocr-in-python 
- https://medium.datadriveninvestor.com/dockerizing-and-hosting-your-flask-web-app-rest-api-on-aws-ec2-9f9c189bf563 
- https://likegeeks.com/downloading-files-using-python/ 

## Sample Output from Browser
![Image](https://github.com/lineality/docker_ubuntu_python_PDF_OCR_basic/blob/main/bowser_output_ocr_mvp1.png)

## Sample PDF
![Image](https://github.com/lineality/docker_ubuntu_python_PDF_OCR_basic/raw/main/cert1.png)
