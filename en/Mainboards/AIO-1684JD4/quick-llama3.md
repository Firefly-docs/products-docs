# Quickly deploy Llama3

To facilitate a quick experience with Llama3, Firefly team has provided an out-of-the-box rapid deployment package, which can be obtained from [Download Center].

## Deployment process

Transfer the deployment package to the AIO-1684JD4 machine, using the `scp` command is recommended:

1. Run the `ifconfig` command on AIO-1684JD4 to view the IP address.
2. Run the `scp` command to transmit the deployment package, for example: `scp llama3.tar linar@ <aibox-ip-addr>:~/`, where <aibox-ip-addr> is the IP address of AIO-1684JD4.

Decompress the deployment package: `tar -xvf ./llama3.tar`.

One command to deploy Llama3: `./talk_to_llama3.sh`.

Note: Networking is not required during the deployment. The first deployment of Llama3 requires a long period of time (3-5 minutes) for software installation.

[Download Center]: https://en.t-firefly.com/doc/download/156.html