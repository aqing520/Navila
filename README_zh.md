<div align="center">

<p align="center">
  <img src="assets/logo.png" width="20%"/>
</p>

# NaVILA：面向导航的腿足机器人视觉-语言-动作模型（RSS'25）

[![网站](https://img.shields.io/badge/网站-6DE1D2?style=for-the-badge&logo=safari&labelColor=555555)](https://navila-bot.github.io/)
[![论文](https://img.shields.io/badge/论文-F75A5A?style=for-the-badge&logo=arxiv&labelColor=555555)](https://arxiv.org/abs/2412.04453)
[![Huggingface](https://img.shields.io/badge/Huggingface-FFD63A?style=for-the-badge&logo=huggingface&labelColor=555555)](https://huggingface.co/collections/a8cheng/navila-legged-robot-vision-language-action-model-for-naviga-67cfc82b83017babdcefd4ad)
[![运动控制代码](https://img.shields.io/badge/运动控制代码-FFA955?style=for-the-badge&logo=github&labelColor=555555)](https://github.com/yang-zj1026/legged-loco)

<p align="center">
  <img src="assets/teaser.gif" width="600">
</p>

</div>

## 💡 项目简介

**NaVILA** 是一个面向腿足机器人导航的两层框架，将视觉-语言-动作模型（VLA）与底层运动技能相结合：

- **高层模块**：利用大型视觉语言模型（基于 LLaMA-3 8B + SigLIP）理解视觉输入和自然语言指令，生成高层语言命令（如"向左转"、"直行"、"停止"）。
- **底层模块**：实时运动策略根据高层命令控制机器人腿部运动，同时实现障碍物规避。

该框架使腿足机器人能够理解自然语言导航指令（如"去找厨房"），并在复杂室内环境中自主完成导航任务。

<p align="center">
  <img src="assets/method.png" width="600">
</p>

## 🗂️ 代码结构

```
Navila/
├── llava/                  # 核心模型代码（基于 VILA/LLaVA 框架）
│   ├── model/              # 模型定义（VLA 架构、视觉编码器、语言模型）
│   ├── data/               # 数据集加载与混合配置
│   ├── train/              # 训练相关代码
│   └── eval/               # 评测相关代码
├── evaluation/             # 导航评测框架（基于 VLN-CE/Habitat）
│   ├── vlnce_baselines/    # VLN-CE 基线与 NaVILA 推理逻辑
│   ├── habitat_extensions/ # Habitat 环境扩展
│   └── scripts/            # 评测运行脚本
├── scripts/                # 训练与数据处理脚本
│   ├── train/              # 训练启动脚本（如 sft_8frames.sh）
│   └── extract_rawframes.py# 视频帧提取工具
└── assets/                 # 图片、GIF 等媒体资源
```

## TODO
- [x] 发布模型权重与评测代码
- [x] 发布训练代码
- [x] 发布 YouTube 人类导览数据集
- [x] 发布 Isaac Sim 评测，详见 [NaVILA-Bench](https://github.com/yang-zj1026/NaVILA-Bench)

## 🚀 训练

### 环境安装

```bash
./environment_setup.sh navila
conda activate navila
```

可选：如需使用 TensorBoard 记录训练日志，请额外安装：

```bash
pip install tensorboardX
```

### 数据集准备

通用视觉问答数据集（`video_chatgpt`、`sharegpt_video`、`sharegpt4v_sft`）请参考 [NVILA](https://github.com/NVlabs/VILA) 的数据准备说明。

导航专用数据集（`envdrop`、`scanqa`、`r2r`、`rxr`、`human`）的标注文件已发布至 [Hugging Face](https://huggingface.co/datasets/a8cheng/NaVILA-Dataset)。

数据集目录结构如下：

```
NaVILA-Dataset
├─ EnvDrop
│   ├─ videos/
│   └─ annotations.json
├─ Human
│   ├─ raw_frames/
│   ├─ videos/
│   ├─ annotations.json
│   └─ video_ids.txt
├─ R2R
│   ├─ train/
│   └─ annotations.json
├─ RxR
│   ├─ train/
│   └─ annotations.json
└─ ScanQA
    ├─ videos/
    └─ annotations/
```

### 启动训练

预训练模型权重：[a8cheng/navila-siglip-llama3-8b-v1.5-pretrain](https://huggingface.co/a8cheng/navila-siglip-llama3-8b-v1.5-pretrain)

请修改 `llava/data/datasets_mixture.py` 中的数据路径，然后运行：

```bash
bash scripts/train/sft_8frames.sh
```

## 📊 评测

### 环境安装

评测框架基于 [VLN-CE](https://github.com/jacobkrantz/VLN-CE)，依赖旧版 [Habitat-Lab](https://github.com/facebookresearch/habitat-lab/tree/v0.1.7) 和 [Habitat-Sim](https://github.com/facebookresearch/habitat-lab/tree/v0.1.7)。

1. 创建 Python 3.10 的 Conda 环境：

```bash
conda create -n navila-eval python=3.10
conda activate navila-eval
```

2. 从源码编译 Habitat-Sim & Lab（v0.1.7），参考 [VLN-CE 安装指南](https://github.com/jacobkrantz/VLN-CE?tab=readme-ov-file#setup)。

   修复 NumPy 兼容性问题：

```bash
python evaluation/scripts/habitat_sim_autofix.py
```

3. 安装 VLN-CE 依赖：

```bash
pip install -r evaluation/requirements.txt
```

4. 安装 VILA 依赖：

```bash
pip install https://github.com/Dao-AILab/flash-attention/releases/download/v2.5.8/flash_attn-2.5.8+cu122torch2.3cxx11abiFALSE-cp310-cp310-linux_x86_64.whl
pip install -e .
pip install -e ".[train]"
pip install -e ".[eval]"
pip install git+https://github.com/huggingface/transformers@v4.37.2
site_pkg_path=$(python -c 'import site; print(site.getsitepackages()[0])')
cp -rv ./llava/train/transformers_replace/* $site_pkg_path/transformers/
cp -rv ./llava/train/deepspeed_replace/* $site_pkg_path/deepspeed/
pip install webdataset==0.1.103
```

### 数据准备

按照 [VLN-CE](https://github.com/jacobkrantz/VLN-CE) 说明，将 R2R 和 RxR 标注及场景数据放置于 `evaluation/data/` 目录下。

### 运行评测

1. 下载模型权重：[a8cheng/navila-llama3-8b-8f](https://huggingface.co/a8cheng/navila-llama3-8b-8f)

2. 在 R2R 上运行评测：

```bash
cd evaluation
# 单 GPU
bash scripts/eval/r2r.sh CKPT_PATH 1 0 "0"
# 多 GPU（以 8 卡为例）
bash scripts/eval/r2r.sh CKPT_PATH 8 0 "0,1,2,3,4,5,6,7"
```

3. 可视化视频保存在：

```bash
./eval_out/CKPT_NAME/VLN-CE-v1/val_unseen/videos
```

<p align="center">
  <img src="assets/sample.gif" width="600">
</p>

4. 汇总评测结果：

```bash
python scripts/eval_jsons.py ./eval_out/CKPT_NAME/VLN-CE-v1/val_unseen NUM_CHUNKS
```

## 📜 引用

```bibtex
@inproceedings{cheng2025navila,
        title={Navila: Legged robot vision-language-action model for navigation},
        author={Cheng, An-Chieh and Ji, Yandong and Yang, Zhaojing and Gongye, Zaitian and Zou, Xueyan and Kautz, Jan and Bıyık, Erdem and Yin, Hongxu and Liu, Sifei and Wang, Xiaolong},
        booktitle={RSS},
        year={2025}
}
```
