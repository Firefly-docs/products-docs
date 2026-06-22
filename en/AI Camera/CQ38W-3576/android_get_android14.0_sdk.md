### Download Android SDK
* The SDK can be obtained by email. Send the order number to <font color=red>sales@t-firefly.com</font> and indicate the required SDK name [firefly_rk3576_android14.0_git_20241126]()

* After downloading, verify the MD5 code:

    ```
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.001
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.002
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.003
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.004
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.005
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.006

    e368c4274125e754eeea35a97306e47b  firefly_rk3576_android14.0_git_20241126.7z.001
    d4436c73b800523dbb1df2ae0d524b26  firefly_rk3576_android14.0_git_20241126.7z.002
    9f6a7ed4f01a8952c135cc5bf6fd4ecd  firefly_rk3576_android14.0_git_20241126.7z.003
    3d2babe7c2c9f737d23e8843c2df3117  firefly_rk3576_android14.0_git_20241126.7z.004
    1d9ae98cc3e0959de19bc1dd628ed112  firefly_rk3576_android14.0_git_20241126.7z.005
    09e35581108741bf4a564829edcd1a5f  firefly_rk3576_android14.0_git_20241126.7z.006

    ```

* After confirming that it is correct, we can unzip:

    ```
    cd ~/proj/
    7z x ./firefly_rk3576_android14.0_git_20241126.7z.001 -oRK3576_Android14.0
    cd ./RK3576_Android14.0
    git reset --hard
    ```

   <font color=red>**Attention: To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.**</font>

#### Second, Update SDK
How to update code after downloading SDK:
```
1. Enter the SDK root directory
cd ~/proj/RK3576_Android14.0

2. we can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3576-android14.0-bundle.7z  -r -o. && mv rk3576-android14.0-bundle .bundle

3. Update the SDK
.bundle/update

4. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

Google Driver [Android14.0 Bundle](https://en.t-firefly.com/doc/download/341.html#other_713)。

<!-- 2. Download remote bundle repository
git clone https://gitlab.com/T-Firefly/rk3576-android14.0-bundle.git .bundle

3. If the download warehouse fails, it may be stuck or failed problems during synchronization. 
we can download and unzip it from the cloud disk link below to the SDK root directory.
 -->

