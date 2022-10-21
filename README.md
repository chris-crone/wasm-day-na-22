# Wasm Day NA 2022: Build, Share, Run Wasm using Docker

Technical content from my talk with [@juntao](https://github.com/juntao) from [WasmEdge](https://wasmedge.org) at the Cloud Native Wasm Day NA 2022.

This repository gives you all the links and instructions you need to try out Docker with [WASI](https://wasi.dev) support.

## Prerequisites

To use Docker with WASI, you will need to download a technical preview build of Docker Desktop:

* [macOS Apple Silicon](https://dockr.ly/3sf56vH)
* [macOS Intel](https://dockr.ly/3VF6uFB)
* [Windows AMD64](https://dockr.ly/3ShlsP0)
* Linux Arm64 ([deb](https://dockr.ly/3TDcjRV))
* Linux AMD64 ([deb](https://dockr.ly/3TgpWH8), [rpm](https://dockr.ly/3eG6Mvp), [tar](https://dockr.ly/3yUhdCk))

> :warning: This is a technical preview build of Docker Desktop and it may not behave as you expect. Back up your containers and images before trying it.

The build has the [containerd image store](https://docs.docker.com/desktop/containerd/) enabled and it cannot be disabled.
Your existing images and containers may not be visible.

## Known issues

1. `docker-compose` may not exit cleanly when interrupted
    - Workaround: Clean up `docker-compose` processes by sending them a SIGKILL (`killall -9 docker-compose`).
1. Pushes to Hub might give an error stating `server message: insufficient_scope: authorization failed`, even after logging in using Docker Desktop
    - Workaround: Run `docker login` in the CLI

## Running a server

If you'd like to setup Docker with WASI on a server, take a look at the instructions in [server](./server).

## Show me the demos!

Click through to the [demos](./demos) folder to get started with Docker + Wasm.
