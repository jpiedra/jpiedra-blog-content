+++
title = "A simple build script"
date = "2017-02-01T10:51:38-05:00"
keywords = [ "jpiedra", "linux", "scripting", "bash", "git" ]
tags = [ "linux", "scripting", "bash", "git", "part 1" ]
+++

Today we go over the process of writing a very simple Bash script. We'll be able to detect whether a build for a <a href="https://gohugo.io/overview/quickstart/">Hugo static site</a> is present, push the files to a Git repository for our website, and automate all the steps involved.

<!--more-->
<h2>Version 1</h2>

I'll begin by showing the code, then explain my process: 

<pre>
#!/bin/bash

build_dir=/var/www/html/blog-dev/public
deploy_dir=/var/www/html/blog-live/jpiedra.github.io

if [ -d $build_dir ]
then
  echo "BUILD FOUND, DETAILS:"
  echo $(stat $build_dir)
  cd $deploy_dir && rm -rf *
  cp -rp $build_dir/* $deploy_dir
  cd $deploy_dir && git status; git add -A; git commit -m "[Scripted Deploy $(date)]"; git push
else
  echo "NO BUILD FOUND"
  echo "  Run 'hugo' to "
  echo "  create latest build"
fi
</pre>

We begin by setting two variables, <b>build_dir</b> and <b>deploy_dir.</b> These store string values indicating the locations of where our built Hugo site's files reside, and the directory where those files should be copied to, respectively. In this case, <b>deploy_dir</b> is a GitHub Pages repository, the files used therein are the same ones responsible for powering this website!

<h2>Testing for Build Directory</h2>

The bulk of this script begins where the <i>if.. ..fi</i> structure is shown. This conditional structure begins with a <a href="http://linuxcommand.org/lc3_man_pages/testh.html">test.</a> Now, because we're writing a Bash script, rather than using the command line, we have a built-in operator available to us, the square brackets [ ]. These are used to perform any test in a Bash script, thus instead of writing <i>test -d /some/directory</i> we can instead write <i>[ -d /some/directory ]</i>.

The -d switch allows us to test whether the provided argument is a directory. The test returns 0 or 1, true and false respectively, depending on whether the provided argument is detected as a valid directory. In this script, we test the value stored in <b>$build_dir</b> (thus, the dollar sign) to check whether a build folder exists in our Hugo site directory - which, by default, would be called <i>public</i> and reside in the Hugo site base directory.

<i>Tip: you can run the command <b>compgen -b</b> with the provided -b switch to get a list of all Bash built-ins available to you. Running the command in your terminal, displays the [ ] built-in near the top of the list.</i>

<h2>Conditional Execution</h2>

In the event our test for a directory fails (directory doesn't exist), we simply echo out a diagnostic message (which would best be output to standard error, more on that later). After <i>else</i> our script prints the messages to stdout, so whatever is echo'ed is visible in the terminal. 

If the build directory does exist, we can begin performing the required steps to push our files to the GitHub pages directory, indicated by <b>$deploy_dir.</b> We'll be making repeated use of a central Bash feature here.

<h2>Command Substitution</h2>

<a href="http://tldp.org/LDP/abs/html/commandsub.html">Command substitution</a> "literally plugs... command output into another context," as the aforementioned page phrases it. Indeed, by using certain operators built into the Bash language, we can get the output of a command and do with it whatever we want - this usually means assinging said output to a variable, parsing it using some other commands, or even saving it to another file.

We'll use command substitution twice here, as indicated by <i>$(...)</i> where the elipses would be replaced by Bash built-ins or Linux commands. Another form that's available to us uses backticks, but we'll stick with the former operator instead.

<h2>Deployment Process</h2>

After echo'ing that we've found the build folder, we run a few commands when the build directory is detected.

<ol>
	<li>Command substitution is used to capture the output of running <i>stat $build_dir</i>, thus printing the result of that command to stdout using echo</li>
	<li>We change into the deployment directory, and remove the files present there, making way for the files from our latest Hugo site build <i>(cd $deploy_dir && rm -rf *)</i></li>
	<li>The contents of the Hugo site build directory are copied into the deployment directory <i>(cp -rp $build_dir/* $deploy_dir)</i></li>
	<li>Finally, we change into the deployment directory and execute a few standard Git commands, that add/commit/push the new files to the current branch</li>
</ol>

A few notes on step 4. The <i>&&</i> operator ensures that the second command, <i>git status</i> is only executed if <i>cd</i> is successful. This is seen elsewhere, in step 2, ensuring we remove the files only if we successfully changed to the desired folder. Also in step 4, we see the final example of command substitution used when we indicate a message for <i>git commit</i> to use. We capture the output of running the Bash built-in <i>time</i> and use that as part of the content for our commit message.

When you run this script, the last part of this deployment process would involve running <i>git push,</i> so you would be expected to provide your credentials in the terminal to push the latest Hugo site files. In summation, having a script such as this can save precious time, by performing various checks for us along the way as well as eliminating the tedium of typing and executing several commands.