* Compile the kernel separately：

```
cd ~/proj/iCore-3568JQ/
./FFTools/make.sh -k -j8
```

* Compile the uboot separately：

```
cd ~/proj/iCore-3568JQ/
./FFTools/make.sh -u -j8
```

* Compile Android upper layer separately：

```
cd ~/proj/iCore-3568JQ/
./FFTools/make.sh -a -j8
```

* Compile ubooot, kernel, Android at the same time：

```
cd ~/proj/iCore-3568JQ/
./FFTools/make.sh -j8
```