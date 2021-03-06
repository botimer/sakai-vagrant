     __       _         _                                    _
    / _\ __ _| | ____ _(_) /\   /\__ _  __ _ _ __ __ _ _ __ | |_
    \ \ / _` | |/ / _` | | \ \ / / _` |/ _` | '__/ _` | '_ \| __|
    _\ \ (_| |   < (_| | |  \ V / (_| | (_| | | | (_| | | | | |_
    \__/\__,_|_|\_\__,_|_|   \_/ \__,_|\__, |_|  \__,_|_| |_|\__|
                                       |___/
    gem install vagrant
    git clone git://github.com/zathomas/sakai-vagrant.git
    cd sakai-vagrant
    git submodule update --init
    vagrant up

Vagrant is a tool for quickly setting up virtual machines for developers. Vagrant has a couple of dependencies: these instructions assume you have ruby, rubygems, git, and the latest version of Oracle’s [VirtualBox](https://www.virtualbox.org/wiki/Downloads). Note: depending on where rubygems is installing things for you, you may need to use `sudo` when running `gem install`, e.g. `sudo gem install vagrant`

The very first time you start this VM, it’s going to download a base Ubuntu server image, install the necessary tools and then checkout the Sakai source code. Once it’s underway, this is a good time to go get lunch. The source code is close to 500MiB, and Subversion takes its sweet time. You can check on how it's doing from time to time by opening a second terminal and measuring the size of the source directory, like this: `du -hs sakai-vagrant/sakai-src`

When it’s finished, you can connect to the new VM like this:

    vagrant ssh

## What’s Included
The system you’ll end up with is very close to what’s described in the [developer setup documentation](https://confluence.sakaiproject.org/display/BOOT/Development+Environment+Setup+Walkthrough) from the Sakai Project. Specifically, you get:

* Java (JDK)
* Subversion for version control
* Maven for build, test, and deploy
* Tomcat for the runtime container
* MySQL for the database backend
* vim, because it’s good to have an editor handy
* Everything else that usually comes with an Ubuntu 12.04 server

One thing that I _didn’t_ bother to include is Eclipse and the extras that you’d use with it, because Vagrant creates headless machines, and any IDE that you decide to use will still be run from your host machine.

All the initial configuration is done for you. The various files that contribute to the config are in `sakai-vagrant/files`.

* `setenv.sh` establishes the JVM options for Tomcat
* `maven.sh` sets a couple of important options for Maven
* `settings.xml` is some optional Maven configuration
* `sakai.properties` is most of the configuration that Sakai uses
* `catalina.properties` tells Tomcat 7 how to load Sakai classes

You can make any tweaks you want to the config files, then run this command to refresh your VM:

    vagrant reload

Vagrant (and puppet, its little helper) is smart enough not to re-do anything that doesn’t need it, so `vagrant reload` is much faster than the initial setup. 

Vagrant creates a fileshare of `sakai-vagrant` for use by the virtual machine, where it’s mounted at `/vagrant`. As mentioned above, your Sakai source code will be in `sakai-vagrant/sakai-src`, which means when you’re logged into the VM, you can get to the source code via `/vagrant/sakai-src`.

## Build and Run
Once the VM is running, it’s ready to build and deploy Sakai. Assuming you’ve already logged in with `vagrant ssh`:

    cd /vagrant/sakai-src
    mvn clean install sakai:deploy
    sudo service tomcat7 start

>ALERT: I have been getting test failures with the 2.9.1 source code. If that’s your experience too, you may need to disable the tests in order to get the build to finish. `mvn clean install sakai:deploy -Dmaven.test.skip=true`.

It uses `/vagrant/maven-repo` as the local Maven repository. The reason for this is that you should be able to blow away the VM at any time without having to download everything again (All 864MiB of it). That’s the same reason we checkout the source code to `/vagrant/sakai-src`.

Once Tomcat is running, you can access Sakai on the _host_ machine at http://localhost:8888/portal This works because the VM is configured to forward port 8888 on the host machine to port 8080 on the VM. Likewise, you can attach a debugger to `localhost:9999` because port 9999 is being forwarded to the debugger listener in Tomcat on the VM.

## Etc.
I used the tomcat7 package for Ubuntu, which breaks Sakai out of the box, but I adjusted the setup so as to un-break it. Here are the locations you’ll want to know about:

    /usr/share/tomcat7 # Tomcat’s bin and lib directories
    /var/lib/tomcat7   # webapps, components, logs, the rest
    
## Motivation
We spend a lot of time setting up servers, tools, various environments. Once you’ve gotten everything just the way you like it, how easy is it to do it all again? We now have the means to describe in a precise, machine-readable form what our machine should look like. If we need to make a tweak, we tweak the specification, not the machine.

Anything that can be automated saves human beings from having to do it.