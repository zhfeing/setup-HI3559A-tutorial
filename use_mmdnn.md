# Introduction to Mmdnn

> *Requirements: tensorflow version: 1.12.0 (with gpu); keras: 2.1.6; pytorch 1.0.0 (not tested); cuda 10.0*

> *TensorFlow parser (Tensorflow -> IR part) is an experimental product, since __the granularity of TensorFlow checkpoint graph is much finer than other platform__. We have to use graph matching-like method to retrieve operators.*

* install mmdnn
```
pip install mmdnn
```
## Keras model to IR model

1. convert keras model `*.h5` to temp files (IR model) which can be used for debugging

```
mmtoir -f keras -w <*.h5> -o converted
```
the temp files are `converted.json, converted.npy and converted.pb`

3. use [MMDNN Visualizer](http://mmdnn.eastasia.cloudapp.azure.com:8080/) to visualize `converted.json` file. 

or another way: use weight and model files separately. 
```
mmtoir -f keras -d <dst file> -n <model.json> -w <weight.h5>
```

## IR model to caffe model

1. convert IR model to caffe code
```
mmtocode -f caffe -n <model.pb> -w <weights.npy> -d <dst file>.py -dw <dst weights file.npy>
```

2. convert caffe code to caffe model
```
mmtomodel -f caffe -in <code file.py> -iw <weights file.npy> -o <caffe target>
```

