# IO 
## Definition
### Main Module - Core-3588JD4

| Name | GPIO Number | IO Type|
| --- | --- | --- |
| IO1 | 36 | INPUT  |
| IO2 | 102 | INPUT  |
| IO3 | 44 | INPUT  |
| IO4 | 38 | INPUT  |
| IO5 | 137 | OUTPUT(relay) |

### Main Module - Core-3588SJD4 AI

| Name | GPIO Number | IO Type|
| --- | --- | --- |
| IO1 | 36 | INPUT  |
| IO2 | 46 | INPUT  |
| IO3 | 47 | INPUT  |
| IO4 | 38 | INPUT  |
| IO5 | 61 | OUTPUT(relay) |

### Main Module - Core-3576JD4

| Name | GPIO Number | IO Type|
| --- | --- | --- |
| IO1 | 127 | INPUT  |
| IO2 | 125 | INPUT  |
| IO3 | 129 | INPUT  |
| IO4 | 126 | INPUT  |
| IO5 | 128 | OUTPUT(relay) |

## Use
For example, to operate `486`, you need:

1. Export the GPIO, run as root user:
    ```
    echo 486 > /sys/class/gpio/export
    ```
2. Operate GPIO, set to read input or output specified value:
    ```
    cd /sys/class/gpio/gpio486
    # set input mode
    echo in > direction
    cat value
    
    # set output mode
    echo out > direction
    # output low level
    echo 0 > value
    # output high level
    echo 1 > value
    ```

There are a few things to note:
- If the GPIO has been applied by other drivers, it cannot be exported successfully, and subsequent read and write operations cannot be performed.
- Involving the multiplexing of IO functions, if the function of the pin is already GPIO, the above method can be used; otherwise, the pinctrl state must be specified in the device tree.
