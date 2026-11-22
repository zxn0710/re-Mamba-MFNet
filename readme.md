
## 一、环境概述

（源代码使用的是双卡，此处使用的是单卡）

- 操作系统：Ubuntu 20.04
- GPU：单卡（vGPU-32GB）
- Python：3.8
- PyTorch：2.0.0 + CUDA 11.8

## 二、创建 Conda 环境

```bash
conda create -n mamba python=3.8
conda activate mamba
```

## 三、安装 PyTorch 2.0.0 + CUDA 11.8

```bash
pip install torch==2.0.0 torchvision==0.15.0 torchaudio==2.0.0 --index-url https://download.pytorch.org/whl/cu118
```

## 四、安装其余依赖

```bash
pip install -r requirements.txt
pip install --no-binary charset_normalizer charset_normalizer==3.3.2
pip install requests==2.32.3
```

## 五、安装 mamba-ssm 和 causal-conv1d

### 方式一：从 PyPI 安装

```bash
pip uninstall mamba-ssm causal-conv1d -y
pip install mamba-ssm==1.1.1 causal-conv1d==1.1.1 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 方式二：手动下载 wheel 上传安装

如果 pip 卡在 GitHub 下载（Remote end closed connection without response），在能访问 GitHub 的机器上下载对应的 wheel，下载链接如下：

- [mamba_ssm-1.1.1+cu118torch2.0cxx11abiFALSE-cp38-cp38-linux_x86_64.whl](https://github.com/state-spaces/mamba/releases/download/v1.1.1/mamba_ssm-1.1.1+cu118torch2.0cxx11abiFALSE-cp38-cp38-linux_x86_64.whl)
- [causal_conv1d-1.1.1+cu118torch2.0cxx11abiFALSE-cp38-cp38-linux_x86_64.whl](https://github.com/Dao-AILab/causal-conv1d/releases/download/v1.1.1/causal_conv1d-1.1.1+cu118torch2.0cxx11abiFALSE-cp38-cp38-linux_x86_64.whl)

上传到服务器 /root/，然后本地安装：

```bash
pip install /root/causal_conv1d-1.1.1+cu118torch2.0cxx11abiFALSE-cp38-cp38-linux_x86_64.whl
pip install /root/mamba_ssm-1.1.1+cu118torch2.0cxx11abiFALSE-cp38-cp38-linux_x86_64.whl
```

wheel 文件名中的关键字段必须与你的环境匹配：

- `cu118`：CUDA 11.8
- `torch2.0`：PyTorch 2.0
- `cp38`：Python 3.8
- `cxx11abiFALSE`：与 PyTorch 编译 ABI 一致

## 六、数据准备

下载哈佛医学院数据集（[https://www.med.harvard.edu/AANLIB/home.html](https://www.med.harvard.edu/AANLIB/home.html)）

或源代码整理过的版本，即随机分割出训练集：测试集 = 8：2，包含184 CT-MRI、269 PET-MRI、357 SPECT-MRI（[https://github.com/yuting77777/AANLIB-database-of-Harvard-Medical-School.git](https://github.com/yuting77777/AANLIB-database-of-Harvard-Medical-School.git)）

放到 `Mamba-MFNet-main/datasets` 目录下

## 七、运行训练

### 单卡运行

```bash
python train_MambaMFNet.py --opt options/MambaMFNet/train_mambamfnet_mif.json
```
可以通过 json 文件修改训练的迭代次数 iter

### 双卡运行

编辑 `options/MambaMFNet/train_mambamfnet_mif.json`，修改以下字段：

- `gpu_ids`：从 `[0]` 改为 `[0, 1]`
- `dist`：从 `false` 改为 `true`

```bash
python -m torch.distributed.launch --nproc_per_node=2 --master_port=1234 train_MambaMFNet.py --opt options/MambaMFNet/train_mambamfnet_mif.json
```

## 八、测试

```bash
python test_MambaMFNet.py
--model_path=./Model/Medical_Fusion-CT-MRI/Medical_Fusion/models/
--iter_number=100000
--dataset=CT-MRI
--A_dir=CT
--B_dir=MRI
```
