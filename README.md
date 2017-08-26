# Secure-HAProxy
A more secure configuration for HAProxy when setting up your Octoprint instance on a Raspberry Pi.

# Getting Started
Before pointing your Raspberry Pi internet facing, lets make sure it is up to date and secure.
```
sudo apt update && apt upgrade
```
Once your Pi is up to date ensure that HAProxy is installed. 
**Note: The OctoPi image comes with HAProxy pre-installed.**
```
sudo apt install haproxy
```
# Create a Jail for HAProxy
In my [haproxy.cfg](/haproxy.cfg) file it tells haproxy to drop its privleges and jail itself to an empty directory without any access permissions.  This is for the unlikely event that someone discovers an exploit in HAProxy and manages to break into your system.  THey will have no privleges and no access to any system files.  The safer the better :)

Make your directory and `chmod 0` to remove permissions.  If this failed you `echo` will print it on your screen.  You may need to append `sudo` before the command if this fails.
```
mkdir /var/empty && chmod 0 /var/empty || echo "Failed"
```

# Edit or Replace the Config File
There are a number of imporvements made in this config file
  +uses port `443` for traffic
  +incorperates HTTPS and SSL
  +jails HAProxy in a chroot with no privledges or access to files
  +forces authentication from all internet bound traffic before reaching your Octoprint control
  +optionally you can open port `9080` to allow anonomys viewers to your webcam
  +webcam stream can be found at <your-ip-address>/webcam

The HAProxy config file can be found at `/etc/haproxy/haproxy.cfg`.  Edit the config file with nano.
```
sudo nano /etc/haproxy/haproxy.cfg
```
At this point you can copy and paste the [haproxy.cfg](/haproxy.cfg) I provided.
```
global
        maxconn 4096
        user haproxy
        group haproxy
        chroot /var/empty
        tune.ssl.default-dh-param 1024
        log 127.0.0.1 local1 debug

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        retries 3
        option redispatch
        option http-server-close
        option forwardfor
        maxconn 2000
        timeout connect 5s
        timeout client  15min
        timeout server  15min

frontend public
        bind *:80
        option forwardfor except 127.0.0.1
        use_backend webcam if { path_beg /webcam/ }
        default_backend octoprint
        errorfile 503 /etc/haproxy/errors/503-no-octoprint.http

frontend internet
        bind 0.0.0.0:443 ssl crt /etc/ssl/snakeoil.pem
        option forwardfor except 127.0.0.1
        acl Authorization http_auth(inetusers)
        http-request auth realm octoprint if !Authorization
        use_backend webcam if { path_beg /webcam/ }
        default_backend octoprint
        errorfile 503 /etc/haproxy/errors/503-no-octoprint.http

frontend webcamonly
        bind *:9080
        option forwardfor except 127.0.0.1
        default_backend webcamstream
        errorfile 503 /etc/haproxy/errors/503-no-octoprint.http

backend octoprint
        reqrep ^([^\ :]*)\ /(.*) \1\ /\2
        reqadd X-Scheme:\ https if { ssl_fc }
        option forwardfor
        server octoprint1 127.0.0.1:5000

backend webcam
        reqrep ^([^\ :]*)\ /webcam/(.*)     \1\ /\2
        server webcam1  127.0.0.1:8080

backend webcamstream
        reqrep ^([^\ :]*)\ /(.*)   \1\ /\?action=stream
        server webcam1  127.0.0.1:8080

userlist inetusers
        group G1
        user [user] insecure-password [password] groups G1
```
Replace `[user]` and `[password]` on the last line with your desired credentials

#  Port Forwarding
Log into your router (I can't help you with this, there are too many brands of routers for me to know them all)
First assign your OctoPi and Static IP address.
Go to the `port forwarding` section of your router's settings and forward port `443` to the static IP address you just assigned your OctoPi.
Optionally, you can also forward port `9080` to allow others to view your webcam without credentials.
