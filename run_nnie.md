# Run NNIE Tutorial

NNIE run model instructions contained in `.wk` file. "Caffe" model can be converted to `.wk` using `nnie_mapper`. "Tensorflow" and "keras" model can convert to "caffe" model by tool ["mmdnn"](./use_mmdnn.md).

Once you generate `.wk` file, you can load it by programming. You can do some simulation with `Visual Studio` and run on chip. Before you run nnie model on chip, remember to load drivers by running:

```
cd path/to/linux/multi-core/ko
./load3559av100_multicore -i -sensor0 imx477 -sensor1 imx477
```
