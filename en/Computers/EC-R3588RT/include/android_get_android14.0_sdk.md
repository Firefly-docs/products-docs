### Download Android SDK
* The SDK can be obtained by email. Send the order number to <font color=red>sales@t-firefly.com</font> and indicate the required SDK name [firefly_rk3588_android14_git_20240822](https://en.t-firefly.com/doc/download/207.html)

* After downloading, verify the MD5 code:

    ```
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.001
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.002
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.003
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.004
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.005
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.006
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.007
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.008
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.009
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.010
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.011

    c9cc4863a3ad3b48f3c74d1f061d0438  firefly_rk3588_android14_git_240822.7z.001
    e81083e1b9b435e4e666f7d08dcd1bd6  firefly_rk3588_android14_git_240822.7z.002
    49b7aace84ab01b9a014b410fa26f42f  firefly_rk3588_android14_git_240822.7z.003
    1632fc98d881d5f4a8ed953afc9c73b1  firefly_rk3588_android14_git_240822.7z.004
    f581633cefdef6103de584501612e5dd  firefly_rk3588_android14_git_240822.7z.005
    f0e591f54a8969755775642567dabb17  firefly_rk3588_android14_git_240822.7z.006
    ee3d00d688ac0ad2bb43f8fe73f354bd  firefly_rk3588_android14_git_240822.7z.007
    09dc458646cce9f53a792d68e0ece3ec  firefly_rk3588_android14_git_240822.7z.008
    add00c4da466d50782ea5dba279c72d1  firefly_rk3588_android14_git_240822.7z.009
    51438ba104f0a67fd4fa09b5fee0d301  firefly_rk3588_android14_git_240822.7z.010
    b733f1d13c070862b5af6ce71052c020  firefly_rk3588_android14_git_240822.7z.011

    ```

* After confirming that it is correct, we can unzip:

    ```
    cd ~/proj/
    7z x ./firefly_rk3588_android14.0_git_20240822.7z.001 -oRK3588_Android14.0
    cd ./RK3588_Android14.0
    git reset --hard
    ```

   <font color=red>**Attention: To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.**</font>

#### Second, Update SDK
Note: Be sure to update the remote warehouse after decompression. The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/RK3588_Android14.0

2. Download remote bundle repository
git clone https://gitlab.com/T-Firefly/rk3588-android14.0-bundle.git .bundle

3. If the download warehouse fails, it may be stuck or failed problems during synchronization. 
we can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3588-android14.0-bundle.7z  -r -o. && mv rk3588-android14.0-bundle .bundle

4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update

5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

Google Driver [Android14.0 Bundle](https://en.t-firefly.com/doc/download/207.html#other_768)。

