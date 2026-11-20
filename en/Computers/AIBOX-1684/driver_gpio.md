# GPIO

The BM1684 chip contains 3 groups of GPIO controllers, each of which controls 32 GPIOs, which correspond to the Linux device nodes as follows:

| GPIO Controller | Linux Device Node | GPIO Physical Number | GPIO Logical Number |
| :-------: | :---------: | :---------: | :------------: |
| #0 | /sys/class/gpio/gpiochip480 | 0 to 31 | 480 to 511 |
| #1 | /sys/class/gpio/gpiochip448 | 32 to 63 | 448 to 479 |
| #2 | /sys/class/gpio/gpiochip416 | 64 to 95 | 416 to 447 |

For example, to operate `GPIO6`, you need:

1. Number conversion: The physical number is `6`, the corresponding logical number is `(480 + (6 - 0)) = 486`, `0` represents the start of the GPIO physical number
2. Export the GPIO, run as root user:
    ```
    echo 486 >/sys/class/gpio/export
    ```
3. Operate GPIO, set to read input or output specified value:
    ```
    cd /sys/class/gpio/gpio486
    # set input mode
    echo in >direction
    cat value
    
    # set output mode
    echo out >direction
    # output low level
    echo 0 >value
    # output high level
    echo 1 >value
    ```

There are a few things to note:
- If the GPIO has been applied by other drivers, it cannot be exported successfully, and subsequent read and write operations cannot be performed.
- Involving the multiplexing of IO functions, if the function of the pin is already GPIO, the above method can be used; otherwise, the pinctrl state must be specified in the device tree.

Generally speaking, if you want to easily operate a GPIO in the user layer, you can add it as a GPIO-LED in the device tree. Please refer to `linux-linaro-stable/arch/arm64/boot/dts/bitmain/bm1684_firefly.dtsi` for details.

