# AI Tutorial

The QCS8550 processor of the EC-A8550JD4 has a Hexagon Digital Signal Processor (DSP), which contains the Neural Processing Unit (NPU) for AI computing.

So in general, NPU and DSP both represent the same Hexagon chip.

## Aidlux Introduction

Aidlux is a complete edge AI development tool suite, it can simplify the AI environment deployment in the Qualcomm platform and help developers accelerate the deployment of AI applications.

Currently, docker is usually used to deploy the Aidlux Web Desktop for development. You can check whether a container named "aidlux" already exists on the device with the following command:

```bash
docker ps -a
```

The pre-installation and installation method of the Aidlux Web Desktop on the EC-A8550JD4 is to be supplemented.

## AI Development

Aidlite is an AI execution framework, designed to fully utilize the computing units (CPU, GPU, NPU/DSP) of the Qualcomm platform for accelerated AI model inference. AidGenSE is a generative AI HTTP service adapting to the OpenAI HTTP protocol, developers can call generative AI through HTTP and quickly integrate it into their applications.

For the detailed steps of the Aidlux Web Desktop deployment and AI development, please visit the [Aidlux Doc Center](https://rhinopi.docs.aidlux.com/en/software/).