# GPIO Usage

The BM1688 chip includes 7 groups of GPIO controllers, each controlling 32 GPIOs, corresponding to the Linux device nodes as follows:

|    GPIO Controller   |      Linux Device Node      | GPIO Physical Number | GPIO Logical Number |
| :------------------: | :-------------------------: | :------------------: | :-----------------: |
|        #porta        | /sys/class/gpio/gpiochip464 |        0 to 31       |      464 to 495     |
|        #portb        | /sys/class/gpio/gpiochip432 |       32 to 63       |      432 to 463     |
|        #portc        | /sys/class/gpio/gpiochip400 |       64 to 95       |      400 to 431     |
|        #portd        | /sys/class/gpio/gpiochip368 |       96 to 127      |      368 to 399     |
|        #porte        | /sys/class/gpio/gpiochip336 |      128 to 159      |      336 to 367     |
|        #portf        | /sys/class/gpio/gpiochip304 |      160 to 191      |      304 to 335     |
| #portg（RTCSYS\_GPIO） | /sys/class/gpio/gpiochip272 |        0 to 31       |      272 to 303     |

Among them, AIO-1688JD4 supports the expansion of a set of GPIO interfaces (I2C extended GPIO):

| GPIO Controller |      Linux Device Node      | GPIO Physical Number | GPIO Logical Number |
| :-------------: | :-------------------------: | :------------------: | :-----------------: |
|      #portx     | /sys/class/gpio/gpiochip496 |        0 to 15       |      496 to 511     |

For example, to operate `porta` the `GPIO6`, the following is required:

1. Number conversion: the physical number is `6`, corresponding to the logical number `(464 + (6 - 0)) = 470`, here `0` represents the start of the GPIO physical number
2. Export this GPIO, run as root user:
   ```
   echo 470 >/sys/class/gpio/export
   ```
3. Operate GPIO, set to read input or output specified value:
   ```
   cd /sys/class/gpio/gpio470
   # 设置输入模式
   echo in >direction
   cat value

   # 设置输出模式
   echo out >direction
   # 输出低电平
   echo 0 >value
   # 输出高电平
   echo 1 >value
   ```

There are the following precautions:

* If the GPIO has already been requested by another driver, it cannot be successfully exported, and subsequent read and write operations cannot be performed.