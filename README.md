## Ports for constructing the 'gnome' and 'gnome-extra' collections

Contributions are welcome. If you don't know what it all about, please take the time to read the documentation at
http://www.nutyx.org/en/build-package.html
(version française)
http://www.nutyx.org/fr/build-package.html

It will explain you what's a collection, a git, a port, the tools around 'cards' etc
### Introduction
How does this works ? This git contains the 2 "gnome" and "gnome-extra" collections. As other collections, they have to be in the right order.  The "gnome-extra" collection need the "gnome" collection.

### How does this works:
First we get this git and the current git localy (step1) as normal user. As we want to install a NuTyX base system in a local directory, we need to become root admin. Before installing the NuTyX in a chroot, we adjust some configuration files (step 2) so that the install-houaphan script pickup them during the installation (step 3). Once the chroot is in place, we want to make the 2 git projects visible into the chroot (step 4 and 5). Now we are ready to start, so we can enter into the chroot (step 6). As we installed a minimal set of packages, we first need to install the 'devel' packages and some extra tools (step 6 and 7). One this is done, we have 2 choices. Because all the packages of this git collections will depends on the 'current' collections (base,cli or gui) we need to synchronise them (step 8).Either we synchronise ALL the existing binaries, means we just want to update a few packages (case 1). Either we want to build ALL the binaries ourself (case 2). So Case 1, we should use option -s and for case 2 it will be -a
### How to test this git:

#### 1. Clone it in your home directory

    $ cd
    $ git clone git://github.com/NuTyX/gnome.git
    $ git clone git://github.com/NuTyX/extra.git
    $ git clone git://github.com/NuTyX/current.git

#### 2. Become root until the end, define and create the directory used by the scripts:

 The script is checking the files /etc/install-houaphan.conf and /etc/install-houaphan.conf.d/cards.conf if they exist, if yes it will use them, so:

    $ su -
    # echo "LFS=/mnt/lfs
    DEPOT=/8.0
    VERSION=current" > /etc/install-houaphan.conf
    # mkdir -p /etc/install-houaphan.conf.d
    # cat > /etc/install-houaphan.conf.d/cards.conf << "EOF"
    dir /8.0/gnome-extra|http://downloads.nutyx.org
    dir /8.0/gnome|http://downloads.nutyx.org
    dir /8.0/gui|http://downloads.nutyx.org
    dir /8.0/cli|http://downloads.nutyx.org
    dir /8.0/base|http://downloads.nutyx.org
    dir /8.0/base-extra|http://downloads.nutyx.org
    base /8.0/base
    base /8.0/base-extra
    logdir /var/log/pkgbuild
    EOF
 We need to have a correct pkgmk.conf file as well so, lets create it:

 
    # cat > /etc/install-houaphan.conf.d/pkgmk.conf << "EOF"
    export CFLAGS="-O2 -pipe"
    export CXXFLAGS="${CFLAGS}"
    case ${PKGMK_ARCH} in
        "x86_64"|"")
            export MAKEFLAGS="-j4"
            ;;
        "i686")
            export CFLAGS="${CFLAGS} -m32"
            export CXXFLAGS="${CXXFLAGS} -m32"
            export LDFLAGS="${LDFLAGS} -m32"
            ;;
        *)
            echo "Unknown architecture selected! Exiting."
            exit 1
            ;;
    esac
    PKGMK_GROUPS=(devel man doc service)
    PKGMK_LOCALES=(fr de it es nl pt da nn sv fi)
    PKGMK_CLEAN="no"
    PKGMK_KEEP_SOURCES="yes"
    PKGMK_SOURCE_DIR="/tmp"
    PKGMK_WORK_DIR="/tmp/work"
    PKGMK_COMPRESS_PACKAGE="yes"
    PKGMK_COMPRESSION_MODE="xz"
    PKGMK_IGNORE_REPO="no"
    PKGMK_IGNORE_COLLECTION="no"
    PKGMK_IGNORE_RUNTIMEDEPS="no"
    EOF


#### 3. Install a base NuTyX system (assume below the user is 'lfs' so adapt to yours)

    # bash /home/lfs/houaphan/scripts/install-houaphan

#### 4. In your chroot Make the directory where the git copy will comes

    # mkdir -v /mnt/lfs/root/{current,extra,gnome}

#### 5. Mount your git project (assume below the user is 'lfs' so adapt to yours)

    # mount -o bind /home/lfs/gnome /mnt/lfs/root/gnome
    # mount -o bind /home/lfs/extra /mnt/lfs/root/extra
    # mount -o bind /home/lfs/current /mnt/lfs/root/current

#### 6. Enter now in your chroot (assume below the user is 'lfs' so adapt to yours)

    # bash /home/lfs/houaphan/scripts/install-houaphan -ec

#### 7. Prepare the first execution of the build script

    # get cards.devel wget vim rsync git tar
 
#### 8. If everything is OK, synchronize the  current 'base', 'cli', 'gui' and 'base-extra' collections binaries

    # cd /root/current
    # bash scripts/base -s
    # bash scripts/cli -s
    # bash scripts/gui -s
    # cd ../extra
    # bash scripts/base-extra -s
    
#### 9. If everything is OK, synchronize the 'gnome' collection binaries in case of a few updates

    # cd /root/gnome
    # bash scripts/gnome -s

#### 10. If everything is OK, check with cards level what's new

    # cards level

 It should shows all the packages available.

#### 11. If you want to re build completly 'gnome' collection from the sources 

    # bash scripts/gnome -a

#### 12. If you want to re build the 'gnome-extra' collection from the sources, add the proper line at the top of the cards.conf file like this:
dd

    # cd /root/gnome
    # bash scripts/gnome-extra -s
    # bash scripts/gnome-extra -a 

Have fun :)
