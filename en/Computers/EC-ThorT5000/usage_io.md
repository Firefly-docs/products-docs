
# IO 

## INPUT 
```
sudo su
gpioget `gpiofind "PAL.01"`
```

## OUTPUT
### Set High
```
sudo su
gpioset `gpiofind "PT.06"`=1
```

### Set Low
```
sudo su
gpioset `gpiofind "PT.06"`=0
```


