<h1 align="center">TFTP & NFS Zedboard Configuration</h1>


<h2 align="left">Repo Overview</h2>

This repo is intended to capture usefull Petalinux confiuration settings and patches.

All toochain versions were 2022.1.

The hardware platform this project runs on is the Zedboard from Digilent.

<h2 align="left">Project Configuration Changes</h2>

This section covers all project configuration changes.


<h3 align="left"> U-boot Env Vars</h3>

In this process the u-boot varaible updates are implemented statically at build time. This is accomplished by editing the u-boot-xlnx source code. TFTP requires *ipaddr* and *serverip* are set correctly when u-boot launches on the target. The approach with the current toolchain is to use *petalinux-devtool* to create a u-boot-xlinx source directory, edit the source code and when finished create a patch file that petalinux can use.

This process starts with creating the u-boot-xlnx source repository in the petalinux workspace. To do this, run the following command:


*petalinux-devtool modify u-boot-xlnx*

Once run, the resulting source code can be found here:

*<proj_root\>/components/yocto/workspace/source/u-boot-xlnx/*

All editing to u-boot source code is done on the devtool branch and can be tested using the build command.  As the notes from devtool state, the patch branch should be rebased to devtool and the final patch will be generated from this branch.

Branches

* devtool
* devtool-no-overrides
* devtool-override-k26
* master
* xlnx_rebase_v2022.01

The patch branch which is rebased and used is the *devtool-override-k26* branch. 

Once edits and rebase are done, change to the *master* branch. Now the patch can be created by running the following command:

*petalinux-devtool finish u-boot-xlnx <target_layer\>*

The target layer (from my testing) should be */project-spec/meta-user/*

The layer can be identified by the /configs directory.

For this u-boot example, the patch file will be placed in */recipes-bsp/u-boot/files/0001_patch_filename*. The patch consists of a git diff and the *u-boot-xlnx_%.bbappend* file can be found one level up and should inclue a *SRC_URI+=* call to include the new patch file.






<h3 align="left">BOOT ARGS</h3>

<h4 align="left">Approach 1</h4>

Modified boot arguments must be passed to the kernel by modifying the system-user.dtsi file. The missing bootarg *nfsvers=3* causes nfs rootfs failure when booting and it doesn't seem *petalinux-config* implements this needed update.

* modify the system-user.dtsi file

    	chosen{
		bootargs="console=ttyPS0,115200 earlycon root=/dev/nfs nfsroot=192.168.1.109:/home/NFSshare,nfsvers=3,tcp ip=dhcp rw";
        };

    * found in *project-spec/meta-user/recipes-bsp/device-tree/files*

The system-user.dtsi file is automatically pulled in by the *device-tree.bbappend* file found one directory above system-user.dtsi.



<h4 align="left">Approach 2</h4>

This approach uses the device-tree.bbappend to patch the bootargs in the device tree.  It consists of modifying *project-spec/meta-user/recipes-bsp/device-tree/device-tree.bbappend* and adding the following line.

    # Append nfsvers=3 to bootargs
    sed -i '/bootargs =/s/,tcp/&,nfsvers=3/' ${DT_FILES_PATH}/system-conf.dtsi

Link to Xilinx forum post: [forum post](https://support.xilinx.com/s/question/0D52E00006iHpx0SAC/nfs-boot-zynq-ultrascale-mpsoc)



<h3 align="left">INITRD BUG</h3>

Discuss deploy bug when using tftp and nfs.  This is a issue where the INITRD tells Zynq to boot from rootfs on tftp server rather than use nfs.

When using tftp and nfs there seems to be a deployment bug regarding a pxe config file.  When the linux build (petalinux-build) is finished the output products are found in /image/linux directory. In this directory there is another directory(pxelinux.cfg) which holds a file called *default*. The last line in this default file should be removed (line starting with INITRD). This prevents the Zynq from pulling the rootfs off of the nfs server and instead causes it to unpack the zip in the outputs directory. Removing this will allow the Zynq to use the rootfs on the nfs server when tftp and nfs are used in combination.

















