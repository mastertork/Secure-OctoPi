# Secure-OctoPi

Many 3D printing enthusiasts turn to the open source software [Octoprint](http://octoprint.org/) to run their 3D printer so they do not have to dedicate an entire computer to run thier printer. Octoprint convieniently comes with its own Raspberry Pi ready image called [Octopi](http://octoprint.org/download/).  Of course once Octopi is set up the typical next thought is that you would like to be able to access your Octopi from wherever you are over the internet.  The Octoprint FAQ section warns that it [does not have sufficent security](https://github.com/foosel/OctoPrint/wiki/FAQ#i-want-to-access-my-octoprint-installation-from-the-internet-but-i-only-want-people-i-know-to-be-able-to-see-anything) to be pointed internet facing by itself.  Octoprint instead reccommends that you use HAProxy as a reverse proxy to provide authentication and security for Octoprint.  In this [tutorial](https://github.com/mastertork/Secure-OctoPi/wiki/Secure-OctoPi-Wiki) I will show you how to configure HAProxy to be a bit more secure than most tutorials you will find on the web.

### [Read my full tutorial](https://github.com/mastertork/Secure-OctoPi/wiki/Secure-OctoPi-Wiki) in the [Wiki](https://github.com/mastertork/Secure-OctoPi/wiki/Secure-OctoPi-Wiki) if you need a step by step walkthrough to get this running.

## Quick and Dirty
If you know what you're doing, just take my [haproxy.cfg](/haproxy.cfg) and copy it to your Pi in the `/etc/haproxy/haproxy.cfg` folder.
Edit the last line of the config file for your `[user]` and `[password]`.

You will also need to create a directory `/var/empty` and `chmod 0` it to jail haproxy once it's finished loading.
