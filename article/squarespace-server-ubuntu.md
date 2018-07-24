+++
date = "2017-01-30T18:11:31-05:00"
keywords = [ "jpiedra", "squarespace", "squarespace-server", "ubuntu" ]
title = "Installing squarespace-server on Ubuntu 14.04 LTS"
tags = [ "linux", "squarespace", "design", "troubleshooting" ]
+++

This article details how to install the <a href="http://developers.squarespace.com/local-development/">Squarespace local development server</a> on Ubuntu 14.04 LTS.

I recently took on a small project, some work on customizing Squarespace templates. In the course of doing this, I required the Squarespace local development server in order to quickly modify and test changes to the template. I use a VirtualBox VM I built some time ago using Vagrant, based on Ubuntu 14.04 LTS.

If you've encountered problems installing this program on the aforemention Linux distro, read on and follow these steps, as they ended up working for me very well!

<!--more-->
<h2>Potential Errors</h2>

After following the instructions on Squarespace's official website (link above) you might have come across the following error:

<pre>
$ npm install -g @squarespace/server
@squarespace/server@1.0.7 copyrunscript /usr/lib/node_modules/.staging/@squarespace/server-b43b0a9f
node scripts/copyfile.js build/distributions/local-developer/bin/run.bat darwin:build/distributions/local-developer/bin/osx-local-developer win32:build/distributions/local-developer/bin/win-local-developer.bat linux:build/distributions/local-developer/bin/linux-local-developer build/distributions/local-developer/bin/local-developer

copying build/distributions/local-developer/bin/linux-local-developer to build/distributions/local-developer/bin/run.bat
npm ERR! Linux 4.2.0-27-generic
npm ERR! argv "/usr/bin/nodejs" "/usr/bin/npm" "install" "-g" "@squarespace/server"
npm ERR! node v6.9.4
npm ERR! npm  v3.10.10
npm ERR! path ../lib/node_modules/@squarespace/server/build/distributions/local-developer/bin/run.bat
npm ERR! code EACCES
npm ERR! errno -13
npm ERR! syscall symlink

npm ERR! Error: EACCES: permission denied, symlink '../lib/node_modules/@squarespace/server/build/distributions/local-developer/bin/run.bat' -> '/usr/bin/squarespace-server'
npm ERR!     at Error (native)
npm ERR!  { Error: EACCES: permission denied, symlink '../lib/node_modules/@squarespace/server/build/distributions/local-developer/bin/run.bat' -> '/usr/bin/squarespace-server'
npm ERR!     at Error (native)
npm ERR!   errno: -13,
npm ERR!   code: 'EACCES',
npm ERR!   syscall: 'symlink',
npm ERR!   path: '../lib/node_modules/@squarespace/server/build/distributions/local-developer/bin/run.bat',
npm ERR!   dest: '/usr/bin/squarespace-server' }
npm ERR! 
</pre>

This may occur if you attempt to install the program as a <i>user with insufficient permissions.</i> Installing as root may resolve the problem, as well as <a href="https://docs.npmjs.com/getting-started/fixing-npm-permissions">providing the user NPM is running under with the required permissions.</a>

However, even following that step might not end matters for you; the next error might come up:

<pre>
$ sudo npm install -g @squarespace/server
npm WARN lifecycle @squarespace/server@1.0.7~preinstall: cannot run in wd %s %s (wd=%s) @squarespace/server@1.0.7 npm run accept && node scripts/execif.js --exists=build npm run copyrunscript /usr/lib/node_modules/.staging/@squarespace/server-b43b0a9f
npm ERR! Linux 4.2.0-27-generic
npm ERR! argv "/usr/bin/nodejs" "/usr/bin/npm" "install" "-g" "@squarespace/server"
npm ERR! node v6.9.4
npm ERR! npm  v3.10.10
npm ERR! path /usr/lib/node_modules/@squarespace/server/build/distributions/local-developer/bin/run.bat
npm ERR! code ENOENT
npm ERR! errno -2
npm ERR! syscall chmod

npm ERR! enoent ENOENT: no such file or directory, chmod '/usr/lib/node_modules/@squarespace/server/build/distributions/local-developer/bin/run.bat'
npm ERR! enoent ENOENT: no such file or directory, chmod '/usr/lib/node_modules/@squarespace/server/build/distributions/local-developer/bin/run.bat'
npm ERR! enoent This is most likely not a problem with npm itself
npm ERR! enoent and is related to npm not being able to find a file.
npm ERR! enoent 
</pre>

The error messages may seem a bit opaque, but it appears that a required folder either wasn't created or found during the installation process. 

<h2>A workaround: install "locally," then globally</h2>
I ended up having to rely on a workaround. When I say "locally" I really mean, first installing the squarespace-server dependency for a specific node project, then <i>moving that installation</i> over to the location it must reside at in order to be used globally.

Installing squarespace-server for a specific project is easy enough, just omit the global flag (-g) when you install it to your site template's project directory. The part that might be tricky for some is "moving" the files over, as it involves not only moving the files over to a new directory, but applying permissions and creating a symbolic link as well. The presence of the symbolic link is what will allow us to launch squarespace-server globally. The steps are as follows:
<ol>
	<li>Clone the files for your Squarespace site, as instructed here: http://developers.squarespace.com/local-development</li>
	<li>Change into the 'template' directory created after cloning</li>
	<li>Install the squarespace-server package normally (not globally)
	<pre>$ npm install @squarespace/server</pre>
  	</li>
	<li>You should now have a directory:<b>template/node_modules/@squarespace</b>
	</li>
	<li>Copy the files for this module into your system-wide 'node_modules' directory:
	<pre>$ sudo cp -rp node_modules/@squarespace /usr/lib/node_modules/</pre>
	</li>
	<li>Change the permissions for this copied folder as needed. This location will vary depending on your operating system (for our purposes, we're using a Vagrant Box based on Ubuntu 14.04, thus the user shown below:
  	<pre>$ sudo chown -R vagrant:root @squarespace
$ sudo chmod -R 755 @squarespace</pre>
	</li>
	<li>Finally, create a symbolic link, allowing us to run 'squarespace-server' from anywhere:<pre>$ sudo ln -s /usr/lib/node_modules/@squarespace/server/build/distributions/local-developer/bin/run.bat /usr/bin/squarespace-server</pre></li>
</ol>

You should now be able to use the local development server, from inside your site's 'template' directory:
<pre>$ squarespace-server https://[sitename].squarespace.com --auth</pre>

Hopefully this solution works for you. Share in the comments if this helped you get squarespace-server running!
