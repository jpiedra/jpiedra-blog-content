+++
title = "Website Deployment Scripts"
date = 2018-02-18T13:58:41-05:00
link = "https://github.com/jpiedra/S3SiteDeploy"
section1_title = "command line tools"
section1_thumbnails = [
    "/img/tn_ps.png",
    "/img/tn_as.png"
]
panelcolor = "#4E4E7E"
+++

To ease the process of deploying static websites or front-end SPA's to <a class='projectpanel_body_content_link' href="https://aws.amazon.com/s3/">Amazon Web Services' S3 buckets</a>, I wrote some <a class='projectpanel_body_content_link' href="https://docs.microsoft.com/en-us/powershell/">Powershell</a> scripts that handle configuring a new S3 bucket to use for website hosting, as well as uploading content of various standard types (HTML, CSS, JavaScript, etc.) to the bucket afterwards. The scripts use <a class='projectpanel_body_content_link' href="https://aws.amazon.com/cli/">Amazon's Command-Line Interface.</a>