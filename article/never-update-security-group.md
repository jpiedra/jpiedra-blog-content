+++
title = "Never Update an AWS Security Group Again"
date = 2018-07-08T13:58:48-04:00
keywords = [ "jpiedra", "scripting", "aws", "python", "ec2" ]
tags = [ "scripting", "aws", "python", "ec2" ]
draft = false
+++

Sweat the small stuff! Script that itch you can't scratch...

<!--more-->
<h2>A Moving Target</h2>
If you're someone with a personal cloud platform account, using it to put together projects, demonstrations, or really anything on it from a home computer, odds are good that you're probably not accessing said platform from a static IP address. 

In the case of Amazon Web Services, access to Elastic Cloud Compute (EC2) instances is controlled through the configuration of <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html">security groups.</a> You configure a security group with one or more ingress and egress traffic rules, indicating what traffic is allowed over which ports, from which IP addresses or subnets. Those security groups are applied to EC2 instances, and that's how traffic to, from, or between you, those instances, and other AWS resources gets defined. 

You can set up a security group initially to allow access - say, over port 22 for SSH sessions - from your home device. At the moment, AWS will conveniently determine your device's external IP address, and set up the access you desire. 

This helps in the near-term, but needs to be updated as soon as your internet service provider (ISP) changes your IP address. While spending all of 5-10 minutes accessing your AWS console to manually update security groups may not sound so bad, it would be nice if an automated solution to that problem was possible.

<h2>Enter Boto3</h2>
<a href="https://boto3.readthedocs.io/en/latest/guide/quickstart.html">Boto</a> is, in their words, "the Amazon Web Services (AWS) SDK for Python." You can use Boto to write Python code to interact directly with the various components of the AWS platform: RDS, S3, EC2, among others. Scripted access to security groups is folded under the EC2 service, so you can use Boto to write Python code that <a href="https://boto3.readthedocs.io/en/latest/reference/services/ec2.html#securitygroup">instantiates a client</a> to interact with the <i>ec2</i> service and use that same client to configure your security groups as desired. 

<h2>Assumptions</h2>
To use Boto, you need to follow the installation instructions defined in the link above to install the Python library. You also need to make sure you have access to AWS <a href="https://boto3.readthedocs.io/en/latest/guide/quickstart.html#configuration">configured using your access key and secret token,</a> both of these generated for an account that has access to view and update security groups. For more information on configuring an account to modify EC2 resources and security groups, <a href="https://docs.aws.amazon.com/AWSEC2/latest/APIReference/ec2-api-permissions.html">please see this article.</a>

<h2>Instantiating an EC2 Client</h2>
Let's get right to coding and set up the bare minimum we need to view existing security groups:
<pre>
import sys, json, requests
try:
    import boto3
    from botocore.exceptions import ClientError
except ImportError:
    sys.exit("Could not import 'boto3', please install this package.")
if(len(sys.argv) != 2):
    <b>sys.exit("USAGE: python ipme.py [security-group-id]")</b>
else:
    group_id = str(sys.argv[1])
<b>
client = boto3.client('ec2')
</b>
try:<b>
    response = client.describe_security_groups(
        GroupIds=[
            group_id,
        ],
        DryRun=False
    )</b>
except ClientError as e:
    sys.exit(e)
except Exception as ex:
    sys.exit(ex)
else:
    if response['SecurityGroups']:
        info = response['SecurityGroups'][0]
        old_access = response['SecurityGroups'][0]['IpPermissions'][0]
        print('Found security group (id={0[GroupId]}) named {0[GroupName]}'.format(info))
    else:
        sys.exit('Nothing found matching that criteria (SSH rule).')
</pre>

The former outlines the start of a script that accepts a single parameter, the <i>group id</i> for the security group you'd like to update regularly. For our purposes, we would work with whatever security group was configured already, using the home IP address at the time it was configured, and pass the group ID of that security group into our script. Let's work with a security group with ID <b>sg-6d3c3c26</b> that configures SSH access from <b>1.2.3.4</b>.

<h2>Using Filtering</h2>
One useful feature baked into Boto is the ability to <a href="https://boto3.readthedocs.io/en/latest/reference/services/ec2.html#EC2.Client.describe_security_groups">filter out the objects you want returned from your programmatic calls to AWS.</a> We can use this to make sure we're <i>only</i> ever getting back security groups that actually have an SSH ingress traffic rule configured. After all, the point of our script will be to update an existing SSH security group regularly, making sure the IP is current. We wouldn't want to add ingress SSH traffic to a security group that didn't have it already. 

<pre>
...
# find security group by that id
try:
    response = client.describe_security_groups(
        <b>Filters=[
            # only concerned with ssh
            {
                'Name': 'ip-permission.from-port',
                'Values': [ '22' ]
            },
            {
                'Name': 'ip-permission.to-port',
                'Values': [ '22' ]
            },
            {
                'Name': 'ip-permission.protocol',
                'Values': [ 'tcp' ]
            },
        ],</b>
        GroupIds=[
            group_id,
        ],
        DryRun=False
    )
# throw error or exception, whatever comes first
except ClientError as e:
    sys.exit(e)
except Exception as ex:
    sys.exit(ex)
else:
    if response['SecurityGroups']:
        info = response['SecurityGroups'][0]
        <b>old_access = response['SecurityGroups'][0]['IpPermissions'][0]</b>
        print('Found security group (id={0[GroupId]}) named {0[GroupName]}'.format(info))
    else:
        sys.exit('Nothing found matching that criteria (SSH rule).')
</pre>

This ensures that, even if a security group ID for a group that does not define traffic over port 22 is passed in, no action will be taken, as the call to AWS will fail to find a security group matching the criteria defined in the <i>Filters</i> argument.

<h2>Updating Our Security Group</h2> 
At this point in our script, we can retrieve an existing security group, by ID, that already has ingress traffic over port 22 defined.

<a href="https://imgur.com/W1cdQ5Q"><img src="https://i.imgur.com/W1cdQ5Q.gif" title="source: imgur.com" /></a>

At this point, we have two tasks left. Rather than update the existing ingress traffic rule in place, it's easier to just revoke that ingress traffic and then add a new rule later, with the same from- and to-port, protocol, and our updated IP address. We're thinking ahead here, and storing the object representing the current ingress rule from our response in a variable <i>old_access</i>. We'll use that object in a call soon to remove access over port 22 from our old IP address.

Here's the code we'll use to determine our new external IP address, revoke access over port 22 from our old IP address, and authorize access over port 22 with the new one:
<pre>
...
<b>ext_ip = requests.get('https://checkip.amazonaws.com/')</b>
ext_ip = ext_ip.text.strip() + '/32'
try:
    data = <b>client.revoke_security_group_ingress</b>(
        GroupId=group_id,
        IpPermissions=[
            old_access
        ]
    )
    data = <b>client.authorize_security_group_ingress</b>(
        GroupId=group_id,
        IpPermissions=[
            {
                <b>'IpProtocol': 'tcp',
                'FromPort': 22,
                'ToPort': 22,
                'IpRanges': [{'CidrIp': ext_ip}]</b>
            }    
        ]
    )
except ClientError as e:
    sys.exit(e)
except Exception as ex:
    sys.exit(ex)
else:
    <b>print('Ingress Successfully Set %s' % data)</b>
</pre>

All told, our script can find the security group we defined, and update it with our current external IP address. Note the first octet of the <i>Source IP</i> address updating:

<a href="https://imgur.com/rRWYH7q"><img src="https://i.imgur.com/rRWYH7q.gif" title="source: imgur.com" /></a>

The final version of this script can be found here: 
<script src="https://gist.github.com/jpiedra/7cd4de8617bb968029b85e0b0617f4b6.js"></script>