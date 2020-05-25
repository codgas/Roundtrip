# Roundtrip

![model](https://github.com/kimmo1019/Roundtrip/blob/master/model.jpg)


Roundtrip is a deep generative neural density estimator which exploits the advantage of GANs for generating samples and estimates density by either importance sampling or Laplace approximation. This repository provides source code and instructions for reproducing results in our [paper](https://arxiv.org/abs/2004.09017), including simulation data and real data. Note that the supplementary file of Roundtrip paper is also provided in this repository.

## Table of Contents

- [Requirements](#Requirements)
- [Install](#install)
- [Reproduction](#reproduction)
	- [Simulation Data](#simulation-data)
    - [Real Data](#real-data)
        - [UCI Datasets](#uci-datasets)
        - [Image Datasets](#image-datasets)
    - [Outlier Detection](#outlier-detection)
- [Contact](#contact)
- [Cite](#Cite)
- [License](#license)

## Requirements

- TensorFlow=1.13.1

## Install

Roundtrip can be downloaded by
```shell
git clone https://github.com/kimmo1019/Roundtrip
```
Software has been tested on a Linux (Centos 7) and a MacOS platform with Python2.7.

## Reproduction

This section provides instructions on how to reproduce results in the original paper.

### Simulation data

We tested Roundtrip on three types of simulation datasets. (1) Indepedent Gaussian mixture. (2) 8-octagon Gaussian mixture. (3) Involute.

The main python script `main_density_est.py` is used for implementing Roundtrip. Model architecture for Roundtrip can be find in `model.py`. Data loader or data sampler can be find in `util.py`.

Taking the (1) for an example, one can run the following commond to train a Roundtrip model with indepedent Gaussian mixture data.

```shell
CUDA_VISIBLE_DEVICES=0 python main_density_est.py  --dx 2 --dy 2 --train True --data indep_gmm --epochs 100 --cv_epoch 30 --patience 5
[dx]  --  dimension of latent space
[dy]  --  dimension of observation space
[train]  --  whethre use train mode
[data]  --  dataset name
[epochs] -- maximum training epoches
[cv_epoch] -- epoch where (cross) validation begins
[patience] -- patience for early stopping
```
After training the model, you will have three part of outputs, which are marked by a unique timestamp `YYYYMMDD_HHMMSS`. This timestamp records the exact time when you run the script.

 1) `log` files and estimated density can be found at folder `data/density_est_YYYYMMDD_HHMMSS_indep_gmm_x_dim=2_y_dim=2_alpha=10.0_beta=10.0`.
 
 2) Model weights will be saved at folder `checkpoint/density_est_YYYYMMDD_HHMMSS_indep_gmm_x_dim=2_y_dim=2_alpha=10.0_beta=10.0`. 
 
 3) The training loss curves were recorded at folder `graph/density_est_YYYYMMDD_HHMMSS_indep_gmm_x_dim=2_y_dim=2_alpha=10.0_beta=10.0`, which can be visualized using TensorBoard.

 Next, we want to visulize the estimated density on a 2D region. One can then run the following script. 

 ```shell
 CUDA_VISIBLE_DEVICES=0 python results_analyze.py --dx 2 --dy 2 --timestamp YYYYMMDD_HHMMSS --data indep_gmm --epoch epoch
 [YYYYMMDD_HHMMSS] --  timestamp in the last training step
 [epoch] -- epoch for loading model weights
 ```

 we suggest to use the epoch recorded in the last line of the `log_test.txt` file in the output part 1). Then the estimated density (.png) on a 2D grid region will be saved in the same data folder `data/density_est_YYYYMMDD_HHMMSS_indep_gmm_x_dim=2_y_dim=2_alpha=10.0_beta=10.0`. 

 It also easy to implement Roundtrip with other two simulation datasets by changing the `data`.

- 8-octagon Gaussian mixture
    Model training:
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est.py  --dx 2 --dy 2 --train True --data eight_octagon_gmm --epochs 300 --cv_epoch 200 --patience 5
    ```
    Density esitmation on a 2D grid region:
    ```shell
    CUDA_VISIBLE_DEVICES=0 python results_analyze.py --dx 2 --dy 2 --timestamp YYYYMMDD_HHMMSS --data eight_octagon_gmm --epoch epoch
    ```
- involute
    Model training:
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est.py  --dx 2 --dy 2 --train True --data involute --epochs 300 --cv_epoch 200 --patience 5
    ```
    Density esitmation on a 2D grid region:
    ```shell
    CUDA_VISIBLE_DEVICES=0 python results_analyze.py --dx 2 --dy 2 --timestamp YYYYMMDD_HHMMSS --data involute --epoch epoch
    ```

### Real Data

Next, we tested Roundtrip on different types of real data including five datasets from UCI machine learning repository and two image datasets. We provided freely public access to all related datasets (UCI datasets, image datasets, and OODS datasets), which can be download from a [zenodo repository](https://zenodo.org/record/3748270#.XpFvgdNKhTY). All you need is to download the corresponding dataset (e.g., `AreM.tar.gz`), uncompress the data under `data` folder.


#### UCI Datasets

The original UCI datasets were from [UCI machine learning repository](http://archive.ics.uci.edu/ml/datasets.php). As the real data has no groud truth for density, we evaluate Roundtrip by calculating the average log likelihood on the test data. Similar to the simulation data, we take `AreM` dataset for an example, one can directly run

```shell
CUDA_VISIBLE_DEVICES=0 python main_density_est.py  --dx 3 --dy 6 --train True --data uci_AReM --epochs 300 --cv_epoch 20 --patience 10 --use_cv True
```
Note that all the dataset from UCI machine learning repository will be added a prefix `uci_` to the data name. The average log likelihood and stantard deviation can be found in `log_test.txt` under data folder `data/density_est_YYYYMMDD_HHMMSS_uci_AreM_x_dim=2_y_dim=2_alpha=10.0_beta=10.0`.

We also provide scripts for implementing Roundtrip with other UCI dataset.

- CASP
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est.py  --dx 5 --dy 9 --train True --data uci_CASP --epochs 300 --cv_epoch 20 --patience 10 --use_cv True
    ```
- HEPMASS
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est.py  --dx 8 --dy 21 --train True --data uci_HEPMASS --epochs 300 --cv_epoch 20 --patience 10 --use_cv True
    ```
- BANK
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est.py  --dx 8 --dy 17 --train True --data uci_BANK --epochs 300 --cv_epoch 20 --patience 10 --use_cv True
    ```
- YPMSD
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est.py  --dx 20 --dy 90 --train True --data uci_YPMSD --epochs 300 --cv_epoch 20 --patience 10 --use_cv True
    ```

#### Image Datasets

MNIST and CIFAR-10 were used in our study. Unlike previous experiments, we focus on conditional density estimation where a ont-hot encoded class label will be introduced to the networks as an additional input.

Download data from [zenodo repository](https://zenodo.org/record/3748270#.XpFvgdNKhTY) and uncompress the two datasets under `data` folder.

One can run the conditional image generation and conditional denstiy estimation simultaneously through the following script.

- MNIST

    Model training
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est_img.py  --dx 100 --dy 784 --train True --data mnist --epochs 100 --cv_epoch 50 --patience 5
    ```
    Model test
    ```shell
    python results_analyze.py --data mnist --path path
    [path] -- path to the frist part of outputs (e.g., data/density_est_YYYYMMDD_HHMMSS_mnist_x_dim=100_y_dim=784_alpha=10.0_beta=10.0)
    ```

- CIFAR-10

    Model training
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est_img.py  --dx 100 --dy 3072 --train True --data cifar10 --epochs 1000 --cv_epoch 500 --patience 5
    ```
    Model test
    ```shell
    python results_analyze.py --data cifar10 --path path
    ```
After model test, the generated images can be found in the first part of outputs.

### Outlier Detection

We introduced three outlier detection datasets (Shuttle, Mammography, and ForestCover) from [ODDS library](http://odds.cs.stonybrook.edu/). Download the three datasets (`ODDS.tar.gz`) from the [zenodo repository](https://zenodo.org/record/3748270#.XpFvgdNKhTY). Uncompress it under the `data` folder.

One can run the following commonds to train a Roundtrip model and evaluate by precision at K.

- Shuttle

    Model training
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est.py  --dx 3 --dy 9 --train True --data odds_Shuttle --epochs 300 --cv_epoch 30 --patience 10
    ```
    Model evaluation
    ```shell
    python results_analyze.py --data odds_Shuttle --epoch epoch --path path
    [epoch] -- epoch for loading model weights (e.g., epoch recorded in the last line in log_test.txt)
    [path] -- path to the frist part of outputs (e.g., data/density_est_YYYYMMDD_HHMMSS_odds_Shuttle_x_dim=3_y_dim=9_alpha=10.0_beta=10.0)
    ```
- Mammography

    Model training
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est.py --dx 3 --dy 6 --train True --data odds_Mammography --epochs 300 --cv_epoch 30 --patience 10
    ```
    Model evaluation
    ```shell
    python results_analyze.py --data odds_Mammography --epoch epoch --path path
    ```
- ForestCover

    Model training
    ```shell
    CUDA_VISIBLE_DEVICES=0 python main_density_est.py --dx 4 --dy 10 --train True --data odds_ForestCover --epochs 300 --cv_epoch 30 --patience 10
    ```
    Model evaluation
    ```shell
    python results_analyze.py --data odds_ForestCover --epoch epoch  --path path
    ```
The precision at K of Roundtrip, One-class SVM and Isolation Forest will be calculated and printed.

## Contact

Roundtrip has various downstream applications including unsupervised learning, likelihood-free Bayesian inference and sequential Markov chain Monte Carlo (MCMC). We are working on some of the applications now, always open to cooperation opportunities. If you're interested, do not hesitate to contact me.

Also Feel free to open an issue in Github or contact `liu-q16@mails.tsinghua.edu.cn` if you have any problem in Roundtrip.

## Cite

If you use Roundtrip in your research, please consider citing our paper 
```
@misc{liu2020roundtrip,
    title={Roundtrip: A Deep Generative Neural Density Estimator},
    author={Qiao Liu and Jiaze Xu and Rui Jiang and Wing Hung Wong},
    year={2020},
    eprint={2004.09017},
    archivePrefix={arXiv},
    primaryClass={cs.LG}
}
```

## License

This project is licensed under the MIT License - see the LICENSE.md file for details


