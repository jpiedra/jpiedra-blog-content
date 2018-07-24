+++
date = "2018-01-20T16:29:20-05:00"
title = "Scripted S3 Site Deployment with AWS CLI and Powershell, Part 1"
keywords = [ "jpiedra", "aws", "powershell", "scripting", "part-1", "s3" ]
tags = [ "aws", "powershell", "scripting", "s3", "part 1" ]
+++

Amazon Web Services, through their Simple Storage Service (S3), provide an inexpensive and flexible solution for hosting static websites. These would normally be sites that are developed on a local environment (using <a href="https://gohugo.io/overview/quickstart/">Hugo</a> or <a href="https://jekyllrb.com/">Jekyll</a>), then pushed to either a web server or - in this case - a cloud-based storage platform configured to make the generated pages public. This post discusses a scripted approach to building a bucket you want to use to host an S3 website from scratch.

<!--more-->
<h2>AWS CLI</h2>
Before doing anything that entails making scripted requests to AWS resources, the AWS Command Line Interface will be required on your system. <a href="https://docs.aws.amazon.com/cli/latest/userguide/installing.html">Refer to Amazon's documentation for more details.</a>

Once you have the AWS CLI configured, with keys set up for an account that has permission to configure S3 resources and policies, we can begin writing a script to build a new S3 bucket host site hosting. The finished project can be found at <a href="https://github.com/jpiedra/S3SiteDeploy">this Github repository.</a>

<h2>Stubbing the Commandlet</h2>
For this example, we'll be building a Powershell script that accepts the <i>name</i> of a bucket you'd like to create. We also specify a parameter to indicate which <i>path</i> on our local system to copy files from. 

<ol>
<li>First, a check is performed to see if the bucket exists. If no bucket exists using the provided name, then the AWS CLI command will be run to create one using that name.</li>
<li>We create the bucket, checking for success of that operation.</li>
<li>Finally, we apply a default policy so those contents are publically available. While S3 does offer a highly programmable way to define bucket resource access and permissions, we can assume for our purposes that all parts of this website should be publically readable. Default contents for a website can be uploaded as well.</li>
</ol>

Using the text-editor of our choice, we create a new file, <i>New-S3Site.ps1</i> and add the following contents. While any editor will do, using <a href="https://docs.microsoft.com/en-us/powershell/scripting/getting-started/fundamental/windows-powershell-integrated-scripting-environment--ise-?view=powershell-6">Powershell Integrated Scripting Environment (ISE)</a> gives us the added bonus of running our script while writing it.

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
    $bucketName = "s3://$($Name)"
    # make sure the bucket doesn't already exist, checking for an ErrorRecord
    # if your bucket was created successfully, proceed to configure/upload defaults
        # configure default minimal viable policy, using Set-Content
            # upload your site contents
        }
    }  
</pre>

<h2>Check for the S3 Bucket</h2>
The name parameter's value is used immediately to create a legal S3 URL, prefixed with the value <i>s3://</i>. We can pass this new variable, <i>$bucketName</i> to our first AWS CLI command, <b>aws s3 ls</b>. Exactly as you'd expect on a Unix system, this will attempt to do a listing of the bucket. 

<pre>
...
    $bucketName = "s3://$($Name)"
    # make sure the bucket doesn't already exist
    <b>aws s3 ls $bucketName</b>
...
</pre>

While this gets us our check very quickly, to better shape the behavior of our script, we should capture the result of running this command to a variable that can then be used to determine the next step to take, rather than simply print it out on the screen. If you run this command after providing a non-existant bucket name, you'll see something similar to this:
</br>
<a href="https://imgur.com/ooOYquz"><img src="https://i.imgur.com/ooOYquz.png" title="source: imgur.com" /></a>
</br>
</br>
Now let's capture the contents of this error message by doing two things:
<ol>
<li>Redirect the output of this command to standard out, from standard error</li>
<li>Store the output in a new variable</li>
</ol>

<pre>
...
    $bucketName = "s3://$($Name)"
    # make sure the bucket doesn't already exist
    <b>$result = aws s3 ls $bucketName 2>&1</b>
...
</pre>

The <i>$result</i> variable will now contain an ErrorRecord object if ls fails. (Actually, this turns out to be an <i>array</i> with one object inside of it, of type ErrorRecord) We know that, if the result of this ls command is an ErrorRecord, that the bucket does not exist. This is useful information that can be leveraged to perform the check we need before proceeding with the creation process, because if we <i>don't</i> get an error record, then <i>$result</i> will be null (as nothing came from standard error), and we can check if the bucket exists by checking whether or not <i>$result</i> is null. If null, the ls succeeded, and thus we should stop the script because there's a bucket there we don't want to tamper with!

<ol>
<li>Wrap the code that checks for the bucket in a <i>try</i> block</li>
<li>In our <i>try</i> block, check the type of the variable <i>$result</i></li>
<li>If the type is NOT an ErrorRecord, then throw an exception. Use the exception in a <i>catch</i> block immediately after <i>try</i>, wherein we exist from our script</li>
</ol>

<pre>
    $bucketName = "s3://$($Name)"
    # make sure the bucket doesn't already exist, checking for an ErrorRecord
    try {
        $result = aws s3 ls $bucketName 2>&1
        <b>if ($result -eq $null -or $result[0].GetType().Name -ne "ErrorRecord") {
            throw [System.Exception]::new("The specified bucket $bucketName already exists.");
        }</b>
    } catch {
        <b># if the bucket already exists, don't do anything and exit noisily!
        Write-Error $PSItem.Exception.Message
        Exit</b>
    } 
</pre>

<h2>Create the S3 Bucket</h2>

The next AWS CLI command creates our bucket. The constructed <i>bucketName</i> variable is used here. The command is <b>aws s3 mb</b> and here, mb is short form for "make bucket."

The output of this command is also written to standard error, so we make use of redirection to standard out and place those contents in a variable, which we check later for a specific pattern. If the mb operation succeeds, the contents of the variable will contain "make_bucket" in it. Here's how we check for that:

<pre>
    $bucketName = "s3://$($Name)"
    # make sure the bucket doesn't already exist, checking for an ErrorRecord
    try {
        $result = aws s3 ls $bucketName 2>&1
        if ($result -eq $null -or $result[0].GetType().Name -ne "ErrorRecord") {
            throw [System.Exception]::new("The specified bucket $bucketName already exists.");
        }
    } catch {
        # if the bucket already exists, don't do anything and exit noisily!
        Write-Error $PSItem.Exception.Message
        Exit
    } 

    <b>$bucket = aws s3 mb $bucketName 2>&1</b>

    # if your bucket was created successfully, proceed to configure/upload defaults
    if ($bucket | Select-String -Pattern "make_bucket") {
        # configure default minimal viable policy, using Set-Content
            # upload your site contents
    }
</pre>

<h2>Configure a Public Policy for the Bucket</h2>

To configure a policy that makes all the files in our bucket publically readable, we apply the action <i>GetObject</i> for any principal <i>denoted by "\*"</i> on all resources under our bucket, which would end up being the Amazon Resource Name (ARN) <i>arn:aws:s3:::acme.org/*</i> for a hypothetical bucket named acme.org. We use the <a href="https://awspolicygen.s3.amazonaws.com/policygen.html">Amazon Policy Generator</a> to generate such a policy for us. 

<pre>
{
    "Version":"2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadForGetBucketObjects",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::BUCKET_NAME/*"
            ]
        }
    ]
}
</pre>

We paste the resulting policy into a file called <i>static-site-policy.json</i>, in the same directory as our script, that our script will use. The file <i>static-site-policy.json</i> will become our template, and all we need to do to have it work for our new bucket, is replace the constant BUCKET_NAME with the value of the bucket name we passed in as a parameter. At this point, we would have the following as our complete script:

<pre>
    [CmdletBinding()]
    Param(
        [Parameter(Mandatory=$true,Position=1)]
        [String]
        $Name,
        # Name of the path on the local computer to copy files from, assume current working directory by default
        [String]
        $Path = $pwd.Path
    )
    $bucketName = "s3://$($Name)"
    # make sure the bucket doesn't already exist, checking for an ErrorRecord
    try {
        $result = aws s3 ls $bucketName 2>&1
        if ($result -eq $null -or $result[0].GetType().Name -ne "ErrorRecord") {
            throw [System.Exception]::new("The specified bucket $bucketName already exists.");
        }
    } catch {
        # if the bucket already exists, don't do anything and exit noisily!
        Write-Error $PSItem.Exception.Message
        Exit
    } 

    $bucket = aws s3 mb $bucketName 2>&1

    # if your bucket was created successfully, proceed to configure/upload defaults
    if ($bucket | Select-String -Pattern "make_bucket") {
        <b># configure default minimal viable policy, using Set-Content
        (Get-Content .\static-site-policy.json).replace('BUCKET_NAME', $Name) | Set-Content .\policy.json
        if (Test-Path -Path "policy.json") {
            # upload your site contents
            write-host (Get-Content "policy.json")
            aws s3api put-bucket-policy --bucket $Name --policy file://policy.json
            .\Upload-S3Site.ps1 -Name $Name -Path $Path
        }</b>
    }
</pre>

At this point, our complete script can create a bucket if it doesn't exist, using a name we provide, and sets a policy for the bucket (which is really just generated by Powershell's Get-Content and Set-Content commandlets, to replace the BUCKET_NAME value) using the command <b>aws s3api put-bucket-policy</b> which applies the policy in the actual policy file, <i>policy.json.</i>

In the next post for this series, the final piece in our puzzle will be discussed - the <i>Upload-S3Site</i> script, which will upload all of our website's files to the bucket. The <i>Path</i> parameter we defined here will play an important role in this second script. 


Additional Resources
<ul>
<li><a href="https://docs.aws.amazon.com/cli/latest/reference/s3/index.html">AWS CLI s3 Commands</a></li>
<li><a href="https://docs.aws.amazon.com/cli/latest/reference/s3api/index.html">AWS CLI s3api Commands</a></li>
<li><a href="https://docs.microsoft.com/en-us/powershell/scripting/getting-started/fundamental/windows-powershell-integrated-scripting-environment--ise-?view=powershell-6">Powershell Integrated Scripting Environment (ISE)</a></li>
<li><a href="https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content?view=powershell-6">Get-Content</a> and <a href="https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-content?view=powershell-6">Set-Content</a> Commandlets</li>