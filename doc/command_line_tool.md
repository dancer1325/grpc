# gRPC command line tool

## Overview

* CL tool
  * comes with gRPC repository
  * requirements
    * built -- from -- source
  * next steps
    * move out to grpc-tools repository / stand alone application

## Core functionality

* allows
  * Send unary rpc
  * about metadata
    * attach
    * display received ones
  * handle common authentication to server
  * Infer request/response types -- from -- server reflection result.
  * Find the request/response types -- from a -- given proto file.
  * Read
    * proto request / text form.
    * request / wire form
      * | protobuf messages == serialized binary form
  * Display proto response | text form.
  * Write response / wire form -- to a -- file.

* should support
  * list, through server reflection,  
    * server services
    * methods
  * fine-grained auth control
    * _Example:_ use this oauth token -- to talk to the -- server
  * Send streaming rpc.

## Code location

* requirements
  * [installation instructions](../BUILDING.md)
* build -- via -- `cmake`

    ```
    $ mkdir -p cmake/build
    $ cd cmake/build
    $ cmake -DgRPC_BUILD_TESTS=ON ../..
    $ make grpc_cli
    ```

* main file is | https://github.com/grpc/grpc/blob/master/test/cpp/util/grpc_cli.cc

## Prerequisites
* TODO:

Most `grpc_cli` commands need the server to support server reflection. See
guides for
[Java](https://github.com/grpc/grpc-java/blob/master/documentation/server-reflection-tutorial.md#enable-server-reflection)
,
[C++](https://github.com/grpc/grpc/blob/master/doc/server_reflection_tutorial.md)
and
[Go](https://github.com/grpc/grpc-go/blob/master/Documentation/server-reflection-tutorial.md)

Local proto files can be used as an alternative. See instructions
[below](#Call-a-remote-method).

## Usage

### List services

`grpc_cli ls` command lists services and methods exposed at a given port

-   List all the services exposed at a given port

    ```sh
    $ grpc_cli ls localhost:50051
    ```

    output:

    ```none
    helloworld.Greeter
    grpc.reflection.v1alpha.ServerReflection
    ```

    The `localhost:50051` part indicates the server you are connecting to.

-   List one service with details

    `grpc_cli ls` command inspects a service given its full name (in the format
    of \<package\>.\<service\>). It can print information with a long listing
    format when `-l` flag is set. This flag can be used to get more details
    about a service.

    ```sh
    $ grpc_cli ls localhost:50051 helloworld.Greeter -l
    ```

    `helloworld.Greeter` is full name of the service.

    output:

    ```proto
    filename: helloworld.proto
    package: helloworld;
    service Greeter {
      rpc SayHello(helloworld.HelloRequest) returns (helloworld.HelloReply) {}
    }

    ```

### List methods

-   List one method with details

    `grpc_cli ls` command also inspects a method given its full name (in the
    format of \<package\>.\<service\>.\<method\>).

    ```sh
    $ grpc_cli ls localhost:50051 helloworld.Greeter.SayHello -l
    ```

    `helloworld.Greeter.SayHello` is full name of the method.

    output:

    ```proto
    rpc SayHello(helloworld.HelloRequest) returns (helloworld.HelloReply) {}
    ```

### Inspect message types

We can use `grpc_cli type` command to inspect request/response types given the
full name of the type (in the format of \<package\>.\<type\>).

-   Get information about the request type

    ```sh
    $ grpc_cli type localhost:50051 helloworld.HelloRequest
    ```

    `helloworld.HelloRequest` is the full name of the request type.

    output:

    ```proto
    message HelloRequest {
      optional string name = 1;
    }
    ```

### Call a remote method

We can send RPCs to a server and get responses using `grpc_cli call` command.

-   Call a unary method Send a rpc to a helloworld server at `localhost:50051`:

    ```sh
    $ grpc_cli call localhost:50051 SayHello "name: 'gRPC CLI'"
    ```

    output: `sh message: "Hello gRPC CLI"`

    `SayHello` is (part of) the gRPC method string. Then `"name: 'world'"` is
    the text format of the request proto message. For information on more flags,
    look at the comments of `grpc_cli.cc`.

-   Use local proto files

    If the server does not have the server reflection service, you will need to
    provide local proto files containing the service definition. The tool will
    try to find request/response types from them.

    ```sh
    $ grpc_cli call localhost:50051 SayHello "name: 'world'" \
      --protofiles=examples/protos/helloworld.proto
    ```

    If the proto file is not under the current directory, you can use
    `--proto_path` to specify new search roots (separated by colon on
    Mac/Linux/Cygwin or semicolon on Windows).

    Note that the tool will always attempt to use the reflection service first,
    falling back to local proto files if the service is not found. Use
    `--noremotedb` to avoid attempting to use the reflection service.

-   Send non-proto rpc

    For using gRPC with protocols other than protobuf, you will need the exact
    method name string and a file containing the raw bytes to be sent on the
    wire.

    ```bash
    $ grpc_cli call localhost:50051 /helloworld.Greeter/SayHello \
      --input_binary_file=input.bin \
      --output_binary_file=output.bin
    ```

    On success, you will need to read or decode the response from the
    `output.bin` file.
