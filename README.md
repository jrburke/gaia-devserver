# gaia-devserver

A node-based web server for the [Gaia](https://wiki.mozilla.org/Gaia) user
interface that runs on [Boot to Gecko](https://wiki.mozilla.org/B2G) (B2G).

## Why is this needed? <a name="why"></a>

B2G consists of three levels:

* Gonk: a Linux-based hardware abstraction layer for the physical mobile device.
* Gecko: The web renderer used in Firefox that runs on top of Gonk.
* Gaia: a collection of web apps that run in Gecko. They provide some default
  apps like a home screen and dialer.

All apps on B2G are web apps. That means they are served from an URL. This is
wicked awesome. They can run offline by using technologies like AppCache and
IndexedDB.

Gaia is served from a domain, **gaiamobile.org**. Normally this domain is
mapped to a local host when doing development.

This devserver will allow running the Gaia UI from your local machine, so that
you can play with the Gaia UI in a Nightly Firefox build. This makes it super
fast to develop apps since you have a fully array of developer tools available
to you and your "modify/reload" dev path is faster.

The downside is that some things are best tested on a are real device/phone.
However, for trying out web app development, registering as a web app, and
trying out some of the new device APIs, this is a nice, low-cost way to get
started.

The Gaia page has instructions on how to
[modify an Apache web server](https://wiki.mozilla.org/Gaia/Hacking#Hosting_Gaia_using_Apache)
to serve Gaia. However, I prefer to use Node to spin up a focused server so that
I do not have to muck with a local Apache installation, and I do not have to do
the work as root. Also, since I like using
[Persona/BrowserID](http://www.mozilla.org/en-US/persona/) in Node apps, I
will be using Node for the back-end of my web apps.

**Remember**, B2G and Gaia are still **extremely** new, and still actively
developed. Life on the edge can be difficult, but you get access to the latest
developments. An analogy from the iOS world: think of of it as getting access
to iOS a year before Apple shipped it in the first iPhone.

## Prerequisites

* Node
* You have some developer tools installed, like make and git.
* You are using a Linux/OS X type of OS. These instructions were run on OS X.
  You can probably get this to work on Windows too, but probably has a few
  different steps (pull request that adds those instructions welcome)
* Download the [Firefox Nightly](http://nightly.mozilla.org/).
  Install in /Applications on OS X.

Firefox Nightly will receive the latest updates that allow the B2G APIs to work.
On the flip side, it can update every day, and very occasionally the nightly
build can have problems that make it hard to use.

## Setup

These instructions assume you start the work in your home directory. Adjust
the paths accordingly if you do the installation somewhere else. These
instructions were run on OS X, but should be roughly translatable to other OSes.

First, modify your /etc/hosts file to have an entry to map gaiamobile.org to
your local host:

    127.0.0.1     gaiamobile.org

Then, in your home directory:

    mkdir b2g
    cd b2g
    git clone git://github.com/andreasgal/gaia.git
    git clone git://github.com/jrburke/gaia-devserver.git
    cd gaia
    GAIA_DOMAIN=gaiamobile.org:8424 make
    cd ..
    node gaia-devserver/devserver.js docRoot=gaia/apps

Then in a separate terminal window, launch Firefox Nightly using the special
profile what was created by the gaia make command:

    /Applications/FirefoxNightly.app/Contents/MacOS/firefox -profile /YOUR/PATH/TO/gaia/profile --no-remote http://homescreen.gaiamobile.org:8424

Where /YOUR/PATH/TO/gaia/profile is the absolute path to the gaia/profile
directory.

This profile is nice to use since it is different from your main Firefox
profile, and it has the special configuration that allows some of gaia apps
special API access.

Once Firefox Nightly starts up, then do the following:

    * Open a new tab and go to `about:config`.
    * Search for `browser.offline-apps.notify` and set it to **false**. This
      avoids the prompts that Nightly would show for each app that uses appcache.
    * Create a **new** config value:
        * Right-click and select the `New, Boolean` option
        * For the name, use `dom.w3c_touch_events.enabled`
        * For the value set to `true`

## Staying up-to-date

The code changes frequently. Before filing issues, be sure to have the latest
code:

### Firefox Nightly

If you open the **About Nightly** menu option, it will do a check to see if you
have the latest Nightly.

### Gaia

Close down Firefox Nightly first, then from inside the gaia directory:

    cd gaia
    git pull origin master
    GAIA_DOMAIN=gaiamobile.org:8424 make

Reopen Firefox Nightly. You may also then want to clear the browser caches used
in Nightly. Go to the `Tools, Clear Recent History` menu item, then:

    * Select "Everything" for the "Time range to clear"
    * In the Details section, check all the checkboxes
    * Click "Clear Now".

If you delete the `gaia/profile` dir, then see the <a href="#setup">Setup</a>
section above to redo the preference changes in Nightly.

### This server

    cd gaia-devserver
    git pull origin master
