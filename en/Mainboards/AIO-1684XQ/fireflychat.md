# FireflyChat

FireflyChat is a graphical application platform for LLM developed by the Firefly team. Users can quickly experience the actual use of various large models on the visual WebUI provided by FireflyChat.

![](../../../bm1684_img/fireflychat.png)

## Installation

You need to obtain the required installation package from the [Download Center] to install FireflyChat. There are two installation packages, which are `fireflychat*.deb` and `libfirefly-ai*.deb`.

After obtaining the installation package, transfer it to AIO-1684XQ and run the following command to install it:

```bash
# Install venv
sudo apt update
sudo apt install python3-venv

# Install FireflyChat
sudo dpkg -i libfirefly-ai*.deb
sudo dpkg -i fireflychat*.deb
```

After installing FireflyChat, you need to import the model file to FireflyChat to use it properly.

## Model import

### LLM

Users can obtain the model files (.bmodel) provided by the Firefly team from the LLM model resource in the [Download Center].

After obtaining the model file, transfer it to AIO-1684XQ and run the following command to import it:

```bash
# Move the model file location, where <model_path> replace with the actual model file path
sudo mv <model_path> /models/LLM/
```

### Embedding model

Users can obtain the model files (.bmodel) provided by the Firefly team from the embedding model resource in the [Download Center].

After obtaining the model file, transfer it to AIO-1684XQ and run the following command to import it:

```bash
# Move the model file location, where <model_path> replace with the actual model file path
sudo mv <model_path> /models/embedding/
```

### Restart FireflyChat

After adding or modifying the required model files, you need to restart FireflyChat for the new model to take effect:

```bash
sudo systemctl restart FireflyChat
```

## Software usage

### WebUI usage (Main usage methods)

FireflyChat will run automatically after it is installed successfully. You can directly access FireflyChat by entering `<ip_address>:7860` in the browser.

Note: <ip_address> must be replaced with the actual IP address, for example: `172.16.11.18:7860`.

### Software background management

FireflyChat is an application registered as the systemd service. You can control the running status of FireflyChat by running the following commands:

```bash
# Stop FireflyChat
sudo systemctl stop FireflyChat

# Restart FireflyChat
sudo systemctl restart FireflyChat

# Disable the automatic startup of FireflyChat
sudo systemctl disable FireflyChat

# Viewing run logs
sudo journalctl -u FireflyChat
```

[Download Center]: https://en.t-firefly.com/doc/download/195.html