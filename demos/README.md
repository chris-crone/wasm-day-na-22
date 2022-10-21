# Docker + WasmEdge demos

## Demo 1: WasmEdge

> :information_source: This demo does not use Docker.

A demo of using Wasm with Tensorflow to do image classification. Link to demo repository: https://github.com/WasmEdge/wasmedge_hyper_demo/tree/main/server-tflite

## Demo 2: Docker Compose

This demo shows how you can build a Wasm application using Docker and then run it side by side with a container.
The Wasm application is a simple Rust Web server that reads and writes data to a MySQL database.

> :information_source: Before you can follow this demo flow, you need to make sure that you're using a version of Docker Desktop that includes Wasm support.
> See the [README](../README.md) at the root of this repository.

Start by cloning the demo [repository](https://github.com/second-state/microservice-rust-mysql):

```console
$ git clone https://github.com/second-state/microservice-rust-mysql.git
Cloning into 'microservice-rust-mysql'...
remote: Enumerating objects: 75, done.
remote: Counting objects: 100% (75/75), done.
remote: Compressing objects: 100% (42/42), done.
remote: Total 75 (delta 29), reused 48 (delta 14), pack-reused 0
Receiving objects: 100% (75/75), 19.09 KiB | 1.74 MiB/s, done.
Resolving deltas: 100% (29/29), done.
```

You can then navigate into the directory and run the demo using `docker compose up`. It will take some time as downloads the MariaDB image and it builds the Wasm application:

```console
$ cd microservice-rust-mysql
$ docker compose up
[+] Running 0/1
 ⠿ server Warning                                                                                                  0.4s
[+] Building 4.8s (13/15)
...
microservice-rust-mysql-db-1      | 2022-10-19 19:54:45 0 [Note] mariadbd: ready for connections.
microservice-rust-mysql-db-1      | Version: '10.9.3-MariaDB-1:10.9.3+maria~ubu2204'  socket: '/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
```

In another terminal, you can see that we have built a new image, named `server` that contains the Wasm application:

```console
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
server       latest    2c798ddecfa1   2 minutes ago   3MB
```

You can inspect the image and see that it has been built for WASI:

```console
$ docker image inspect server | grep -A 3 "Architecture"
        "Architecture": "wasm32",
        "Os": "wasi",
        "Size": 3001146,
        "VirtualSize": 3001146,
```

You can initialize the database by calling the `/init` endpoint on the Web server:

```console
$ curl -s http://localhost:8080/init
{"status":true}% 
```

You can query the `/orders` endpoint to see what's in the database. This will return an empty list:

```console
$ curl -s http://localhost:8080/orders
[]%
```

You can then seed the database with some orders by posting the contents of the `orders.json` file and checking the `/orders` endpoint again:

```console
$ curl -s http://localhost:8080/create_orders -X POST -d @orders.json
{"status":true}% 

$ curl -s http://localhost:8080/orders | jq
[
  {
    "order_id": 1,
    "product_id": 12,
    "quantity": 2,
    "amount": 56,
    "shipping": 15,
    "tax": 2,
    "shipping_address": "Mataderos 2312"
  },
  {
    "order_id": 2,
    "product_id": 15,
    "quantity": 3,
    "amount": 256,
    "shipping": 30,
    "tax": 16,
    "shipping_address": "1234 NW Bobcat"
  },
  {
    "order_id": 3,
    "product_id": 11,
    "quantity": 5,
    "amount": 536,
    "shipping": 50,
    "tax": 24,
    "shipping_address": "20 Havelock"
  },
  {
    "order_id": 4,
    "product_id": 8,
    "quantity": 8,
    "amount": 126,
    "shipping": 20,
    "tax": 12,
    "shipping_address": "224 Pandan Loop"
  },
  {
    "order_id": 5,
    "product_id": 24,
    "quantity": 1,
    "amount": 46,
    "shipping": 10,
    "tax": 2,
    "shipping_address": "No.10 Jalan Besar"
  }
]
```

You can delete orders by performing a GET on the `/delete_order` endpoint setting the ID in the query string:

```console
$ curl -s http://localhost:8080/delete_order\?id\=2
{"status":true}%

$ curl -s http://localhost:8080/orders | jq
[
  {
    "order_id": 1,
    "product_id": 12,
    "quantity": 2,
    "amount": 56,
    "shipping": 15,
    "tax": 2,
    "shipping_address": "Mataderos 2312"
  },
  {
    "order_id": 3,
    "product_id": 11,
    "quantity": 5,
    "amount": 536,
    "shipping": 50,
    "tax": 24,
    "shipping_address": "20 Havelock"
  },
  {
    "order_id": 4,
    "product_id": 8,
    "quantity": 8,
    "amount": 126,
    "shipping": 20,
    "tax": 12,
    "shipping_address": "224 Pandan Loop"
  },
  {
    "order_id": 5,
    "product_id": 24,
    "quantity": 1,
    "amount": 46,
    "shipping": 10,
    "tax": 2,
    "shipping_address": "No.10 Jalan Besar"
  }
]
```

To stop the service and remove all data, simply hit `ctrl + c` in the terminal where you're running Compose. You can then remove everything with `docker compose down --volumes`

```console
^CGracefully stopping... (press Ctrl+C again to force)
[+] Running 2/2
 ⠿ Container microservice-rust-mysql-db-1      Stopped                                                             0.3s
 ⠿ Container microservice-rust-mysql-server-1  Stopped                                                             0.0s

 $ docker compose down --volumes
[+] Running 2/0
 ⠿ Container microservice-rust-mysql-db-1      Removed                                                             0.0s
 ⠿ Container microservice-rust-mysql-server-1  Removed                                                             0.0s
```
