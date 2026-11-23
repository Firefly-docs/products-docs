# Debug Usage

If you are doing U-Boot or kernel development, the USB serial adapter (short for USB to serial TTL adapter) is very useful for checking the system startup log, especially when there is no graphical desktop display.

## Hardware Connection [step]

Serial port to USB adapter, there are four pins:

- 3.3V Power, no need to connect.
- GND, Ground, connect to GND pin of the board.
- TXD, Transmit, connect to TX pin of the board.
- RXD, Receive, connect to RX pin of the board.

<Callout title="Note" type="warn">

If you encounter the problem that TX and RX cannot input and output when using other serial adapters, you can try to exchange TX and RX connections.

</Callout>

<Callout title="Note" type="warn">

EC-A3399ProC is an integrated host computer. The debug serial port is not directly exposed; you need to open the case to access the debug serial port on the internal motherboard.

</Callout>

EC-A3399ProC serial port connection diagram:

<center>

<img alt="" src="../../../modules_img/USB-TO-TTL-Serial/usb-to-ttl-serial_AIO-3399ProC.png" width="800">

</center>

## Serial Parameters [step]

EC-A3399ProC uses the following serial parameters:

- Baud rate: 1500000
- Data bit: 8
- Stop bit: 1
- Parity check: none
- Flow control: none

## Using Serial Debug [step]

<CodeBlockTabs defaultValue="Windows">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="Windows">Windows</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="Ubuntu">Ubuntu</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="Windows">

    ### Install Driver

    Download driver and install:

    - [CH340](https://www.wch.cn/downloads/CH341SER_EXE.html)
    - [PL2303](https://www.prolific.com.tw/en/portfolio-item/pl2303gl/)
    - [CP210X](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers)

    After the adapter is inserted, the system will prompt for the discovery of new hardware and initialization, and then the corresponding COM port can be found in the device manager:

    <center>

    <img alt="" src="../../../modules_img/USB-TO-TTL-Serial/debug_find_com.png" width="800">
    </center>

    ### Install Software

    Putty or SecureCRT is generally used on Windows; the free version of MobaXterm is recommended. It is a powerful terminal software, and its usage is similar to the tools above.

    [Download MobaXterm](https://mobaxterm.mobatek.net/):

    1. Select `session` as `Serial`.
    2. Modify `Serial port` to the COM port found in the device manager.
    3. Set `Speed (bsp)` to 1500000.
    4. Click `OK` button.

    <center>

    <img alt="" src="../../../modules_img/USB-TO-TTL-Serial/debug_set_MobaXterm1.png" width="800">
    </center>

    <center>

    <img alt="" src="../../../modules_img/USB-TO-TTL-Serial/debug_set_MobaXterm2.png" width="800">
    </center>

  </CodeBlockTab>

  <CodeBlockTab value="Ubuntu">

    There are many options available on Ubuntu:

    - minicom
    - picocom
    - kermit

    The following shows how to use minicom.

    ### Install minicom

    ```bash
    sudo apt-get install minicom
    ```

    After connecting the serial port line, check the serial port device file. The following example is `/dev/ttyUSB0`:

    ```console
    $ ls /dev/ttyUSB*
    /dev/ttyUSB0
    ```

    Run:

    ```console
    $ sudo minicom
    Welcome to minicom 2.7
    OPTIONS: I18n
    Compiled on Jan  1 2014, 17:13:19.
    Port /dev/ttyUSB0, 15:57:00
    Press CTRL-A Z for help on special keys
    ```

    Based on the above tips: Press `Ctrl-a` and then press `Z` again to bring up the help menu.

    Press `O` to enter the setting interface, select `Serial port setup`, then enter the letter prompted earlier, and set it as follows:

    ```text
       E -    Bps/Par/Bits       : 1500000 8N1
       F - Hardware Flow Control : No
       G - Software Flow Control : No
    ```

    <Callout title="Note" type="warn">

    `Hardware Flow Control` and `Software Flow Control` should be set to `No`, otherwise you may not be able to input.

    </Callout>

    After finishing the setting, go back to the previous menu and select `Save setup as dfl` to save as the default configuration, which will be used by default later.

  </CodeBlockTab>
</CodeBlockTabs>