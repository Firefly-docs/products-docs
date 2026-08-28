# FAQs

## What are the default login credentials?

The default Debian user name and password are both `firefly`. Run `sudo -s` when administrator privileges are required.

## How do I verify that the RK182X module is detected?

Check the module connection and power supply first, then follow the environment checks and examples in [RK182X platform](ai_rk182x.md).

## The board cannot enter USB upgrade mode

Set the `USB SEL` switch to `1`, connect the OTG port with a reliable Type-A data cable, power off the board, and use the `MaskRom` key to enter MaskRom mode. Then check whether the upgrade tool detects the board.