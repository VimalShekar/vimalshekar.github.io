---
layout: post
title: Uploading to an S3 bucket from bitbucket pipelines
date: 2018-04-10
category: ScriptSamples
---

I was recently writing a build script that should get triggered whenever I push my code to Bitbucket using the pipelines feature, see [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines). Basically you create a yaml file with tasks. Each time you push code, Bitbucket spins up a linux docker container to run these tasks one after another. In my case the task was simple - I just had to package my powershell scripts into a zip file and upload it to my AWS S3 bucket. 

# Pre-Reqs:
1. To upload files to S3, first create a user account and set the type of access to allow 
"Programmatic access", see [this](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html). You should also set permissions to ensure that the user has access to the bucket. Make sure you set this up first.

2. Create a pipeline using steps in [this link](https://confluence.atlassian.com/bitbucket/get-started-with-bitbucket-pipelines-792298921.html). This will create a bitbucket-pipelines.yml file in the root of your repository. Note that you cannot use tabs, you have to indent using spaces. You should use the bitbucket's interface when possible, because it warns you when you have \t characters in the yaml file.

# Yaml file
You can write line comments using "#". The first real line in the yaml file specifies the docker image you want to use. In my case I used the standard Python 3.5.1 image, so I have "image: python:3.5.1" as the image.

Bitbucket allows you to create a default pipeline, or seperate pipelines for each of your branches (patterns). There are predefined sections for "pipelines", "branches", "step" that Bitbucket pipelines understand. I just wanted one for my master branch and one for my dev branch - so I used the following template. "master" refers to the master branch in my repository and "devbranch" refers to my development branch. We would be using the Amazon provided boto3 package to perform upload, so one of the steps in your yaml will be to use pip to install the package. You can *cache* pip, so that your image already has it and doesn't have to download it each time you run the pipeline.

{% highlight yaml %}
    image: python:3.5.1

    pipelines:
        branches:
            master:
                -step:
                    caches: # Caching pip to reduce time taken for the pipeline
                        - pip
                    script:
                        # This is a comment, Will add steps for the master branch here

            devbranch:
                -step:
                    caches:
                        - pip
                    script:
                        # This is a comment, Will add steps for the dev branch here
{% endhighlight %}

I prefer to keep my build scripts and python requirements file it in a build folder in my repository. So the first step is to cd to this folder and then call pip to install the requirements. The requirements.txt file just has this one line for now:
    {% highlight yaml %}
    boto3
    {% endhighlight %}

The steps to be performed are:
    echo "Build started"
    cd .\build
    pip install -r requirements.txt
    python ./BuildScript.py package_name
    echo "Build complete"

Here's my completed bitbucket-pipelines.yml file.
{% highlight yaml %}
    image: python:3.5.1

    pipelines:
        branches:
            master:
                -step:
                    caches: # Caching pip to reduce time taken for the pipeline
                        - pip
                    script:
                        - echo "Master branch Build started"
                        - cd .\build
                        - pip install -r requirements.txt
                        - python ./BuildScript.py package_name
                        - echo "Build complete"

            devbranch:
                -step:
                    caches:
                        - pip
                    script:
                        - echo "Dev branch Build started"
                        - cd .\build
                        - pip install -r requirements.txt
                        - python ./BuildScript.py package_name
                        - echo "Build complete"
                        
{% endhighlight %}

Now for the actual Python script, thats pretty straight forward. You import boto3, create an instance of boto3.resource for the s3 service. Call the upload_file method and pass the file name.   
  
In the below example:  
- "src_files" is an array of files that I need to package.   
- "package_name" is the package name.  
- "bucket_name" is the S3 bucket name that I want to upload to.  
- "my_aws_access_key_id" and "my_aws_secret_access_key" are the access keys, I have hard coded theses just as an example. I would strongly advice you against doing that, especially if your repository is not a private one.  
- ZipDirectory is a function that wraps around shutil.make_archive to create the package.  
- CreateZipPackageAndUpload is a function that copies the file to a temporary folder, zips the folder and uploads to the S3 bucket.  
  
Here's the sample python script:  
  
{% highlight python %}
################################################################
# imports
import os
import shutil
import sys, getopt 
import boto3


################################################################
# global variables
currDir = os.curdir
src_files = [
    currDir + os.sep + "script1.ps1",
    currDir + os.sep + "script2.ps1",
    currDir + os.sep + "script3.ps1",
]
package_name = "mypackage.zip"
bucket_name = "myS3bucket"
my_aws_access_key_id='MyUserID', 
my_aws_secret_access_key='MyUserAccessKey'


################################################################
# function definitions

#
# Check if string is null or empty
#
def IsStrNullOrEmpty(str):
    if str and str.strip():
        return False
    else:
        return True


#
# Zips the contents of a directory to create a zip file.
#
def ZipDirectory(srcDirectoryPath, zipFileName, dstDirectoryPath = currDir, format = "zip"):
    # check if paramters are null or empty
    if IsStrNullOrEmpty(zipFileName):
        print("ZipDirectory():Paramter zipFileName is null or empty")
        return False        
    
    if IsStrNullOrEmpty(srcDirectoryPath):
        print("ZipDirectory():Paramter srcDirectoryPath is null or empty")
        return False
    
    # check if the file already exists, if so - delete the existing one
    if os.path.exists(zipFileName) and os.path.isfile(zipFileName):                        
        print("ZipDirectory():File with the name {0} already exists. Deleting it now.".format(zipFileName))
        os.remove(zipFileName)
    
    if zipFileName.endswith(".zip"):
        zipFileName = zipFileName.replace(".zip", "")
    
    # Make the archive
    print("ZipDirectory():Creating file {0}.zip".format(zipFileName))
    shutil.make_archive(zipFileName, format, dstDirectoryPath, srcDirectoryPath)
    
    if os.path.exists(zipFileName):
        print("ZipDirectory():File is now created:{0}.zip".format(zipFileName))
        return True
    else:
        print("ZipDirectory():File creation failed:{0}.zip".format(zipFileName))
        return False

#
# Function to create the zip file package containing scripts.
#
def CreateZipPackageAndUpload(zipFileName = "", files_to_zip = src_files, bucket_name = "myS3bucket" ):
    
    # check if the folder exists, if so - delete the existing one
    if os.path.exists(zipFileName):
        if os.path.isdir(zipFileName):
            print("CreateZipPackageAndUpload():Directory with the name {0} already exists. Deleting it now.".format(zipFileName))
            shutil.rmtree(zipFileName)
        else:
            print("CreateZipPackageAndUpload():File with the name {0} already exists. Deleting it now.".format(zipFileName))
            os.remove(zipFileName)

    if zipFileName.endswith(".zip"):
        zipFileName = zipFileName.replace(".zip", "")

    # create the folder and copy the files to it. 
    os.mkdir(zipFileName)
    dst_foldername = zipFileName + os.sep
 
    # copy the remaining files from the root folder
    for filename in src_files:
        if os.path.exists(filename):             
            print("CreateZipPackageAndUpload():Copying file {0} to folder {1}.".format(filename, dst_foldername))
            shutil.copy2( filename, dst_foldername )
        else:
            print("CreateZipPackageAndUpload():File {0} does not exist, and hence could not be copied.".format(filename))
    
    # Create the zip file
    ZipDirectory(zipFileName, zipFileName)    
    print("CreateZipPackageAndUpload(): zip file {0} created.".format(zipFileName + ".zip"))

    # Remove Directory
    shutil.rmtree(zipFileName)    

    # Upload the zip file    
    s3.Bucket(bucket_name).upload_file( zipFileName + ".zip", zipFileName + ".zip" )
    print("CreateZipPackageAndUpload(): zip file {0} was uploaded to S3 bucket.".format(zipFileName + ".zip"))
    
    Bucket_Objs = [obj.key for obj in s3.Bucket(bucket_name).objects.all()]    
    print("--> Printing contents in the bucket: %s" % Bucket_Objs)

    return


################################################################
# Script execution starts here...

print("Hello world!")

# Create an AWS S3 resource object
s3 = boto3.resource(
    's3', 
    aws_access_key_id = my_aws_access_key_id, 
    aws_secret_access_key = my_aws_secret_access_key
)

if(s3 == None):
    print("Error: Could not create an S3 resource object")    
else:
    print("Confirmed that AWS S3 buckets are reachable...")
    #Create the package and upload it to S3
    CreateZipPackageAndUpload(package_name, src_files, bucket_name )

print("Goodbye world")

################################################################
{% endhighlight %}

Hope this helps!
