# IO 
## Definition
### Module - Core-3588SJD4-AI

* Phoenix terminal pin definition

| GND  |                 485A                 |                485A                 | GND  | CAN_H |                485A                 |
| :--: | :----------------------------------: | :---------------------------------: | :--: | :---: | :---------------------------------: |
| 12V  |                 485B                 |                485B                 | GND  | CAN_L |                485B                 |
|      | /dev/ttysWK2<br />re-de pin：GPIO122 | /dev/ttysWK1<br />re-de pin：GPIO35 |      |       | /dev/ttysWK0<br />re-de pin：GPIO34 |

* RS232 Interface Definition

|  UP  | /dev/ttyS9 |
| :--: | :--------: |
| DOWN | /dev/ttyS6 |

### Module - Core-3576JD4

* Phoenix terminal pin definition

| GND  |                 485A                 |                485A                 | GND  | CAN_H |                485A                 |
| :--: | :----------------------------------: | :---------------------------------: | :--: | :---: | :---------------------------------: |
| 12V  |                 485B                 |                485B                 | GND  | CAN_L |                485B                 |
|      | /dev/ttysWK2<br />re-de pin：GPIO119 | /dev/ttysWK1<br />re-de pin：GPIO98 |      |       | /dev/ttysWK0<br />re-de pin：GPIO99 |

* RS232 Interface Definition

|  UP  | /dev/ttyS8 |
| :--: | :--------: |
| DOWN | /dev/ttyS3 |

## RS485 RE-DE PIN USE

For example, to operate `122`, you need:

1. Export the GPIO, run as root user:
    ```
    echo 122 > /sys/class/gpio/export
    ```
2. Operate GPIO, set to read input or output specified value:
    ```
    cd /sys/class/gpio/gpio122
    
    # Set GPIO output mode
    echo out > direction
    # GPIO outputs low level, RS485 is set to data receiving mode
    echo 0 > value
    # GPIO outputs high level, RS485 is set to data receiving mode
    echo 1 > value
    ```

There are a few things to note:
- If the GPIO has been applied by other drivers, it cannot be exported successfully, and subsequent read and write operations cannot be performed.
- Involving the multiplexing of IO functions, if the function of the pin is already GPIO, the above method can be used; otherwise, the pinctrl state must be specified in the device tree.
