---
layout: post
title: Downloading from S3 bucket using C#
date: 2018-04-12
category: CodeSamples ScriptSamples
---

A couple of days ago, I wrote a python script and Bitbucket build pipeline that packaged a set of files from my repository into a zip file and then uploaded the zip file into an AWS S3 bucket. Thats one side done, so anytime my scripts change, I push to Bitbucket and that automatically updates my S3 bucket. Now its time to write the other side, the client that downloads the file from the S3 bucket and extracts it. 

If your bucket is a public one, then anyone has access to the URL and so downloading it becomes easy. You can use the HttpClient class to easily download the file.

If access to the bucket is restricted, then you can use AWS APIs that are part of the AWS SDK for .NET, to download the file. That is what we will look into today:

# Pre-Reqs:
1. In order to access the bucket, you have to have to authenticate with AWS. There are many ways to do that, in my case - I created an IAM user with programmatic access and gave him read permissions on my bucket (Refer [this](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)). Once you have done that, you will have the access key and the secret key for that user.

2. To work with AWS S3, Amazon has provided a set of APIs. You can either download and install the complete AWS SDK for .NET, or just install the AWSSDK.S3 package using NuGetPackage manager. To install just the AWS.S3 just open Visual Studio NuGet package manager, and search for AWSSDK.S3, then click the install option. Once installed, the NuGet Package manager will add a reference to your project automatically.

![NuGet]({{ "/images/NuGet-AWSS3.jpg" | absolute_url }})


# The Code
Now that you have your IAM user credentials and the AWSSDK.S3 package installed, lets focus on getting our zip file from S3 and extracting it. 

1. First, create an instance of AmazonS3Config and specify the region name:
{% highlight csharp %}
    //-- Set the region or service url in the config
    AmazonS3Config config = new AmazonS3Config();
    config.RegionEndpoint = RegionEndpoint.GetBySystemName(regionName);  //-- replace regionName with your region name Ex: "us-east-1"
{% endhighlight %}

2. Then create an instance of AmazonS3Client class and pass your access key and secret key along with the using the AmazonS3Config object you just created.
{% highlight csharp %}
    AmazonS3Client s3Client = new AmazonS3Client(accessKey, secretKey, config);
{% endhighlight %} 

3. If you know your bucket name, you create a ListObjectsRequest object with the bucket name and pass that to AmazonS3Client's ListObjects()method. It's as simple as this:
{% highlight csharp %}
    ListObjectsRequest request = new ListObjectsRequest();
    request.BucketName = bucketName;

    ListObjectsResponse response = s3Client.ListObjects(request);
    foreach (var objt in response.S3Objects)
    {
        Console.WriteLine("Object Name:{0}   Last modified:{1}", objt.Key, objt.LastModified);                
    }
{% endhighlight %}  

4. To download the object, you create a GetObjectRequest object with the bucket name and the object Key that you want to retrieve and then pass that to AmazonS3Client's GetObject method. You can write the object to the file system using WriteResponseStreamToFile() method of the GetObjectResponse.
{% highlight csharp %}
    GetObjectRequest request1 = new GetObjectRequest();
    request1.BucketName = bucketName;
    request1.Key = caFileName;

    GetObjectResponse response1 = s3Client.GetObject(request1);
    response1.WriteResponseStreamToFile("C:\\temp\\mypackage.zip");    
{% endhighlight %}  

5. To Extract the zip file, you could use .NET's built in [ZipFile class](https://msdn.microsoft.com/en-us/library/system.io.compression.zipfile(v=vs.110).aspx). Be sure to add a reference to System.Io.Compression.FileSystem library, and add "using System.IO.Compression" to your namespace declarations.
{% highlight csharp %}
    ZipFile.ExtractToDirectory("C:\\temp\\mypackage.zip", "C:\\temp\\extracted");    
{% endhighlight %} 

Here's the complete code:

{% highlight csharp %}

using System;
using System.Collections.Generic;
using System.IO.Compression;
using Amazon;
using Amazon.S3;
using Amazon.S3.Model;

namespace DownloadFromS3
{
    class Program
    {
        static string accessKey = "YourAccessKeyHere";
        static string secretKey = "YourSecretKeyHere";
        static string bucketName = "YourBucketNameHere";
        static string regionName = "YourRegion"; //Ex: "us-east-1";
        static string zipFileName = "mypackage.zip";        
        static string extractToFolder = "C:\\temp\\MyPackage";

        static void Main(string[] args)
        {   
            //-- Set the region or service url in the config
            AmazonS3Config config = new AmazonS3Config();
            config.RegionEndpoint = RegionEndpoint.GetBySystemName(regionName);
            
            //-- Create the client
            AmazonS3Client s3Client = new AmazonS3Client(accessKey, secretKey, config);

            //-- To List contents of our bucket, uncomment the below section of code
            //ListObjectsRequest request = new ListObjectsRequest();
            //request.BucketName = bucketName;
            //ListObjectsResponse response = s3Client.ListObjects(request);
            //foreach (var objt in response.S3Objects)
            //{
            //    Console.WriteLine("{0}\t{1}", objt.Key, objt.LastModified);                
            //}

            //-- Get the Core-Analysis zip package
            GetObjectRequest request1 = new GetObjectRequest();
            request1.BucketName = bucketName;
            request1.Key = zipFileName;
            GetObjectResponse response1 = s3Client.GetObject(request1);

            // Use a temporary file name to download the zip file.
            string tmpfilename = System.IO.Path.GetTempFileName();            
            response1.WriteResponseStreamToFile(tmpfilename);            

            ZipFile.ExtractToDirectory(tmpfilename, extractToFolder);
            Console.WriteLine("File Extracted to:{0}", extractToFolder);

            // Delete the temp file after contents are extracted
            if( System.IO.File.Exists(tmpfilename) )
            {
                System.IO.File.Delete(tmpfilename);
            }
        }
    }
}

{% endhighlight %}

Hope this helps!
