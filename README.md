# SpikingJelly
![GitHub last commit](https://img.shields.io/github/last-commit/fangwei123456/spikingjelly) [![Documentation Status](https://readthedocs.org/projects/spikingjelly/badge/?version=latest)](https://spikingjelly.readthedocs.io/zh_CN/latest) [![PyPI](https://img.shields.io/pypi/v/spikingjelly)](https://pypi.org/project/spikingjelly) [![PyPI - Python Version](https://img.shields.io/pypi/pyversions/spikingjelly)](https://pypi.org/project/spikingjelly) ![repo size](https://img.shields.io/github/repo-size/fangwei123456/spikingjelly)

English | [中文](./README_cn.md)

![demo](./docs/source/_static/logo/demo.png)

SpikingJelly is an open-source deep learning framework for Spiking Neural Network (SNN) based on [PyTorch](https://pytorch.org/).

The documentation of SpikingJelly is written in both English and Chinese: https://spikingjelly.readthedocs.io.

- [Installation](#installation)
- [Build SNN In An Unprecedented Simple Way](#build-snn-in-an-unprecedented-simple-way)
- [Fast And Handy ANN-SNN Conversion](#fast-and-handy-ann-snn-conversion)
- [CUDA-Enhanced Neuron](#cuda-enhanced-neuron)
- [Device Supports](#device-supports)
- [Neuromorphic Datasets Supports](#neuromorphic-datasets-supports)
- [Tutorials](#Tutorials)
- [Publications and Citation](#publications-and-citation)
- [Contribution](#contribution)
- [About](#about)

## Installation

Note that SpikingJelly is based on PyTorch. Please make sure that you have installed PyTorch before you install SpikingJelly.

The odd version number is the developing version, which is updated with GitHub/OpenI repository. The even version number is the stable version and available at PyPI.

**Install the last stable version (0.0.0.0.8) from** [**PyPI**](https://pypi.org/project/spikingjelly/):

```bash
pip install spikingjelly
```

**Install the latest developing version from the source codes**:

From [GitHub](https://github.com/fangwei123456/spikingjelly):
```bash
git clone https://github.com/fangwei123456/spikingjelly.git
cd spikingjelly
python setup.py install
```
From [OpenI](https://git.openi.org.cn/OpenI/spikingjelly)：
```bash
git clone https://git.openi.org.cn/OpenI/spikingjelly.git
cd spikingjelly
python setup.py install
```
## Build SNN In An Unprecedented Simple Way

SpikingJelly is user-friendly. Building SNN with SpikingJelly is as simple as building ANN in PyTorch:

```python
class Net(nn.Module):
    def __init__(self, tau=100.0, v_threshold=1.0, v_reset=0.0):
        super().__init__()
        # Network structure, a simple two-layer fully connected network, each layer is followed by LIF neurons
        self.fc = nn.Sequential(
            nn.Flatten(),
            nn.Linear(28 * 28, 14 * 14, bias=False),
            neuron.LIFNode(tau=tau, v_threshold=v_threshold, v_reset=v_reset),
            nn.Linear(14 * 14, 10, bias=False),
            neuron.LIFNode(tau=tau, v_threshold=v_threshold, v_reset=v_reset)
        )

    def forward(self, x):
        return self.fc(x)
```

This simple network with a Poisson encoder can achieve 92% accuracy on MNIST test dataset. Read [the tutorial of clock driven](https://spikingjelly.readthedocs.io/zh_CN/latest/tutorial_en.clock_driven.html) for more details. You can also run this code in Python terminal for training on classifying MNIST:

```python
>>> import spikingjelly.clock_driven.examples.lif_fc_mnist as lif_fc_mnist
>>> lif_fc_mnist.main()
```

Read [spikingjelly.clock_driven.examples](https://spikingjelly.readthedocs.io/zh_CN/latest/spikingjelly.clock_driven.examples.html) to explore more advanced networks!

## Fast And Handy ANN-SNN Conversion

SpikingJelly implements a relatively general ANN-SNN Conversion interface. Users can realize the conversion through PyTorch or ONNX packages. What's more, users can customize the conversion module to add to the conversion. 

```python
class ANN(nn.Module):
    def __init__(self):
        super().__init__()
        self.network = nn.Sequential(
            nn.Conv2d(1, 32, 3, 1),
            nn.BatchNorm2d(32, eps=1e-3),
            nn.ReLU(),
            nn.AvgPool2d(2, 2),

            nn.Conv2d(32, 32, 3, 1),
            nn.BatchNorm2d(32, eps=1e-3),
            nn.ReLU(),
            nn.AvgPool2d(2, 2),

            nn.Conv2d(32, 32, 3, 1),
            nn.BatchNorm2d(32, eps=1e-3),
            nn.ReLU(),
            nn.AvgPool2d(2, 2),

            nn.Flatten(),
            nn.Linear(32, 10),
            nn.ReLU()
        )

    def forward(self,x):
        x = self.network(x)
        return x
```

This simple network with analog encoding can achieve 98.51% accuracy after converiosn on MNIST test dataset. Read [the tutorial of ann2snn](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven/5_ann2snn.html) for more details. You can also run this code in Python terminal for training on classifying MNIST using converted model:

```python
>>> import spikingjelly.clock_driven.ann2snn.examples.cnn_mnist as cnn_mnist
>>> cnn_mnist.main()
```

## CUDA-Enhanced Neuron

SpikingJelly provides two backends for multi-step neurons (read [Tutorials](#Tutorials) for more details). You can use the user-friendly `torch` backend for easily codding and debugging, and use `cupy` backend for faster training speed.

The followed figure compares execution time of two backends of Multi-Step LIF neurons (`float32`):

<img src="./docs/source/_static/tutorials/clock_driven/11_cext_neuron_with_lbl/exe_time_fb.png" alt="exe_time_fb"  />

`float16` is also provided by the `cupy` backend and can be used in [automatic mixed precision training](https://pytorch.org/docs/stable/notes/amp_examples.html).

To use the `cupy` backend, please install [CuPy](https://docs.cupy.dev/en/stable/install.html). Note that the `cupy` backend only supports GPU, while the `torch` backend supports both CPU and GPU.

## Device Supports

-   [x] Nvidia GPU
-   [x] CPU

As simple as using PyTorch.

```python
>>> net = nn.Sequential(nn.Flatten(), nn.Linear(28 * 28, 10, bias=False), neuron.LIFNode(tau=tau))
>>> net = net.to(device) # Can be CPU or CUDA devices
```

## Neuromorphic Datasets Supports
SpikingJelly includes the following neuromorphic datasets:

| Dataset        | Source                                                       |
| -------------- | ------------------------------------------------------------ |
| ASL-DVS        | Graph-based Object Classification for Neuromorphic Vision Sensing |
| CIFAR10-DVS    | CIFAR10-DVS: An Event-Stream Dataset for Object Classification |
| DVS128 Gesture | A Low Power, Fully Event-Based Gesture Recognition System    |
| N-Caltech101   | Converting Static Image Datasets to Spiking Neuromorphic Datasets Using Saccades |
| N-MNIST        | Converting Static Image Datasets to Spiking Neuromorphic Datasets Using Saccades |

Users can use both the origin events data and frames data integrated by SpikingJelly:

```python
from spikingjelly.datasets.dvs128_gesture import DVS128Gesture
root_dir = 'D:/datasets/DVS128Gesture'
event_set = DVS128Gesture(root_dir, train=True, data_type='event')
frame_set = DVS128Gesture(root_dir, train=True, data_type='frame', frames_number=20, split_by='number')
```
More datasets will be included in the future.

If some datasets' download link are not available for some users, the users can download from the OpenI mirror:

https://git.openi.org.cn/OpenI/spikingjelly/datasets?type=0

All datasets saved in the OpenI mirror are allowable by their licence or authors' agreement.

## Tutorials

SpikingJelly provides elaborate tutorials. Here are some of tutorials:

| Figure                                                       | Tutorial                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![t0](./docs/source/_static/tutorials/clock_driven/0_neuron/0.png) | [Neurons](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/0_neuron.html) |
| ![t2](./docs/source/_static/tutorials/clock_driven/2_encoding/5.png) | [Encoder](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/2_encoding.html) |
| ![t3](./docs/source/_static/tutorials/clock_driven/3_fc_mnist/2d_heatmap.png) | [Use single-layer fully connected SNN to identify MNIST](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/3_fc_mnist.html) |
| ![t4](./docs/source/_static/tutorials/clock_driven/4_conv_fashion_mnist/y10.png) | [Use convolutional SNN to identify Fashion-MNIST](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/4_conv_fashion_mnist.html) |
| ![t5](./docs/source/_static/tutorials/clock_driven/5_ann2snn/2.png) | [ANN2SNN](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/5_ann2snn.html) |
| ![t6](./docs/source/_static/tutorials/clock_driven/6_dqn_cart_pole/512@66.gif) | [Reinforcement Learning: Deep Q Learning](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/6_dqn_cart_pole.html) |
| ![t10](./docs/source/_static/tutorials/clock_driven/10_propagation_pattern/layer-by-layer.png) | [Propagation Pattern](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/10_propagation_pattern.html) |
| ![t13](./docs/source/_static/tutorials/clock_driven/13_neuromorphic_datasets/dvsg.gif) | [Neuromorphic Datasets Processing](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/13_neuromorphic_datasets.html) |
| ![t14](./docs/source/_static/tutorials/clock_driven/14_classify_dvsg/network.png) | [Classify DVS128 Gesture](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/14_classify_dvsg.html) |
| ![t15](./docs/source/_static/tutorials/clock_driven/15_recurrent_connection_and_stateful_synapse/SRNN_example.png) | [Recurrent Connections and Stateful Synapses](https://spikingjelly.readthedocs.io/zh_CN/latest/clock_driven_en/15_recurrent_connection_and_stateful_synapse.html) |

Other tutorials that are not listed here are also available at the document https://spikingjelly.readthedocs.io.

## Publications and Citation

Publications using SpikingJelly are recorded in [Publications](./publications.md). If you use SpikingJelly in your paper, you can also add it to this table by pull request.

If you use SpikingJelly in your work, please cite it as follows:

```
@misc{SpikingJelly,
	title = {SpikingJelly},
	author = {Fang, Wei and Chen, Yanqi and Ding, Jianhao and Chen, Ding and Yu, Zhaofei and Zhou, Huihui and Tian, Yonghong and other contributors},
	year = {2020},
	howpublished = {\url{https://github.com/fangwei123456/spikingjelly}},
	note = {Accessed: YYYY-MM-DD},
}
```

## Contribution

You can read the issues and get the problems to be solved and latest development plans. We welcome all users to join the discussion of development plans, solve issues, and send pull requests.

Not all API documents are written in both English and Chinese. We welcome users to complete translation (from English to Chinese, or from Chinese to English).

## About

[Multimedia Learning Group, Institute of Digital Media (NELVT), Peking University](https://pkuml.org/) and [Peng Cheng Laboratory](http://www.szpclab.com/) are the main developers of SpikingJelly.

![PKU](./docs/source/_static/logo/pku.png)![PCL](./docs/source/_static/logo/pcl.png)

The list of developers can be found [here](https://github.com/fangwei123456/spikingjelly/graphs/contributors).

