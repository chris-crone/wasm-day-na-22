# Installing Docker Engine with WASM support

This will guide you in building and setting up Docker Engine with WASI on a Linux host.
The approach will allow you to run Docker with WASI alongside your existing Docker Engine install.

Prerequisites:
- [Docker Desktop](https://docs.docker.com/desktop/install/linux-install/) or [Docker on your server](https://docs.docker.com/engine/install/#server)

## Install WasmEdge

Follow the instructions [here](https://wasmedge.org/book/en/quick_start/install.html) to install WasmEdge.

You will also need to install the containerd shim.
1. Downloading the `containerd-shim-wasmedge-v1` from their [release page](https://github.com/second-state/runwasi/releases)
2. Put it somewhere in your `$PATH` (e.g. `/usr/local/bin`)

## Building and installing dockerd

First clone and build dockerd:

```bash
git clone https://github.com/rumpl/moby.git && cd moby && git checkout wasmedge && make binary
```

We need to configure the Docker daemon to use the containerd-snapshotter feature.
If you have an existing `/etc/docker/daemon.json`, you may want to back it up:

```bash
sudo cp /etc/docker/daemon.json /etc/docker/daemon.json.bak
```

Then edit or create the `/etc/docker/daemon.json` file to activate the containerd-snapshotter feature:

```console
$ cat /etc/docker/daemon.json
{
  "features": {
    "containerd-snapshotter": true
  }
}
```

Finally start the Docker daemon:

```bash
sudo sh -c "LD_LIBRARY_PATH=$HOME/.wasmedge/lib ./bundles/binary-daemon/dockerd -D -H unix:///tmp/docker.sock --data-root /tmp/root --pidfile /tmp/docker.pid"
```

## Run

As we're running this daemon alongside the other, you will need to create a docker context.
You can do this using the following command. Note that it points to the `docker.sock` that we specified above:

```bash
docker context create wasm --docker "host=unix:///tmp/docker.sock"
```

You can now run Wasm workloads by setting the `--runtime` flag to tell the Docker Engine to use the Wasm shim that we installed:

```console
$ docker --context wasm run --runtime=io.containerd.wasmedge.v1 rumpl/wasmtest echo 'hello from wasm'
Unable to find image 'rumpl/wasmtest:latest' locally
7cbb9105cdaf: Download complete 
dce2b2760aba: Download complete 
004a1e2d62d7: Download complete 
WARNING: The requested image's platform (wasi/wasm) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
hello from wasm
exiting
```

To avoid the platform warning, you can use the `--platform` flag and set `wasi/wasm32`:

```console
$ docker run --runtime=io.containerd.wasmedge.v1 --platform wasi/wasm32 rumpl/wasmtest echo 'hello from wasm'
exiting
hello from wasm
```

To avoid needing to set the `--context` flag, you can switch the context:

```bash
docker context use wasm
```

Once you switch to the `wasm` context, you can run `docker compose up` in the [demos](../demos)

## Reset

Stop the running Docker daemon by sending it a SIGINT.

Restore your backup of your daemon configuration if you made one:

```bash
sudo cp /etc/docker/daemon.json.bak /etc/docker/daemon.json
```

Reconfigure your Docker context to the default:

```bash
docker context use default
```
