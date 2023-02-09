# Adding Docker (and Virtualization) capability on your Intel Edison

## 0. Before starting
First of all I recommend you to build the Edison image as you always do (following the [instructions provided before](https://edison-fw.github.io/meta-intel-edison/)),
that way you'll ensure that the image build fine and we can go on adding more "components".

Ensure you are in `my_Edison_Workspace/out/linux64` folder before continuing,\
check then that build environment is active `source poky/oe-init-build-env`
<br><br>

## 1. Downloading additional layers
In order to be able to compile docker-ce package you'll need to fetch some others Yocto layers:

1 - `cd poky` move to the folder that will contain the new layer\
2 - `git clone -b honister https://git.yoctoproject.org/meta-virtualization` get the virtualization layer\
3 - `cd ..`

NOTE that you have to specify after the `-b` tag the correct branch you are compiling on
<br><br>

## 2. Adding layers to the layers file
1 - now we'll need to edit the file containing our layer definitions
```
sudo nano build/conf/bblayers.conf
```

2 - add the following lines after `BBLAYERS ?= " \`: \
```
/home/username/my_Edison_Workspace/out/linux64/poky/meta-openembedded/meta-filesystems \
/home/username/my_Edison_Workspace/out/linux64/poky/meta-virtualization \
```
be sure to put the right path to the layer you just downloaded, changing your username. The meta-filesystem layer is already present on the edison build environment, you just need to add it to the `bblayers.conf` file
<br><br>

## 3. Adding docker-ce recipe
1 - edit `edison-image.bb` file\
```
sudo nano ../../meta-intel-edison/meta-intel-edison-distro/recipes-core/images/edison-image.bb
```

2 - add to the bottom of it the following line:\
```
`IMAGE_INSTALL:append = " docker-ce"`
```
this will tell the compiler to compile `docker-ce` package
<br><br>

## 4. Adding virtualization flag in Kernel
1 - edit `local.conf` file\
```
sudo nano build/conf/local.conf
```
2 - add to the bottom of the file the following line\
`DISTRO_FEATURES:append = " virtualization"`
<br><br>

## 5. Adding fragment to enable kernel features Docker requires
We are now going to add a kernel fragment file that is needed to enable certain kernel features Docker requires to run\
\
1 - Now download the file  `fragment.cfg` that can be found [here](https://drive.google.com/file/d/19bxvTQ04M4o-VQcKqkcqkVXELqVn7Pet/view?usp=share_link) (or in this GitHub repo) and copy it inside `my_Edison_Workspace/meta-intel-edison/meta-intel-edison-bsp/recipes-kernel/linux/files`\
2 - Go to `my_Edison_Workspace/meta-intel-edison/meta-intel-edison-bsp/recipes-kernel/linux` find the corrisponding kernel file you are building and add to the bottom the line:\
`SRC_URI:append = " file://fragment.cfg"`

If you don't know the kernel you are compiling copy this line inside all files in the directory
<br><br>

## 6. Compiling again
Now you can compile the image again
```
bitbake linux-yocto -c cleansstate
bitbake -k edison-image
```
once done you can flash the image as usual with `flashall`
<br><br>

## 7. Booting and setting up
1 - After rebooting you can enable the Docker Daemon
```
systemctl enable docker
systemctl start docker
```
2 - You should now be able to test Docker:
```
docker run hello-world
```
<br><br><br>
## -1. How to find kernel flags to be enabled for Docker
You should first enable 
```
CONFIG_IKCONFIG=y
CONFIG_IKCONFIG_PROC=y
CONFIG_IKHEADERS=y
```
flags in order to run the `check-config.sh` file that checks required flags so after adding those to your kernel you can do:
```
wget https://github.com/moby/moby/raw/master/contrib/check-config.sh
sudo bash check-config.sh
```
it will output all required flags, show which one you have and which one you are missing.\
Now you can follow the steps [here](https://edison-fw.github.io/meta-intel-edison/5.1-Bitbake-tricks#configuring-the-kernel-and-grab-the-kernel-fragment) to generate your kernel fragment with those configs enabled.\
I recommend to add features not as modules because some of them won't be loaded correctly