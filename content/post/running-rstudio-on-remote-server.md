+++
tags = ["installation", "R"]
Description = ""
date = "2015-11-23T16:14:21+02:00"
menu = "main"
title = "Running RStudio on Remote Server"
Summary = "In this tutorial, we are going to build a Remote R Server."
+++


In this tutorial, we are going to build a Remote R Server. Following this tutorial, you will be able to run your R codes on a remote server so that it runs more efficiently or it doesn't heat up your laptop. You can read the [detailed benefits](https://support.rstudio.com/hc/en-us/articles/200552306-Getting-Started). You can use any remote server for this purpose. You can use AWS, DigitalOcean, Scaleway.

**Warning:** Scaleway has currently ARM processors thus require compilation from source code. Also some R packages are not compatible with ARM processors.

See [RStudio server installation page](https://www.rstudio.com/products/rstudio/download-server/) for up-to-date version.

On your server install R-base:

    $ sudo apt-get install r-base

Then we will install RStudio server. If you processor is 64 bit:

    $ sudo apt-get install gdebi-core
    $ wget https://download2.rstudio.org/rstudio-server-0.99.486-amd64.deb
    $ sudo gdebi rstudio-server-0.99.486-amd64.deb

If it's 32 bit:

    $ sudo apt-get install gdebi-core
    $ wget https://download2.rstudio.org/rstudio-server-0.99.486-i386.deb
    $ sudo gdebi rstudio-server-0.99.486-i386.deb

If you are using Scaleway, your processor is ARM. So you have to compile RServer on your own. See the [build script](https://github.com/jrowen/ARM-rstudio-server) for this purpose. This script was prepared for Chromebook, so you don't have to follow the guideline provided there.

    $ git clone https://github.com/jrowen/ARM-rstudio-server
    $ cd ARM-rstudio-server
    $ sh build_rstudio.sh

This can take a while (maybe an hour). You can use nohup or  [tmux](http://askubuntu.com/questions/8653/how-to-keep-processes-running-after-ending-ssh-session) to avoid losing your session.


Check if rstudio-server.service file exists. If it doesn't, copy it:

    $ ls /etc/systemd/system/rstudio-server.service
    $ cp src/cpp/server/extras/systemd/rstudio-server.service /etc/systemd/system/

Now we can start rstudio-server service with:

    $ service rstudio-server start
    $ service --status-all

You should see a + sign in front of rstudio-server. In order to login the web panel, you have to create a custom user.

    $ adduser michael

Answer the questions and now you can access your web panel via http://<server-ip>:8787 with your user michael. In this panel, you can use web-based RStudio.


Some requirements could be Pandoc for Knitting HTML, MySQL Client for RMySQL:

    $ sudo apt-get install pandoc
    $ libmysqlclient-dev

Nevertheless, you may encounter some packages (like caret) being missing due to ARM incompatibility.
