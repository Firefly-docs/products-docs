## Update kernel and U-Boot online

This section introduces a simple process of online update. Pack the kernel, U-Boot or other files that need to be updated into a deb installation package, and then import it to the local package warehouse to download and update automatically on the device. For user reference only.

### Prepare deb installation package

The kernel and U-Boot need to be upgraded during the operation, and the modified related files have been prepared in advance: `uboot.img`, `trust.img`, `boot.img`.

deb is the package format of Debian Linux. The most important thing for packaging is to create a `control` file in the `DEBIAN` directory. First create the deb working directory, and then create the corresponding directories and files in the deb directory:

```bash
mkdir deb
cd deb
mkdir firefly-firmware # Create firefly-firmware directory
cd firefly-firmware
mkdir DEBIAN # Create a DEBIAN directory, this directory is required.
mkdir -p usr/share/{kernel,uboot}
mv ~/boot.img ~/deb/firefly-firmware/usr/share/kernel # Place the corresponding file in the corresponding directory
mv ~/uboot.img ~/deb/firefly-firmware/usr/share/uboot
mv ~/trust.img ~/deb/firefly-firmware/usr/share/uboot
```

The files stored in the `DEBIAN` directory are the control files for the deb package installation and the corresponding script files.

Create the control file `control` and the script file `postinst` in the `DEBIAN` directory.

The content of the `control` file is as follows, which is used to record software identification, version number, platform, dependency information and other data.

```bash
Package: firefly-firmware # File directory name
Version: 1.0 # Version number
Architecture: arm64 # Architecture
Maintainer: neg # Maintainer, you can customize
Installed-Size: 1
Section: test
Priority: optional
Descriptionon: This is a deb test
```

The content of the `postinst` file is as follows, which is to write the kernel and U-Boot files that need to be updated into the script of the corresponding partition with the `dd` command:

```bash
echo "-----------uboot updating------------"
dd conv=fsync,notrunc if=/usr/share/uboot/uboot.img of=/dev/disk/by-partlabel/uboot

echo "-----------trust updating------------"
dd conv=fsync,notrunc if=/usr/share/uboot/trust.img of=/dev/disk/by-partlabel/trust

echo "-----------kernel updating------------"
dd conv=fsync,notrunc if=/usr/share/kernel/boot.img of=/dev/disk/by-partlabel/boot
```

Description: The `postinst` script is a script that runs after unpacking the data, and the corresponding ones are:

* preinst: a script that runs before unpacking the data;
* prerm: When uninstalling, the script that runs before deleting files;
* postrm: a script to run after deleting the file;

Users can understand other related knowledge points by themselves. Only the `preinst` script is used here.

The following is the created directory tree:

```
deb
└── firefly-firmware
    ├── DEBIAN
    │ ├── control
    │ └── postinst
    └── usr
        └── share
            ├── kernel
            │ └── boot.img
            └── uboot
                ├── trust.img
                └── uboot.img
```

Enter the deb directory and use the `dpkg` command to generate the deb package:

```
dpkg -b firefly-firmware firefly-firmware_1.0_arm64.deb
```

When generating a deb package, it is generally named according to this specification: `package_version-reversion_arch.deb`.

### Create a local warehouse

First install the required packages:

```
sudo apt-get install reprepro gnupg
```

Then use the GnuPG tool to generate a GPG key. After executing the command, follow the prompts:

```
gpg --gen-key
```

Execute `sudo gpg --list-keys` to view the key information just generated:

```
sudo gpg --list-keys

gpg: WARNING: unsafe ownership on homedir'/home/firefly/.gnupg'
gpg: checking trust database
gpg: marginals needed: 3 completes needed: 1 trust model: pgp
gpg: Depth: 0 Validity: 4 Signed: 0 Trust: 0-, 0q, 0n, 0m, 0f, 4u
gpg: The next trust database check will be conducted on May 29, 2021
/home/firefly/.gnupg/pubring.kbx
--------------------------------
pub rsa3072 2019-05-31 [SC] [Valid until: 2021-05-30]
      BCB65788541D632C057E696B8CBC526C05417B76
uid [absolutely] firefly <firefly@t-chip.com>
sub rsa3072 2019-05-31 [E] [Valid until: 2021-05-30]
```

Next to create a package warehouse, first create a directory:

```bash
cd /var/www
mkdir apt # package warehouse directory
mkdir -p ./apt/incoming
mkdir -p ./apt/conf
mkdir -p ./apt/key
```

Export the previously generated key to the warehouse folder, and please correspond to the user name and email address you created.

```
gpg --armor --export firefly firefly@t-chip.com> /var/www/apt/key/deb.gpg.key
```

Create a `distributions` file in the `conf` directory with the following content:

```bash
Origin: Neg # Your name
Label: Mian # The name of the library
Suite: stable # (stable or unstable)
Codename: bionic # Release code name, can be customized
Version: 1.0 # release version
Architectures: arm64 # Architecture
Components: main # component name, such as main, universe, etc.
Description: Deb source test # Related description
SignWith: BCB65788541D632C057E696B8CBC526C05417B76 # The GPG key generated in the above step
```

Build a warehouse tree:

```
reprepro --ask-passphrase -Vb /var/www/apt export
```

Add the deb package made in step 2 to the warehouse:

```
reprepro --ask-passphrase -Vb /var/www/apt includedeb bionic ~/deb/firefly-firmware_1.0_arm64.deb
```

You can view the files added in the library:

```bash
root@Desktop:~# reprepro -b /var/www/apt/ list bionic
bionic|main|arm64: firefly-firmware 1.0
```

Your package has been added to the repository, if you want to remove it, use the following command:

```
reprepro --ask-passphrase -Vb /var/www/apt remove bionic firefly-firmware
```

Install nginx server:

```
sudo apt-get install nginx
```

Modify the nginx configuration file `/etc/nginx/sites-available/default` to:

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/apt; //The path of the local package warehouse

    access_log /var/log/nginx/repo.access.log;
    error_log /var/log/nginx/repo.error.log;

    location ~ /(db|conf) {
        deny all;
        return 404;
    }
}
```

Restart the nginx server:

```
sudo service nginx restart
```

### Client update installation

In the client device, first add the source of the local package repository, add a new configuration file `bionic.list` under the directory `/etc/apt/sources.list.d`, the content is as follows:

```
deb http://192.168.31.106 bionic main
```

The IP address is the server address, `bionic` is the warehouse release code name, and `main` is the component name.

Obtain and add the GPG key from the server:

```
wget -O-http://192.168.31.106/key/deb.gpg.key | apt-key add-
```

After the update, you can install the `firefly-firmware_1.0_arm64` package in the custom software source:

```bash
root@firefly:/home/firefly# apt-get update
Hit:1 http://192.168.31.106 bionic InRelease
Hit:2 http://wiki.t-firefly.com/firefly-rk3399-repo bionic InRelease
Hit:3 http://ports.ubuntu.com/ubuntu-ports bionic InRelease
Hit:4 http://archive.canonical.com/ubuntu bionic InRelease
Hit:5 http://ports.ubuntu.com/ubuntu-ports bionic-updates InRelease
Hit:6 http://ports.ubuntu.com/ubuntu-ports bionic-backports InRelease
Hit:7 http://ports.ubuntu.com/ubuntu-ports bionic-security InRelease
Reading package lists... Done

root@firefly:/home/firefly# apt-get install firefly-firmware
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libllvm6.0
Use'apt autoremove' to remove it.
The following NEW packages will be installed:
  firefly-firmware
0 upgraded, 1 newly installed, 0 to remove and 3 not upgraded.
Need to get 0 B/6982 kB of archives.
After this operation, 1024 B of additional disk space will be used.
Selecting previously unselected package firefly-firmware.
(Reading database ... 117088 files and directories currently installed.)
Preparing to unpack .../firefly-firmware_1.0_arm64.deb ...
Unpacking firefly-firmware (1.0) ...
Setting up firefly-firmware (1.0) ...
-----------uboot updating------------
8192+0 records in
8192+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.437281 s, 9.6 MB/s
-----------trust updating------------
8192+0 records in
8192+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.565762 s, 7.4 MB/s
-----------kernel updating------------
39752+0 records in
39752+0 records out
20353024 bytes (20 MB, 19 MiB) copied, 0.1702 s, 120 MB/s
```

You can see that the poinst script in the deb package was executed during the installation process. After the installation is complete, restart the device and the update is complete.

In the `/usr/share` directory, you can also see the `kernel` and `uboot` directories, which store `boot.img`, `uboot.img` and `trust.img` respectively.

Precautions

* In making the deb package, the directory at the same level as `DEBIAN` is regarded as the root directory, that is, the files placed in the same directory as `DEBIAN`, after the deb package is installed on the client, it can be found in the root directory of the client;
* The files and scripts in the deb package should be adjusted according to their actual situation;
* Every time you modify the configuration file in the warehouse, you must re-import the warehouse directory tree;
* In the nginx server configuration, the `root` parameter configures the address of the warehouse, please modify it according to your actual situation;
* When the client adds the new download source file, please check the correct server address, package warehouse code name and component name. Note that the client must connect to the server;
* The client must use the `apt-key add` command to add the GPG key before it can obtain the local warehouse information.