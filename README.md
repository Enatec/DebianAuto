# Auto Debian

Automated Debian 12 installation

Ain was to create a slim and secure base image. Mostly used for virtual hosts in an Hyper-V enviroment.

Most of the security and hardening related work is based on the work of [jhochwald (Joerg Hochwald)](https://github.com/jhochwald/) and [Mokkujin (Chris)](https://github.com/Mokkujin/).

Many of the [cisecurity.org](https://www.cisecurity.org/) requirements and recommendations should be matched!

You can apply the rc.local file to any Debian, or Debian based system. It is meant to help the system administrator built a secure environment. This fille works best for a newly installed system!

Even if all the security is curated carefully we can not guarantee that it will work for you. It will not magically secure any random excisting host. That said, it might be a good starting point to built a secure system/environment.

## What it does

- It will install a slim Debian 12 server system
- It will create a default User with the name `enadmin`
- The password of `enadmin` user is: `Start1234`, and you should change this directly after the installation
  - You can change the user and/or the password in the `preseed.cfg` file
  - Please remember to change to `rc.local` if you change to username. Please see line 23 for this change.
- The user `enadmin` can administer the freshly installed system and is allowed to login remotely (you must change the SSH keys!), and use `sudo`
  - Please see Line 111 in the `rc.local` file, this is where you need to place your own SSH public key
- The user `root` is disabled! Please keep this in mind!!!

## Further reading CIS Security recommendation:

- **Center for Internet Security**: [https://www.cisecurity.org/](https://www.cisecurity.org/)
- **CIS recommendations**: [CIS Benchmarks](https://learn.cisecurity.org/benchmarks)

## Found a bug or Issue?

If you find something bad (like a bug, error, or any issue), please report it here by open an [Issue](https://github.com/Enatec/DebianAuto/issues).

Or even better: Fork the Repository, fix it and submit a [pull request](https://github.com/Enatec/DebianAuto/pulls), so others can participate too!

See the [Contribution Guide](CONTRIBUTING.md) for more details!

## Create an ISO Image

```shell
#!/usr/bin/env bash
###
# This script will create a new ISO with the preseed.cfg file and the rc.local script
# The rc.local script will be executed at the end of the installation
#
# All commands are fully qualified to ensure the script can be run from anywhere and no aliases are used
# This script is designed to be run from the Windows Subsystem for Linux (WSL) on Windows 11
# Many of the commands are run with sudo, so you will need to ensure you have sudo access on your WSL
# You will also need to ensure you have the required packages installed (genisoimage and syslinux-utils)
###

# This should be the name of the ISO you want to use as a base (this is the Debian 10.4.0 netinst ISO)
isoIn=debian-12.4.0-amd64-netinst.iso
# This is the name of the new ISO that will be created
isoOut=preseed-${isoIn}
# This is the source directory where the files are located
SourceDirectory='/mnt/c/DEV/DebianAuto'
# This is the target directory where the files will be copied to
TargetDirectory='/home/enadmin/DebianAuto'

# Ensure we have the required packages
/usr/bin/sudo /usr/bin/apt install genisoimage syslinux-utils -y >/dev/null 2>&1
# Clean-up existing ISO in the source directory
/usr/bin/sudo /usr/bin/rm -f $SourceDirectory/$isoOut >/dev/null 2>&1
# Clean-up target directory
/usr/bin/sudo /usr/bin/rm -rf $TargetDirectory >/dev/null 2>&1
# Copy files to target directory
/usr/bin/cp -rT $SourceDirectory $TargetDirectory
# Goto target directory
pushd $TargetDirectory >/dev/null 2>&1
# Check if the ISO exists in the target directory
if [ -f $isoIn ]; then
   # Check if we need to remove the old ISO
   if [ -f $isoOut ]; then
      # Remove the old ISO
      /usr/bin/sudo /usr/bin/rm -f $isoOut
   fi
   # Remove any old directories
   /usr/bin/sudo /usr/bin/rm -rf new-iso vanilla-iso >/dev/null 2>&1
   # Create new directories
   /usr/bin/mkdir -p vanilla-iso new-iso >/dev/null 2>&1
   # Mount the ISO
   /usr/bin/sudo /usr/bin/mount -o loop $isoIn vanilla-iso >/dev/null 2>&1
   # Copy the ISO contents to the new directory
   /usr/bin/cp -rT vanilla-iso/ new-iso/
   # Unmount the ISO
   /usr/bin/sudo /usr/bin/umount vanilla-iso >/dev/null 2>&1
   # Goto the Jumpstart directory
   pushd Jumpstart/ >/dev/null 2>&1
   if [ -f ../debian.tar.gz ]; then
      # Remove the old tarball
      /usr/bin/sudo /usr/bin/rm -f ../debian.tar.gz >/dev/null 2>&1
   fi
   # Change permissions on rc.local
   /usr/bin/chmod 700 ./rc.local >/dev/null 2>&1
   # Create the tarball
   /usr/bin/tar -czf ../debian.tar.gz . >/dev/null 2>&1
   # Return to the target directory
   popd >/dev/null 2>&1
   # Copy the tarball and preseed.cfg to the new ISO
   /usr/bin/cp debian.tar.gz new-iso/
   /usr/bin/cp preseed.cfg new-iso/
   # Change permissions on the preseed.cfg
   /usr/bin/chmod 644 new-iso/isolinux/adtxt.cfg
   # Add the preseed.cfg entry to the boot menu
   /usr/bin/cat >> new-iso/isolinux/adtxt.cfg <<EOF
label auto-wipe
menu label ^Automatic - DESTRUCTIVE
kernel /install.amd/vmlinuz
append auto=true priority=critical vga=788 file=/cdrom/preseed.cfg initrd=/install.amd/initrd.gz --- quiet
EOF
   # Change permissions on the preseed.cfg
   /usr/bin/chmod 444 new-iso/isolinux/adtxt.cfg
   # Change to the new ISO directory
   pushd new-iso/ >/dev/null 2>&1
   # allow write access to the md5sum.txt file
   /usr/bin/chmod +w md5sum.txt >/dev/null 2>&1
   # Generate the new md5sum.txt file
   /usr/bin/find -follow -type f ! -name md5sum.txt -print0 2>/dev/null | /usr/bin/xargs -0 md5sum > md5sum.txt
   # remove write access to the md5sum.txt file
   /usr/bin/chmod -w md5sum.txt >/dev/null 2>&1
   # Go back to the target directory
   popd >/dev/null 2>&1
   # Create the new ISO
   /usr/bin/sudo /usr/bin/genisoimage -r -J -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o $isoOut new-iso >/dev/null 2>&1
   # Optional, if you want to make the ISO hybrid (USB bootable) for VM testing
   /usr/bin/sudo /usr/bin/isohybrid $isoOut >/dev/null 2>&1
   # Remove the temporary directories
   /usr/bin/sudo /usr/bin/rm -rf new-iso vanilla-iso >/dev/null 2>&1
   # Remove the tarball
   /usr/bin/sudo /usr/bin/rm -f debian.tar.gz >/dev/null 2>&1
   # Copy the ISO to the source directory
   /usr/bin/sudo /usr/bin/cp -rf $isoOut $SourceDirectory
   # Change ownership and permission of the ISO
   /usr/bin/sudo /usr/bin/chown enadmin:enadmin $SourceDirectory/$isoOut
   /usr/bin/sudo chmod 644 $SourceDirectory/$isoOut
else
   # Whoops, we can't find the ISO
   echo "Unable to find $isoIn"
fi

# Return to where we started
popd >/dev/null 2>&1
# Remove the target directory (Clean-up)
/usr/bin/sudo /usr/bin/rm -rf $TargetDirectory >/dev/null 2>&1
```

Please check and review all the values, before just start the script!

### Requirements

- `debian-12.4.0-amd64-netinst.iso` (can be [downloaded](https://www.debian.org/download) from the Debian project)
- The files of this repository
- A Linux based system (We use Debian 12 as WSL2 Image)
- All required packages will be downloaded (if required)

### Alternatives

You can also deploy the `preseed.cfg` via a Server, e.g., Web-Server or File-Share. As long as the system can reach it during the installation.

In addition, you can change the `preseed.cfg` file (and the very end) to download the Jumstart file from the same location where the `preseed.cfg` itself is found, or any other location the system can reach. It is not required to build your own ISO or put the Jumpstart file on the install media.

## Contribution

**More then welcome!**

Please see the [Contribution Guide](CONTRIBUTING.md) for more details!

## Default License

In our opinion: All the stuff here should be free, and the license should be as flexible as possible.

### BSD 3-Clause License

Copyright (c) 2023, enabling Technology GmbH - All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.
3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

_By using the Software, you agree to the License, Terms and Conditions above!_

### DISCLAIMER

- Use at your own risk, etc.
- This is open-source software, if you find an issue try to fix it yourself. There is no support and/or warranty in any kind
- This is a third-party Software
- The developer of this Software is NOT sponsored by or affiliated with the Debian project
- The Software is not supported by the Debian projects
- By using the Software, you agree to the License, Terms, and any Conditions declared and described above
- If you disagree with any of the terms, and any conditions declared: Just delete it and build your own solution
