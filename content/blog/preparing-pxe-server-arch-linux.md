+++
title = "Part 0: Preparing the PXE Server on Arch Linux"
author = ["Dimitar Petrov"]
date = 2020-07-21T19:45:00+03:00
lastmod = 2020-07-21T19:56:00+03:00
tags = ["pxe", "linux"]
draft = false
+++

<div class="note">
  <div></div>

_TODO Explain the benefits of setting an PXE server in home LAN_
_TODO Develop a graph of my home infrastructure with ditaa_
_TODO Explain that this works only for BIOS booting_

</div>

In this series I am going to demonstrate how to set up a Linux environment for power users on Arch Linux and utilize literate DevOps, using Spacemacs and Org-Babel.

_Note: This step is an optional one, you can always just boot from USB stick and continue from Part 1 of the Arch Install Series_

Let's configure a PXE server following the official arch linux [guide](https://wiki.archlinux.org/index.php/PXE).

```sh
ssh durden
```

Next download the latest iso file from an official repo and verify signature.

```sh
mkdir ~/archstore; cd ~/archstore
repo="http://mirror.rackspace.com/archlinux/iso/latest/"

latest_rel=$(curl -s $repo | perl \
              -ne 'print $1 . "\n" if ($_ =~ /.*href="(.*)\.iso"/)')

printf "Latest Arch release: %s\n" $latest_rel

iso=${latest_rel}.iso
sig=${latest_rel}.iso.sig
sha1="sha1sums.txt"

wget --quiet ${repo}${iso}
wget --quiet ${repo}${sig}
wget --quiet ${repo}${sha1}

printf "Files:\n"
du -sh *

shasum ${sha1}
if [[ $? -eq 0 ]]; then echo "SHA1 Sums: Correct"; fi

sudo pacman-key -v ${sig}
if [[ $? -eq 0 ]]; then echo "SIG: Verified"; fi
```

```text
Latest Arch release: archlinux-2017.04.01-x86_64

Files:
479M	archlinux-2017.04.01-x86_64.iso
8.0K	archlinux-2017.04.01-x86_64.iso.sig
8.0K	sha1sums.txt

bec7a89b9f4d76f1f9fb56285fc1e164429db773  sha1sums.txt
SHA1 Sums: Correct

==> Checking archlinux-2017.04.01-x86_64.iso.sig...
gpg: assuming signed data in 'archlinux-2017.04.01-x86_64.iso'
gpg: Signature made Sat 01 Apr 2017 02:34:21 PM EEST
gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC
" [full]
SIG: Verified
```

Please feel free to visit [Arch Master Signing Keys](https://www.archlinux.org/master-keys/) and make sure the signature/iso you downloaded is not compromised. There is always the possibility that you got served [Malicious Image](https://www2.cs.arizona.edu/stork/packagemanagersecurity/attacks-on-package-managers.html#explanation).

Information regarding the signature can be extracted with `gpg --list-packets`

```sh
gpg --list-packets < ${sig}
```

```text

off=0 ctb=89 tag=2 hlen=3 plen=307
:signature packet: algo 1, keyid 7F2D434B9741E8AC
	version 4, created 1491046461, md5len 0, sigclass 0x00
	digest algo 8, begin of digest 54 88
	hashed subpkt 33 len 21 (issuer fpr v4 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC)
	hashed subpkt 2 len 4 (sig created 2017-04-01)
	subpkt 16 len 8 (issuer key ID 7F2D434B9741E8AC)
	data: [2045 bits]
```

Now let's mount the image and prepare the files for the PXE server

```sh
dt=$(date "+%Y%m%d")
sudo mkdir /srv/pxe
sudo mount -o loop,ro ${iso} /mnt
sudo rsync -aAXv /mnt/ /srv/pxe > rsync.log.${dt}
tail -3 rsync.log.${dt}
du -sm /srv/pxe
```

```text
sent 500,489,449 bytes  received 1,908 bytes  11,246,996.79 bytes/sec
total size is 500,361,240  speedup is 1.00

478	/srv/pxe
```
