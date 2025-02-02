# When Counting Meets HMER: Counting-Aware Network for Handwritten Mathematical Expression Recognition

## 目录

- [1. 简介]()

- [2. 数据集和复现精度]()

- [3. 准备数据与环境]()
    - [3.1 准备环境]()
    - [3.2 准备数据]()
    - [3.3 准备模型]()
    
- [4. 开始使用]()
    - [4.1 模型训练]()
    - [4.2 模型评估]()
    - [4.3 模型预测]()
    - [4.4 查看训练、评估日志]()
 
- [5. 模型推理部署]()
    - [5.1 基于Inference的推理]()
    
- [6. 自动化测试脚本]()

- [7. LICENSE]()

- [8. 参考链接与文献]()

    

## 1. 简介

Counting-Aware Network（CAN）是2022年ECCV会议收录的手写数学公式识别新算法，其主要创新点如下：

- 设计了多尺度计数模块（Multi-Scale Counting Module，MSCM）计数每一个符号的出现次数，从而提升检测的准确率；
- 设计了结合计数的注意力解码器：使用位置编码表征特征图中不同空间位置，加强模型对于空间位置的感知。

![image-20220905140634676](images/architechture.png)

**论文:** [When Counting Meets HMER: Counting-Aware Network for Handwritten Mathematical Expression Recognition](https://arxiv.org/abs/2207.11463)

**参考repo:** [https://github.com/LBH1024/CAN](https://github.com/LBH1024/CAN)


**目录说明**
```
--CAN
    --alignment                  # 对齐相关代码，删除后为完整复现代码
        --step1-5
            --data               # 存放假数据（fake_data)和网络权重
            --lite_data          # 轻量数据集（16张，用于数据读取对齐）
            --can_paddle         # paddle 复现代码
            --can_ref            # torch 实现参考代码
            01_test_forward.py   # 前向对齐主函数
            02_test_data.py      # 数据读取对齐主函数
            03_test_metric.py    # 评估指标对齐主函数
            04_test_loss.py      # 损失函数对齐主函数
            05_test_backward.py  # 反向对齐主函数
            config.yaml          # 用于对齐的配置文件，包含路径等信息
            torch2paddle.py      # 权重转换函数（torch权重转换为paddle）
            utilities.py         # 常用方法
        --result                 # 对齐结果
        README.md		 # 对齐相关说明
    --checkpoints                # 存放训练保存的模型文件
    --images                     # 存放仓库显示图片
    --logs                       # 存放tensorboard日志
    --models                     # 模型定义
    --paddlevision               # 数据集加载、优化器等工具函数
        --datasets               # 数据集加载相关工具函数，数据集存放文件夹
        --metrics                # 损失标准化函数
        --optimizer              # 优化器、学习率相关工具函数
    --scripts
        --train.sh
        --eval.sh
        --infer.sh
        --export.sh
    --test_images                # 存放小数据集
	--lite_data.tar
    --test_model		 # 存放推理、预测相关文件
    --test_tipc
    --tools                      # 训练、评估、预测、推理等函数
        --train.py               # 训练、评估主函数
        --eval.py                # 训练、评估功能函数
        --infer.py               # 推理主函数
        --export.py              # 导出静态模型主函数
        --predict.py             # 预测主函数
    --utils                      # 通用工具函数
    analysis.md			 # 复现实验分析
    config.yaml			 # 训练、评估、模型使用等相关参数配置
    requirements.txt		 # 项目所需相关python工具包
```

## 2. 数据集和复现精度

本repo使用的数据集为 CROHME，内容为手写公式图片和对应的识别序列。

格式如下：

- 数据集大小：CROHME共包含8884个样本(其中训练样本8835），识别序列由111类符号组成。
- 数据集下载链接：[百度网盘](https://pan.baidu.com/share/init?surl=qUVQLZh5aPT6d7-m6il6Rg)，提取码1234。
- 数据格式：图像数据存在pkl文件中，符号序列存在txt文本中，请根据对应文件读取方法进行读取。

| 模型      | 参考精度 | 复现精度 | 下载链接 |
|:---------:|:------:|:----------:|:----------:|
| CAN | 57.00 | 51.72   | [预训练模型](https://pan.baidu.com/s/1bWG8UNK_GA9UxXkZ4RD7XA) 提取码：n5ea    [Inference模型](https://pan.baidu.com/s/1KU8NR0oCVn_pWD6wYm28AA) 提取码：ipz9    [日志](https://pan.baidu.com/s/18G-dXlU3b1ja014wQiqlag) 提取码：ohu2    [预测模型](https://pan.baidu.com/s/1aUKUAWzRMbnDYxjHGRD_ZA) 提取码：em1l。

参考repo使用Adadelta优化器训练模型，由于torch和paddle对于Adadelta的底层实现存在差异，导致使用paddle的Adadelta训练模型难以实现参考精度，并且学习过程出现困难，训练多次震荡。具体实验分析，见[analysis.md](https://github.com/Lllllolita/CAN_Paddle/blob/master/analysis.md)，以及[训练、验证tensorboard日志](https://pan.baidu.com/s/1prO4DRLq2T99cDvdSGTumQ)，提取码：p6he。日志提供了基于torch和paddle，使用Adadelta优化器的训练、验证日志曲线。

因此，本repo使用Adadelta和SGD两种优化器训练模型，并与参考repo使用SGD优化器训练的结果进行对比。最终结果表明，同样使用SGD训练，基于相同的参数初始化方式、固定随机种子、使用相同的训练参数调整策略，paddle优于torch精度，二者相差0.61%。具体实验分析，见[analysis.md](https://github.com/Lllllolita/CAN_Paddle/blob/master/analysis.md)，以及[训练、验证tensorboard日志](https://pan.baidu.com/s/1prO4DRLq2T99cDvdSGTumQ)，提取码：p6he。日志提供了基于torch和paddle，使用SGD优化器的训练、验证日志曲线。

本repo默认设置为基于SGD训练，初始学习率为0.01，可在[config.yaml](https://github.com/Lllllolita/CAN_Paddle/blob/master/config.yaml)中，修改optimizer为Adadelta，lr为1，以进行Adadelta的实验验证。

## 3. 准备数据与环境

### 3.1 准备环境

* 下载代码

```bash
git clone https://github.com/Lllllolita/CAN_Paddle.git
cd CAN_paddle
```

* 安装paddlepaddle

```bash
# 需要安装2.2及以上版本的Paddle
# 安装GPU版本的Paddle
pip install paddlepaddle-gpu==2.3.2
# 安装CPU版本的Paddle
pip install paddlepaddle==2.3.2
```

更多安装方法可以参考：[Paddle安装指南](https://www.paddlepaddle.org.cn/)。

* 安装requirements

```bash
pip install -r requirements.txt
```

### 3.2 准备数据

您可以在[百度网盘](https://pan.baidu.com/share/init?surl=qUVQLZh5aPT6d7-m6il6Rg)下载全量数据集，提取码1234。下载数据集后，将CROHME文件夹放置于paddlevision/datasets文件夹中。
**CROHME文件夹目录说明**
```
--CROHME
    --14_test_images.pkl                  # 测试图片
    --14_test_labels.txt		  # 测试标签
    --train_images.pkl			  # 训练图片
    --train_labels.txt			  # 训练标签
    --words_dict.txt			  # 词表
```

如果只是希望快速体验模型训练功能，则可以直接解压`test_images/lite_data.tar`，其中包含16张训练图像以及16张验证图像。

```
tar -xf test_images/lite_data.tar
```


### 3.3 准备模型

预训练模型：您可以在[百度网盘](https://pan.baidu.com/s/1bWG8UNK_GA9UxXkZ4RD7XA)下载预训练模型，提取码：n5ea。下载模型文件后，将[config.yaml](https://github.com/Lllllolita/CAN_Paddle/blob/master/config.yaml)中的checkpoint改为模型文件的前缀名。
进入config.yaml，假设模型文件名为CAN_123.pdparams
```
checkpoint: "CAN_123"
```
预测模型：您可以在[百度网盘](https://pan.baidu.com/s/1aUKUAWzRMbnDYxjHGRD_ZA)下载预测模型，提取码：em1l。

inference模型：您可以在[百度网盘](https://pan.baidu.com/s/1KU8NR0oCVn_pWD6wYm28AA)下载inference模型，提取码：ipz9。

训练、验证日志：您可以在[百度网盘](https://pan.baidu.com/s/1prO4DRLq2T99cDvdSGTumQ)下载tensorboard日志（.tfevents文件），提取码：p6he。下载日志后，将logs文件夹放置于CAN_Paddle根目录（替换repo中的logs文件夹）。

## 4. 开始使用


### 4.1 模型训练

1.使用python指令启动(只允许单卡训练)

训练文件在`tools`文件夹的[train.py](https://github.com/Lllllolita/CAN_Paddle/blob/master/tools/train.py)，由于代码中的路径均使用与CAN_Paddle文件夹的相对路径形式表示，因此需要先将CAN_Paddle文件夹指定为python的环境变量，设置为搜索路径的根路径。
进入`CAN_Paddle`文件夹，假设文件夹的绝对路径为`/home/a/CAN_Paddle`
```
export PYTHONPATH=$PYTHONPATH:/home/a/CAN_Paddle
```
然后启动训练命令
```
python tools/train.py --dataset CROHME
```
2.使用bash脚本启动(允许单卡、多卡训练)

进入`CAN_Paddle`文件夹，输入单卡训练指令
```
bash scripts/train_single_card.sh
```
进入`CAN_Paddle`文件夹，输入多卡训练指令
```
bash scripts/train_multi_cards.sh
```
3.若训练启动成功，则会输出日志
```
Loading data
共 111 类符号。
训练数据路径 images: paddlevision/datasets/CROHME/train_images.pkl labels: paddlevision/datasets/CROHME/train_labels.txt
验证数据路径 images: paddlevision/datasets/CROHME/14_test_images.pkl labels: paddlevision/datasets/CROHME/14_test_labels.txt
train dataset: 8835 train steps: 1105 eval dataset: 986 eval steps: 986 
Creating model
CAN_2022-09-21-11-48_decoder-AttDecoder
init tensorboard
Start training
[Epoch 1, iter: 1] wordRate: 0.10467, expRate: 0.00000, lr: 0.00001, loss: 905.89594, avg_reader_cost: 0.17917 sec, avg_batch_cost: 2.57803 sec, avg_samples: 8.0, avg_ips: 3.10314 images/sec.
[Epoch 1, iter: 2] wordRate: 0.05784, expRate: 0.00000, lr: 0.00002, loss: 1293.39844, avg_reader_cost: 0.00120 sec, avg_batch_cost: 0.69045 sec, avg_samples: 8.0, avg_ips: 11.58666 images/sec.
[Epoch 1, iter: 3] wordRate: 0.04320, expRate: 0.00000, lr: 0.00003, loss: 500.51590, avg_reader_cost: 0.00118 sec, avg_batch_cost: 0.85754 sec, avg_samples: 8.0, avg_ips: 9.32896 images/sec.
```
超参数设置于[`config.yaml`](https://github.com/Lllllolita/CAN_Paddle/blob/master/config.yaml)，包括初始学习率、批大小、学习率调参相关设置等。

训练保存的模型`.pdparams`文件位于`checkpoints`文件夹内，默认设置为只保存当前最优模型。

tensorboard保存的日志`.tfevents`文件位于`logs`文件夹内。


### 4.2 模型评估

1.使用python指令启动

评估文件在`tools`文件夹的[`train.py`](https://github.com/Lllllolita/CAN_Paddle/blob/master/tools/train.py)，由于代码中的路径均使用与CAN_Paddle文件夹的相对路径形式表示，因此需要先将CAN_Paddle文件夹指定为python的环境变量，设置为搜索路径的根路径。
进入`CAN_Paddle`文件夹，假设文件夹的绝对路径为`/home/a/CAN_Paddle`
```
export PYTHONPATH=$PYTHONPATH:/home/a/CAN_Paddle
```
然后启动评估命令
```
python tools/train.py --dataset CROHME --test-only
```
2.使用bash脚本启动

进入`CAN_Paddle`文件夹，输入评估指令
```
bash scripts/eval.sh
```
3.若评估启动成功，则会输出日志
```
Loading data
共 111 类符号。
训练数据路径 images: paddlevision/datasets/CROHME/train_images.pkl labels: paddlevision/datasets/CROHME/train_labels.txt
验证数据路径 images: paddlevision/datasets/CROHME/14_test_images.pkl labels: paddlevision/datasets/CROHME/14_test_labels.txt
train dataset: 8835 train steps: 1105 eval dataset: 986 eval steps: 986 
Creating model
CAN_2022-09-22-16-32_decoder-AttDecoder
init tensorboard
[Epoch 1, iter: 1] wordRate: 0.78571, expRate: 0.00000, word_loss: 0.73745, counting_loss: 0.06702
[Epoch 1, iter: 2] wordRate: 0.85185, expRate: 0.00000, word_loss: 0.31574, counting_loss: 0.01928
[Epoch 1, iter: 3] wordRate: 0.80723, expRate: 0.00000, word_loss: 1.06320, counting_loss: 0.32089
```
tensorboard保存的日志`.tfevents`文件位于`logs`文件夹内。

### 4.3 模型预测
1.使用python指令启动

预测文件在`tools`文件夹的[`predict.py`](https://github.com/Lllllolita/CAN_Paddle/blob/master/tools/predict.py)，由于代码中的路径均使用与CAN_Paddle文件夹的相对路径形式表示，因此需要先将CAN_Paddle文件夹指定为python的环境变量，设置为搜索路径的根路径。
进入`CAN_Paddle`文件夹，假设文件夹的绝对路径为`/home/a/CAN_Paddle`
```
export PYTHONPATH=$PYTHONPATH:/home/a/CAN_Paddle
```
预测预训练模型可以通过模型准备的百度网盘的链接进行下载，下载完的模型放在test_model文件夹里,并更名为predict.pdparams，预测样例图片在test_images的test_expamples里。

简单的预测命令如下：

（1）使用cpu进行预测：
```
python tools/predict.py 
```
（2）使用gpu进行预测：
```
python tools/predict.py --device 'gpu'
```
其中，使用gpu预测时，默认使用'gpu:0'进行预测。

可以使用以下命令更换预测使用的模型和预测图片：
```
python tools/predict.py --pretrained your_model_path --img_path your_img_path 
```
其中,--pretrained写入预测所需模型文件夹路径，--img_path需要提供预测图片路径。<br>
输出结果格式如下所示:
```
共 111 类符号。
seq_prob: \frac { 1 } { 3 } \pi r ^ { 2 } h
```
2.使用bash脚本启动(默认使用gpu)

进入`CAN_Paddle`文件夹，输入预测指令
```
bash scripts/predict.sh
```
### 4.4 查看训练、评估日志

下载本repo提供的[tensorboard日志](https://pan.baidu.com/s/1prO4DRLq2T99cDvdSGTumQ)，提取码：p6he，或自行训练并保存日志。下载日志后，将logs文件夹放置于CAN_Paddle根目录（替换repo中的logs文件夹）。日志提供了基于torch和paddle，使用Adadelta、SGD优化器的训练、验证日志曲线。
进入`logs`文件夹，假设文件夹的绝对路径为`/home/a/CAN_Paddle/logs`
```
tensorboard --logdir=/home/a/CAN_Paddle/logs --port=6006
```
若tensorboard启动成功，则会输出
```
2022-09-27 13:47:14.105902: I tensorflow/stream_executor/platform/default/dso_loader.cc:53] Successfully opened dynamic library libcudart.so.11.0

NOTE: Using experimental fast data loading logic. To disable, pass
    "--load_fast=false" and report issues on GitHub. More details:
    https://github.com/tensorflow/tensorboard/issues/4784

Serving TensorBoard on localhost; to expose to the network, use a proxy or pass --bind_all
TensorBoard 2.5.0 at http://localhost:6006/ (Press CTRL+C to quit)
```
启动成功后，在浏览器输入：`http://localhost:6006/` 即可进入tensorboard。


## 5. 模型推理部署
1.使用python指令启动
推理文件在`tools`文件夹的[infer.py](https://github.com/Lllllolita/CAN_Paddle/blob/master/tools/infer.py)，由于代码中的路径均使用与CAN_Paddle文件夹的相对路径形式表示，因此需要先将CAN_Paddle文件夹指定为python的环境变量，设置为搜索路径的根路径。
进入`CAN_Paddle`文件夹，假设文件夹的绝对路径为`/home/a/CAN_Paddle`
```
export PYTHONPATH=$PYTHONPATH:/home/a/CAN_Paddle
```
模型使用inference进行推理部署，inference模型可以通过模型准备的百度网盘的链接进行下载，下载完的模型（包括pdiparams，pdiparams.info，pdmodel文件）放在test_model文件夹里，要把inference和inference_faster文件全部下载下来，预测样例图片在test_images的test_expamples里。

简单的推理命令如下：

（1）使用cpu进行推理：
```
python tools/infer.py 
```
（2）使用gpu进行推理：
```
python tools/infer.py --use_gpu True
```
本项目准备了inference和inference_faster这两个模型，默认使用inference_faster这个模型，inference_faster主要识别序列较短的模型推理速度快于infernce，inference模型可以识别测试集中所有长度的序列，但是推理速度较慢。可以使用以下命令更换推理使用的模型和预测图片：

```
python tools/infer.py  --img_path your_img_path --if_fast (True or False)
```
其中,--img_path需要提供推理图片路径,--if_fast 设置为True则使用inference_faster模型，False则使用inference模型。

输出结果格式如下所示:
```
共 111 类符号。
image_name: ./images/RIT_2014_94.jpeg, result_seq: \frac { 1 } { 3 } \pi r ^ { 2 } h
```
2.使用bash脚本启动(默认使用gpu)

进入CAN_Paddle文件夹，输入预测指令
```
bash scripts/infer.sh
```

## 6. 自动化测试脚本

关于自动化测试脚本的使用信息可查看[TIPC使用文档](test_tipc/README.md)。


## 7. LICENSE

本项目的发布受[Apache 2.0 license](./LICENSE)许可认证。

## 8. 参考链接与文献
[When Counting Meets HMER: Counting-Aware Network for Handwritten Mathematical Expression Recognition](https://arxiv.org/abs/2207.11463)
