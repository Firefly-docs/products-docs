
# IO 
## INPUT 
```
sudo su
gpioget `gpiofind "PAG.01"`
```

## OUTPUT
### Set High
```
sudo su
gpioset `gpiofind "PZ.01"`=1
```

### Set Low
```
sudo su
gpioset `gpiofind "PZ.01"`=0
```



