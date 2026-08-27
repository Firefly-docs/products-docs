# Ubuntu minicom configuration

Press O to enter the setting interface, as follows:

```
+-----[configuration]------+
| Filenames and paths      |
| File transfer protocols  |
| Serial port setup        |
| Modem and dialing        |
| Screen and keyboard      |
| Save setup as dfl        |
| Save setup as..          |
| Exit                     |
+--------------------------+
```

Move the cursor to `Serial port setup`, press enter to enter the Serial port setup interface, then enter the letter prompted earlier, select the corresponding option, and set it as follows:

```
+-----------------------------------------------------------------------+
| A -    Serial Device      : /dev/ttyUSB0                              |
| B - Lockfile Location     : /var/lock                                 |
| C -   Callin Program      :                                           |
| D -  Callout Program      :                                           |
| E -    Bps/Par/Bits       : 1500000 8N1                               |
| F - Hardware Flow Control : No                                        |
| G - Software Flow Control : No                                        |
|                                                                       |
|    Change which setting?                                              |
+-----------------------------------------------------------------------+
```

<font color=#ff0000>**Note:** `Hardware Flow Control` and `Software Flow Control` should be set to `No`, otherwise it may not be impossible to input.</font>

After finishing the setting, go back to the previous menu and select `Save setup as dfl` to save as the default configuration, which will be used by default later.