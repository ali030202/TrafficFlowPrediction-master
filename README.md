# Traffic Flow Prediction

基于历史交通流量时间序列，使用 LSTM、GRU 和 SAE 风格的堆叠全连接网络预测后续流量。项目包含数据预处理、模型训练、已有模型权重和评估可视化。

## 项目简介

数据文件以 <code>time</code> 和 <code>volumn</code> 两列记录时间与流量。预处理脚本只把 <code>volumn</code> 作为模型输入：用前 12 个采样点预测下一个采样点。示例数据的采样间隔为 5 分钟，因此每次预测使用约 1 小时的历史流量。

模型使用 Min-Max 归一化；缩放器只在训练集上拟合，再用于测试集。缺失值在读取时填为 0。训练脚本包含训练/验证划分，测试脚本输出 MAPE、MAE、MSE、RMSE、R² 和 explained variance。

## 模型

- **LSTM**：两层 LSTM（隐藏维度 64），Dropout 0.2，Sigmoid 输出。
- **GRU**：两层 GRU（隐藏维度 64），Dropout 0.2，Sigmoid 输出。
- **SAEs**：项目中的堆叠全连接预测模型；逐层训练浅层网络，再将隐藏层权重迁移到最终模型。

三种模型都使用 MSE 损失和 RMSprop 优化器。默认训练参数为 batch size 256、最多 600 个 epoch、5% 训练数据用于验证。

## 目录结构

代码和数据位于仓库内的重复嵌套目录中：

~~~text
.
├── README.md
└── TrafficFlowPrediction-master/
    └── TrafficFlowPrediction-master/
        ├── data/
        │   ├── data.py
        │   └── 100211data/
        │       ├── 100211_all_train.csv
        │       ├── 100211_all_test.csv
        │       ├── 100211_workdays_train.csv
        │       ├── 100211_workdays_test.csv
        │       ├── 100211_weekend_train.csv
        │       └── 100211_weekend_test.csv
        ├── images/
        ├── model/
        │   ├── model.py
        │   ├── 100211_all/
        │   ├── 100211_workdays/
        │   └── 100211_weekend/
        ├── train.py
        └── test.py
~~~

## 环境准备

建议使用兼容旧版 Keras API 的 TensorFlow/Keras 2.x 环境。仓库没有提供锁定版本的 requirements.txt；代码依赖 NumPy、Pandas、scikit-learn、Matplotlib 和 Keras/TensorFlow。

~~~bash
python -m pip install numpy pandas scikit-learn matplotlib tensorflow
~~~

进入代码目录：

~~~bash
cd TrafficFlowPrediction-master/TrafficFlowPrediction-master
~~~

> 模型定义使用 keras.layers.recurrent 等旧版 Keras 导入路径；Keras 3 环境可能需要调整导入语句。仓库中已有训练好的 .h5 模型，但不同 TensorFlow/Keras 版本之间的模型兼容性可能不同。

## 训练

在上述代码目录下运行：

~~~bash
python train.py --model lstm
python train.py --model gru
python train.py --model saes
~~~

--model 可选 lstm、gru 或 saes。不传参数时默认训练 LSTM。训练数据默认使用 data/100211data/100211_all_train.csv；data/100211data/100211_all_test.csv 会由预处理函数读取，但当前训练入口没有用它计算测试集指标。

新训练的模型和 loss 日志写入 model/，例如 model/lstm.h5 和 model/lstm loss.csv。这些输出可能覆盖同名文件。

## 测试与结果

项目已有按全量、工作日和周末划分保存的模型权重，位置在 model/100211_all/、model/100211_workdays/ 和 model/100211_weekend/。图表保存在 images/。

测试脚本使用 model/100211_all/ 下的权重，并在周末数据上评估。当前 test.py 有两处需要留意：

1. 它写的是 from data import process_data，而函数实际位于 data/data.py。如果出现导入错误，将该语句改为 from data.data import process_data。
2. 脚本把待测模型列表重新设为仅包含 LSTM，因此当前运行只评估 LSTM。若要比较三种模型，需移除该覆盖语句，并确认列表与名称顺序一致。

评估结果与预测图会由 test.py 打印或写入 images/。仓库中的图片可直接查看：

- [周末流量预测图](TrafficFlowPrediction-master/TrafficFlowPrediction-master/images/pre_weekend.png)
- [工作日流量预测图](TrafficFlowPrediction-master/TrafficFlowPrediction-master/images/pre_workdays.png)
- [LSTM 结构图](TrafficFlowPrediction-master/TrafficFlowPrediction-master/images/LSTM.png)
- [GRU 结构图](TrafficFlowPrediction-master/TrafficFlowPrediction-master/images/GRU.png)
- [SAEs 结构图](TrafficFlowPrediction-master/TrafficFlowPrediction-master/images/SAEs.png)

## 数据与许可

CSV 中的 volumn 是代码读取的列名，改列名时也要同步修改 data/data.py。仓库包含多个训练/测试划分及模型产物；数据来源和采集设备信息未在代码中完整说明。

项目附带的 [MIT License](TrafficFlowPrediction-master/TrafficFlowPrediction-master/LICENSE) 保留了原仓库署名信息。
