xvfb Screenshot Generator
=========================

Background
----------

I built a web application a few years ago called Abetaday which allowed its users to share screen shots of websites instead of “difficult to understand” URLs so that you could “see” where you were headed before you clicked through.

Since then, I’ve received many requests that I explain what I did. Abetaday is asleep these days and so I’ve decided to explain how I created screen shots of websites on-the-fly. To begin, here are some basic requirements:

- [ ] Admin access to your server.
- [ ] Basic knowledge of threading (if using inside a user web application).
- [ ] Intermediate knowledge of Bash (or another Unix shell).
- [ ] Basic knowledge of how Firefox works.

Program Use
-----------

My configuration includes the following programs but you may supplement many with your own preferences:

* Ubuntu Hardy Heron
* Xvfb
* Firefox
* Flash Plugin
* ImageMagick

Install
-------

*ImageMagick*

```sh
wget ftp://ftp.fifi.org/pub/ImageMagick/ImageMagick-6.3.8-0.tar.gz
tar xvzf ImageMagick-6.3.8-0.tar.gz
cd ImageMagick-6.3.8/
./configure
make
sudo make install
sudo apt-get -y install libmagick9-dev
sudo gem install rmagick
```

*Xvfb*

```sh
apt-get install xvfb
apt-get install xorg
cd /etc/init.d
sudo update-rc.d xvfb defaults
```

You should see the following response:

```sh
Adding system startup for /etc/init.d/xvfb ...
/etc/rc0.d/K20xvfb -> ../init.d/xvfb
/etc/rc1.d/K20xvfb -> ../init.d/xvfb
/etc/rc2.d/S20xvfb -> ../init.d/xvfb
/etc/rc3.d/S20xvfb -> ../init.d/xvfb
/etc/rc4.d/S20xvfb -> ../init.d/xvfb
/etc/rc5.d/S20xvfb -> ../init.d/xvfb
/etc/rc6.d/K20xvfb -> ../init.d/xvfb
```

Now, let’s make sure xvfb is ready to run on boot by checking it with sysv-rc-conf. Sysv-rc-conf is a Debian variant (chkconfig being for RedHat variants) that allows you to manage services on your computer. You should check to make sure that xvfb is included by running the application with:

```sh
sudo sysv-rc-conf
```

At this point, you’ll know if you have it installed or not. If sysv-rc-conf isn’t installed, go ahead and get it. If it is installed, you will see a screen that lists the services running on the far left column and then the characters 1, 2, 3, 4, 5, 0, 6, S running across the first row. These characters refer to the runlevel (sets of modes) in which the computer boots:

Mode | Description
1 | Single user or “rescue” or “troubleshooting” mode
2 | Full multi-user mode or “normal use” mode
3 | Full multi-user mode or “normal use” mode
4 | Full multi-user mode or “normal use” mode
5 | Full multi-user mode or “normal use” mode
0 | System halt mode or the state at which the computer shuts down.
6 | System Reboot mode
S | This is a “transition” mode that the system uses when transferring from one mode to another.

Make sure that xvfb is checked for modes 2 to 5, normal multi-user mode.
Also, make the xvfb application is accessible to all with:

```sh
sudo chmod +x xvfb
```
Now, we need to create an executable file that will allow us to easily start, stop, status, and restart the application. Open the xvfb file here:

```sh
cd /etc/init.d
vim xvfb
```

In this file, enter the code found in this project.

Now, you can test your xvfb server with the following commands:

```sh
sudo /etc/init.d/xvfb start
sudo /etc/init.d/xvfb stop
```

*Firefox*

```sh
sudo apt-get install firefox
```

I used Firefox 3.0.11.

We now configure the Firefox javascript page because we don’t want Firefox to load a website upon booting (because it will most likely not be the one we want and therefore inefficient) and we do not want Firefox to restore websites upon booting after a crash (because one request is made at a time and recovering websites from a crash will result in unexpected websites being returned). Navigate to your Firefox folder:

```sh
cd /usr/lib/firefox-3.0.11/defaults/preferences
sudo nano firefox.js
```

In this file, enter the code found in this project.

*Flash Plugin*

To begin, we need to update our sources.list file in order to get this plugin, at least in Hardy Heron distributions:

```sh
sudo nano /etc/apt/sources.list
```

In this file, enter the code found in this project.

```sh
sudo apt-get update
sudo apt-get install flashplugin-nonfree
```

Take Screenshots
----------------

Load some websites inside Firefox running on xvfb. This is the standard command for loading firefox in xvfb with a specified website from a terminal command line:

```sh
DISPLAY=:1 firefox http://www.gregorymazurek.com
```

It doesn’t matter what number you use for DISPLAY=:. If you want to use this within Python, PHP, or Ruby, you will have to make adjustments to the code. For example, in Ruby On Rails, you will need to execute the following command instead:

```sh
`env DISPLAY=:1 firefox http://www.gregorymazurek.com`
```

Here, the ` tick marks allow you to execute terminal command line code from within Ruby On Rails. The env is added to make the server computer be the environment for the DISPLAY command. And finally, if you decide to run this code within Ruby On Rails, you are going to have to wait until everything is executed before Ruby On Rails can continue executing more code. Abetaday didn’t do this. As soon as a user entered a URL, the server ran through its process while the user went on her way, not having to wait until it was finished. In order to do this, I created a new thread within Ruby On Rails:

```sh
Thread.new {`env DISPLAY=:1 firefox http://www.gregorymazurek.com`}
```

Use Linux comands to generate a screenshot of the website you just loaded. Here, the display number that you used earlier is important because we will refer to it in this standard terminal command line code:

```sh
DISPLAY=:1 import -window root -crop 978x597+0+95 -quality 90 /home/folder_to_store_data
```

Here, we are calling DISPLAY=:1 and then using Linux’s import tool to capture the window. Then, we are calling “-window root” in order to specify that we are interested in the entire screen. Then, we are cropping the screen to get rid of borders and the firefox header. Then, we set the quality to 90 and finally specify where we would like to store the end result. Now, you have your first screen shot. If you would like to subsequently change the size of the file, use the following command:

```sh
convert /home/folder_to_store_data/original.jpg -resize 200x122 /home/folder_to_store_data/resized.jpg
```

Conclusion
----------

That’s it. In Abetaday, I did a few more things like maximize the number of screen shots generated (minimizing the queue), logging events, starting/terminating firefox depending on problems, filtering the URL to make sure it was valid/safe, and more. But if you just want the skeleton for generating screen shots on the server, I hope this helps.

License
-------

MIT