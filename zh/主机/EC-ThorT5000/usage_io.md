
# IO 使用 


## INPUT 
```
sudo su
gpioget `gpiofind "PAL.01"`
```

## OUTPUT
### 拉高
```
sudo su
gpioset `gpiofind "PT.06"`=1
```

### 拉低 
```
sudo su
gpioset `gpiofind "PT.06"`=0
```


