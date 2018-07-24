+++
date = "2018-02-11T13:23:20-05:00"
title = "Scripted S3 Site Deployment with AWS CLI and Powershell, Part 2"
keywords = [ "jpiedra", "aws", "powershell", "scripting", "part-2", "s3" ]
tags = [ "aws", "powershell", "scripting", "s3", "part 2" ]
+++

In this final post for a two-part series, we'll go over an additional script that can be used to upload all local static site contents to a previously configured Amazon Web Services S3 bucket. 

<!--more-->
<h2>Not Stubbing the Commandlet</h2>
As previously mentioned, the complete version of the scripts shown here can be found at <a href="https://github.com/jpiedra/S3SiteDeploy">this Github repository.</a> For this example, we'll be building a Powershell script that accepts the <i>name</i> of a bucket to push files to. We've already provisioned the bucket using <i>New-S3Site.ps1</i> so if you were using this script, you would set this to the same value you used when running that script.

It turns out that writing this script is much easier; we're going to use the same AWS S3 CLI command to upload all of our site's contents. This will leave us with multiple, very similar lines where the only difference is the <i>file extension</i> and <i>content type</i> for each type of file we'll upload (HTML, CSS, JS, so on). We'll explain how the bulk of this script works by explaining the command used. The complete script is shown below:

<pre>
    [CmdletBinding()]
    Param(
        # Name of the bucket being configured  
        [Parameter(Mandatory=$true,Position=1)]
        [String]
        $Name,

        # Name of the path on the local computer to copy files from, assume current working directory by default
        [String]
        $Path = $pwd.Path
    )

    # [https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html] :: upload boilerplate files
    aws s3 cp $Path s3://$($Name) --exclude "*" --include "*.html" --recursive --metadata-directive REPLACE `
        --content-type text/html --cache-control public,max-age=604800

    # Upload stylesheets
    aws s3 cp $Path s3://$($Name) --exclude "*" --include "*.css" --recursive --metadata-directive REPLACE `
        --content-type text/css --cache-control public,max-age=604800

    # Upload JavaScript files
    aws s3 cp $Path s3://$($Name) --exclude "*" --include "*.js" --recursive --metadata-directive REPLACE `
        --content-type text/javascript --cache-control public,max-age=604800

    # Upload PNG images
    aws s3 cp $Path s3://$($Name) --exclude "*" --include "*.png" --recursive --metadata-directive REPLACE `
        --content-type image/png --cache-control public,max-age=604800

    # Upload JPG images
    aws s3 cp $Path s3://$($Name) --exclude "*" --include "*.jpg" --recursive --metadata-directive REPLACE `
        --content-type image/jpg --cache-control public,max-age=604800

    # Set the website configuration for the bucket, setting the index and error pages.
    aws s3 website s3://$($Name) --index-document index.html --error-document error.html
</pre>

<h2>AWS Copy Command</h2>

The command <a href="https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html"><b>aws s3 cp</b></a> copies files to, from and between S3 buckets. In addition to copying files, a variety of parameters are provided to change how the copying operations perform, as well as attributes and metadata for each of the copied files. 

The most simple example we could start with would copy files in the current directory to a bucket we previously provisoned, where the name of the bucket is substituted from the <i>$Name</i> parameter in the required format. To copy files inside the directory, we also need to include the <i>--recursive</i> parameter:

<pre>
    <b>aws s3 cp . s3://$($Name) --recursive</b>
</pre>

<h2>Exclude and Include Parameters</h2>
Now it's worth noting that this command will copy over everything in the current directory to the specified bucket. We need to change that behavior, so that this command only copies over a specific format of file. This requires two parameters, <a href="https://docs.aws.amazon.com/cli/latest/reference/s3/index.html#use-of-exclude-and-include-filters"><i>--exclude</i> and <i>--include</i></a>.

The first of these is <i>--exclude</i> which can accept a wildcard value. This causes the copy command to exclude everything - all the files inside of the current directory. Once we've added this parameter, the second parameter <i>--include</i> is used with a wildcard pattern, where anything with the specified pattern in its filename gets picked and copied to the bucket. This has the effect of copying over all the HTML files inside the current directory:

<pre>
    aws s3 cp . s3://$($Name) <b>--exclude "*" --include "*.html"</b> --recursive
</pre>

<h2>Content Type and Cache-Control Metadata</h2>
The <i>--metadata-directive</i> parameter can accept one of two values, COPY and REPLACE. We're interested in the REPLACE option, which will always replace the metadata values for the bucket's copy of the file to whatever we specify in our command. Here, we're only going to specify two metadata values: One to tell S3 that we're uploading a file with a content type of HTML, and another to set the expiration of the file to 7 days. After 7 days, visitors of our website will need to download the file again. 

As a minor formatting concern, we'll break this command into two lines by using the backtick(`) character, which Powershell interprets appropriately:

<pre>
    aws s3 cp . s3://$($Name) --exclude "*" --include "*.html" --recursive <b>--metadata-directive REPLACE `
        --content-type text/html --cache-control public,max-age=604800</b>
</pre>

The <i>--content-type</i> parameter is set to the appropriate <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types">MIME type value that corresponds to an HTML page file.</a> We can get the required value for this file from MDN's "Incomplete list of MIME Types" page. Finally, the <i>--cache-control</i> parameter is set to indicate an expiration of 7 days, expressed as seconds. MDN also has <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control">a page describing possible values for this metadata attribute.</a>

<h2>Choosing our File Types</h2>
At this point we have a command we can use to upload all HTML files in the current directory. Copying other types of files over is as easy as copying this line, and substituing the file pattern used in <i>--include</i>, as well as the <i>--content-type</i> metadata attribute for that file type (easily found from the previous MIME types list). This leaves us with:

<pre>
    # [https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html] :: upload boilerplate files
    aws s3 cp . s3://$($Name) --exclude "*" --include "*.html" --recursive --metadata-directive REPLACE `
        --content-type text/html --cache-control public,max-age=604800

    # Upload stylesheets
    aws s3 cp . s3://$($Name) --exclude "*" --include "*.css" --recursive --metadata-directive REPLACE `
        --content-type text/css --cache-control public,max-age=604800

    # Upload JavaScript files
    aws s3 cp . s3://$($Name) --exclude "*" --include "*.js" --recursive --metadata-directive REPLACE `
        --content-type text/javascript --cache-control public,max-age=604800

    # Upload PNG images
    aws s3 cp . s3://$($Name) --exclude "*" --include "*.png" --recursive --metadata-directive REPLACE `
        --content-type image/png --cache-control public,max-age=604800

    # Upload JPG images
    aws s3 cp . s3://$($Name) --exclude "*" --include "*.jpg" --recursive --metadata-directive REPLACE `
        --content-type image/jpg --cache-control public,max-age=604800
</pre>

<h2>AWS Website Command</h2>
We're nearing the end of writing this script. Our bucket, after running this script, will now have all the files copied over from the current working directory we run this script in. Now we just need to tell AWS which files (already in the bucket) to use as the index and error page for this static website. Fortunately, there's a command for that too: the aptly named <a href="https://docs.aws.amazon.com/cli/latest/reference/s3/website.html"><b>aws s3 website</b></a> command.

<pre>
    aws s3 website s3://$($Name) --index-document index.html --error-document error.html
</pre>

<h2>Finishing Touches</h2>
To make our commandlet a bit more flexible, as well as cooperative with the other commandlet calling it (New-S3Site.ps1) we replace any indication of the current working directory (.) with the <i>Path</i> parameter value. This is the same code we used to define the parameter in that other script, and it will use a default value of the present working directory, allowing us to specify a location as well. Whether using the default value or one we set, the AWS copy command will now look there for which files to copy to the bucket:

<pre>
    [CmdletBinding()]
    Param(
        # Name of the bucket being configured  
        [Parameter(Mandatory=$true,Position=1)]
        [String]
        $Name,

        # Name of the path on the local computer to copy files from, assume current working directory by default
        [String]
        $Path = $pwd.Path
    )

    # [https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html] :: upload boilerplate files
    aws s3 cp <b>$Path</b> s3://$($Name) --exclude "*" --include "*.html" --recursive --metadata-directive REPLACE `
        --content-type text/html --cache-control public,max-age=604800

    # Upload stylesheets
    aws s3 cp <b>$Path</b> s3://$($Name) --exclude "*" --include "*.css" --recursive --metadata-directive REPLACE `
        --content-type text/css --cache-control public,max-age=604800

    # Upload JavaScript files
    aws s3 cp <b>$Path</b> s3://$($Name) --exclude "*" --include "*.js" --recursive --metadata-directive REPLACE `
        --content-type text/javascript --cache-control public,max-age=604800

    # Upload PNG images
    aws s3 cp <b>$Path</b> s3://$($Name) --exclude "*" --include "*.png" --recursive --metadata-directive REPLACE `
        --content-type image/png --cache-control public,max-age=604800

    # Upload JPG images
    aws s3 cp <b>$Path</b> s3://$($Name) --exclude "*" --include "*.jpg" --recursive --metadata-directive REPLACE `
        --content-type image/jpg --cache-control public,max-age=604800

    # Set the website configuration for the bucket, setting the index and error pages.
    aws s3 website s3://$($Name) --index-document index.html --error-document error.html
</pre>

<h2>The Scripts in Action</h2>
Running the scripts produces output indicating our bucket gets created and has files uploaded from our local 'public' folder to the bucket.

<a href="https://imgur.com/cGnfCQB"><img src="https://i.imgur.com/cGnfCQB.gif" title="source: imgur.com" /></a>

Once created, we have the bucket available in the S3 management console with the appropriate access configured.

<a href="https://imgur.com/M2P6bmU"><img src="https://i.imgur.com/M2P6bmU.png" title="source: imgur.com" /></a>

<a href="https://imgur.com/q99Hdwe"><img src="https://i.imgur.com/q99Hdwe.png" title="source: imgur.com" /></a>

And, visiting the S3 bucket URL works as well, with both our index and error pages functioning.

<a href="https://imgur.com/XFIYI9E"><img src="https://i.imgur.com/XFIYI9E.gif" title="source: imgur.com" /></a>

Additional Resources
<ul>
<li><a href="https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html"><b>AWS S3 CP command</b></a></li>
<li><a href="https://docs.aws.amazon.com/cli/latest/reference/s3/index.html#use-of-exclude-and-include-filters">Using the <i>--exclude</i> and <i>--include filters</i></a></li>
<li><a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types">Mozilla Developer Network Incomplete List of MIME types</a></li>
<li><a href="https://docs.aws.amazon.com/cli/latest/reference/s3/website.html"><b>AWS S3 Website command</b></a></li>