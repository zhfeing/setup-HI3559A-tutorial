# Setup HI3559A tutorial

## Setup NFS and TFTP

Read [use_nfs](./use_nfs.txt) for setup nfs and [setup-tftp](./setup-tftp.txt) for setup tftp

## Setup HI3559A develop SDK

Read [setup_Hi3559A_steps.md](./setup_Hi3559A_steps.md)

> Hint:
> there are 2 toolsets for compile. One is `aarch64` for apps running on embedded system, another is `arm-none` which only compiles programmes running without operation system and so `arm-none` is not recommanded for compiling your programmes.

## Compile SSH for Hi3559A

Read [setup_ssh.md](./setup_ssh) for install _ssh_ on Hi3559A board.

## Other Environment

You should also prepare "OpenCV", "Caffe", "Tensorflow", "Keras" on your computer.

## Run Your Programmes

* Read [setup_nnie_mapper.md](./setup_nnie_mapper.md) to setup nnie compiler.

* Read [run_nnie.md](./run_nnie.md) to know how to run "deep learning models" on Hi3559A.
