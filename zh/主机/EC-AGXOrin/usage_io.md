
# IO 使用 
## INPUT 
```
sudo su
gpioget `gpiofind "PAG.01"`
```

## OUTPUT
### 拉高
```
sudo su
gpioset `gpiofind "PZ.01"`=1
```

### 拉低 
```
sudo su
gpioset `gpiofind "PZ.01"`=0
```




