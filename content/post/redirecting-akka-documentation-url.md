+++
tags = ["Scala", "Java", "Akka"]
Description = "This tutorial teaches you how to set redirections for proper Akka version of Akka documentation pages."
date = "2016-04-30T16:40:59+02:00"
menu = "main"
title = "Redirecting Akka Documentation URLs"
keywords = "akka, scala, java, chrome, plugin, redirection, url"
+++


When you search for Akka-related things on Google and you want to display a page on Akka Documentation, two problems might arise:

    1) You are a Scala developer and the Google result is for Java.
    2) You are using version 2.4.2 but the page is for 2.3.13.

or maybe both. You would probably wish a plugin to redirect you to correct page automatically. For example:

    http://doc.akka.io/docs/akka/2.3.13/java/actors.html

to

    http://doc.akka.io/docs/akka/2.4.2/scala/actors.html


You can use [this Chrome plugin](https://chrome.google.com/webstore/detail/redirector/pajiegeliagebegjdhebejdlknciafen/related) to set regex URL redirections.


For Akka version 2.4.2 supporting Scala, add the following regex rules on the settings page of the plugin:

    From: ^http://doc.akka.io/docs/akka/(.*?)/(.*)/(.*)    
    To: http://doc.akka.io/docs/akka/2.4.2/scala/$3


There are similar plugins for Safari but I could see none with regex support.
