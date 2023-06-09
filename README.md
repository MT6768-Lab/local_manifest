## Build CrDroid

### Install the build packages

Several packages are needed to build CrDroid. You can install these using your distribution's package manager.
A [package manager](https://en.wikipedia.org/wiki/Package_manager) in Linux is a system used to install or remove software
(usually originating from the Internet) on your computer. With Ubuntu, you can use the Ubuntu Software Center. Even better, you may also use the `apt-get install`
command directly in the Terminal.

To build CrDroid, you'll need:

* `bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs gnupg gperf imagemagick
   lib32ncurses5-dev lib32readline-dev lib32z1-dev libelf-dev liblz4-tool libncurses5 libncurses5-dev
   libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync zip zlib1g-dev python-is-python3`

For Ubuntu versions older than 20.04 (focal), install also:

* `libwxgtk3.0-dev`

While for Ubuntu versions older than 16.04 (xenial), install:

* `libwxgtk2.8-dev`

### Create the directories

You'll need to set up some directories in your build environment.

To create them:

```
mkdir -p ~/bin
mkdir -p ~/android/crdroid
```

The `~/bin` directory will contain the git-repo tool (commonly named "repo") and the `~/android/crdroid` directory will contain the source code of CrDroid.

### Install the `repo` command

Enter the following to download the `repo` binary and make it executable (runnable):

```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

### Put the `~/bin` directory in your path of execution

In recent versions of Ubuntu, `~/bin` should already be in your PATH. You can check this by opening `~/.profile` with a text editor and verifying the following code exists (add it if it is missing):

```
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

Then, run `source ~/.profile` to update your environment.


### Configure git
Given that `repo` requires you to identify yourself to sync Android, run the following commands to configure your `git` identity:
```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```


### Turn on caching to speed up build

Make use of [`ccache`](https://ccache.samba.org/) if you want to speed up subsequent builds by running:

```
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
```

and adding that line to your `~/.bashrc` file. Then, specify the maximum amount of disk space you want `ccache` to use by typing this:

```
ccache -M 50G
```

where `50G` corresponds to 50GB of cache. This needs to be run once. Anywhere from 25GB-100GB will result in very noticeably increased build speeds
(for instance, a typical 1hr build time can be reduced to 20min). If you're only building for one device, 25GB-50GB is fine. If you plan to build
for several devices that do not share the same kernel source, aim for 75GB-100GB. This space will be permanently occupied on your drive, so take this
into consideration.

You can also enable the optional `ccache` compression. While this may involve a slight performance slowdown, it increases the number of files that fit in the cache. To enable it, run:

```
ccache -o compression=true
```

If compression is enabled, the `ccache` size can be lower (aim for approximately 20GB for one device)


### Initialize the CrDroid source repository

Enter the following to initialize the repository:

```
cd ~/android/crdroid
repo init -u https://github.com/crdroidandroid/android.git -b 13.0 --git-lfs
```

### Import device specific source manifest

Enter the following to import device specific source manifest:

```
git clone https://github.com/MT6768-Lab/local_manifest --depth 1 -b crdroid-13.0 .repo/local_manifests
```

### Download the source code

To start the download of the source code to your computer, type the following:

```
repo sync
```

The CrDroid manifests include a sensible default configuration for repo, which we strongly suggest you use (i.e. don't add any options to sync).
For reference, our default values are `-j 4` and `-c`. The `-j 4` part implies be four simultaneous threads/connections. If you experience
problems syncing, you can lower this to `-j 3` or `-j 2`. On the other hand, `-c` makes repo to pull in only the current branch instead of all branches that are available on GitHub.

This may take a while, depending on your internet speed. Go and have a beer/coffee/tea/nap in the meantime!
The `repo sync` command is used to update the latest source code from CrDroid and Google. Remember it, as you may want to
do it every few days to keep your code base fresh and up-to-date. But note, if you make any changes, running `repo sync` may wipe them away!


### Start the build

Time to start building! Now, type:

```
source build/envsetup.sh
brunch lancelot  # for build lancelot
brunch merlinx   # for build merlinx
```

The build should begin.

Assuming the build completed without errors (it will be obvious when it finishes), type the following in the terminal window the build ran in:

```
cd $OUT
```

There you'll find all the files that were created. 

`crDroidAndroid-13.0-20230608-lancelot-v9.5.zip` or `crDroidAndroid-13.0-20230608-merlinx-v9.5.zip`, which is the CrDroid
installer package.