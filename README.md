pfs on rpi3
===

I created this fork to connect my pCloud drive to my raspberry pi 3 to stream videos. The instructions in this file are tested for an rpi3 with OSMC installed and I'm currently using this setup.

Why not console-client/pcloudcc?
---
I tried that and pcloudcc just uses much more CPU and RAM than this and also had more problems with buffering, even after I tried to fix the caching issues because it uses too much RAM.

Things I changed:
---
* The in-RAM caching defaults are way too high for an rpi3, so I lowered those.
* You cannot mount with --username and --password any more, instead if you try it, mount.pfs will generate an authentication token for you to use instead. I did this because I could not get the curl method to work, so I just used the implementation in pfs.c which seemed to work fine...
* I added the pfs.service file that you can edit after getting your auth key and copy to `/{usr/lib|etc}/systemd/system/pfs.service` - 
With the service file systemctl start|stop|enable|disable pfs should work as expected.


pfs
===

pCloud filesystem client

To compile, you need fuse and the openssl headers. In debian,
they're in libssl-dev and libfuse-dev, in fedora in fuse-devel and
openssl-devel.

## Setup instructions on `systemd` based machines

### Install pfs

#### Debian based
```sh
(sudo) apt-get install libfuse-dev libssl-dev fuse-dbg
git clone https://github.com/Gordin/pfs.git
cd pfs
make
(sudo) make install
```

#### yum
```sh
(sudo) yum install fuse-devel openssl-devel
git clone https://github.com/Gordin/pfs.git
cd pfs
make
(sudo) make install
```

### Get auth token

```sh 
./mount.pfs --username <your pcloud email> --password '<your pcloud password'
```

You will see output similar to the following:
````sh
Getting Auth Token
Got Auth Token: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Use this to mount pCloud: 
/usr/bin/mount.pfs --auth XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX --ssl /home/<your user>/pCloudDrive
````

### Mount pDrive

```sh
mkdir /mnt/pdrive
mount.pfs --auth <you auth token here> --ssl /mnt/pdrive
```

And keep the auth bit.
Note that this is not required, but otherwise you'll have to put your username
and password in the service file.

### Autostart

Edit pfs.service file and fill in these lines

```
User=<user the mounted drive will belong to>
Group=<group the mounted drive will belong to>
ExecStart=/usr/bin/mount.pfs --auth <you auth token here> <path where you want to mount your drive>
```

Copy the file to a diroctery where systemd will find it
```sh
(sudo) cp pfs.service /etc/systemd/system/pfs.service
or
(sudo) cp pfs.service /usr/lib/systemd/system/pfs.service
```

Make the service start automatically:

```sh
(sudo) systemctl enable --now pfs.service
```
