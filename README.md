# Guidelines for building tensorflow C++ libraries on FreeBSD 11.X and 12.X #

Following guidelines have been tested on FreeBSD 11.1 and FreeBSD 11.2
Do not hesitate to report any problem or improvements

## Prerequisites ##

### FreeBSD packages dependencies ###

In addition to typical packages required to build binaries, you will need clang and bazel :

`pkg install clang` <br>
`pkg install bazel`


### Google Protobuf ###

You need a specific version of Protobuf to compile tensorflow.
This depends of the tensorflow version.
 
As for TF 1.10, Protobuf v3.6.1 is required.

`Download https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protobuf-all-3.6.1.tar.gz` <br>
`tar xvf protobuf-all-3.6.1.tar.gz` <br>
`cd protobuf-3.6.1` <br>
`./configure` <br>
`make` <br>
`make install` 


## TensorFlow compilation ##

### Download sources ###

`git clone --recursive https://github.com/tensorflow/tensorflow.git`

### Switch to the required version ###

`cd tensorflow` <br>
`git checkout -b r1.10` <br>
bazel fetch @grpc//:gpr_base <br>

### Configure required features ###

`./configure`


### Apply patches for FreeBSD ###

This is not needed as we won't compile Python <br>
`echo > tensorflow/requirements.txt`

#Adapt paths to your bazel tmp directory <br>
`patch ./tensorflow/core/platform/env.cc < ../patches/env.cc.patch` <br>
`patch ./tensorflow/core/platform/posix/posix_file_system.cc < ../patches/posix_file_system.cc.patch` <br>
`sed -i '' -E 's/if cpu_value in \("Linux"\)\:/if cpu_value in ("Linux","FreeBSD"):/g' third_party/gpus/rocm_configure.bzl` <br>

### Compile ###

You need to adapt --copt (mavx, sse4.2...) to your processor
`bazel build -c opt --copt=-mavx --copt=-msse4.2 --cxxopt=-std=c++11 --host_linkopt=-lexecinfo --config=noaws --config=nogcp --config=nohdfs --config=noignite --config=nokafka --define=using_rocm=false //tensorflow:libtensorflow_cc.so //tensorflow:libtensorflow_framework.so`

Tensorflow libraries are available here: 
 - ./bazel-bin/tensorflow/libtensorflow_framework.so 
 - ./bazel-bin/tensorflow/libtensorflow_cc.so 
 
 



