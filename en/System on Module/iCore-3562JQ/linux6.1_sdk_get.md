## Download the SDK

Please contact sales@t-firefly.com to get SDK download link.

<font color=red>

**Notice:**

**1. SDK use cross-compilation, so use SDK in x86_64 PC, do not download SDK to the device**

**2. We suggest to use Ubuntu20.04 (real PC or docker) to build, other OS may cause building failure**

**3. Do not place or decompress the SDK archive in Virtual Machine share folder or non-english folder**

**4. Please use the regular user to get/compile the SDK, use root privilege may cause building failure**
</font>

Prepare a empty folder to place the SDK, we suggest a folder under your home dir. For example `~/proj`

### Verify and Decompress

Verify the MD5 of SDK archives after download.
```bash
$ md5sum rk3562_6.1.tgz.split*
```

Compare the results with the md5sum.txt, if MD5s are the same, you can decompress now:
```bash
# decompress
cd ~/proj/
cat rk3562_6.1.tgz.split0* | tar -zx

# you will get
~/proj/rk3562_6.1

# install python2
sudo apt update && sudo apt install python2

# check out files from repo
cd ~/proj/rk3562_6.1
python2 .repo/repo/repo sync -l --no-tags
```
