# BETTAFISH Conda 环境部署指南

本文档介绍如何使用 Conda 环境来部署和运行 BETTAFISH 项目。

## 目录

- [前置要求](#前置要求)
- [创建 Conda 环境](#创建-conda-环境)
- [激活环境](#激活环境)
- [安装依赖](#安装依赖)
- [配置环境变量](#配置环境变量)
- [验证安装](#验证安装)
- [运行项目](#运行项目)
- [常见问题](#常见问题)

---

## 前置要求

### 1. 安装 Anaconda 或 Miniconda

**推荐使用 Miniconda（轻量级）**：

```powershell
# 使用 winget 安装（推荐）
winget install -e --id Anaconda.Miniconda3

# 或者从官网下载安装
# https://docs.conda.io/en/latest/miniconda.html
```

**或者安装完整版 Anaconda**：

```powershell
winget install -e --id Anaconda.Anaconda3
```

### 2. 验证 Conda 安装

```powershell
conda --version
# 输出类似：conda 24.x.x
```

---

## 创建 Conda 环境

### 方法一：使用 environment.yml（推荐）

项目根目录创建 `environment.yml` 文件：

```yaml
name: bettafish
channels:
  - pytorch
  - conda-forge
  - defaults
dependencies:
  - python=3.11
  - pip
  - pip:
    - -r requirements.txt
```

然后执行：

```powershell
conda env create -f environment.yml
```

### 方法二：手动创建

```powershell
# 创建 Python 3.11 环境（推荐）
conda create -n bettafish python=3.11 -y

# 或者使用 Python 3.10
conda create -n bettafish python=3.10 -y
```

---

## 激活环境

```powershell
# 激活环境
conda activate bettafish

# 退出环境
conda deactivate
```

---

## 安装依赖

### 1. 使用 pip 安装 requirements.txt

```powershell
# 激活环境后进入项目目录
cd d:\96.qoder\BettaFish

# 安装依赖
pip install -r requirements.txt
```

### 2. 单独安装关键依赖（可选加速）

```powershell
# 安装 PyTorch（CPU 版本）
pip install torch>=2.0.0 --index-url https://download.pytorch.org/whl/cpu

# 安装 PyTorch（GPU 版本，CUDA 12.6）
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu126

# 安装 Playwright（可选，爬虫需要）
playwright install chromium
```

---

## 配置环境变量

### 1. 复制环境变量模板

```powershell
# 进入项目目录
cd d:\96.qoder\BettaFish

# 复制环境变量文件
copy .env.example .env
```

### 2. 编辑 .env 文件

根据您的需求编辑 `.env` 文件，主要配置项包括：

#### 数据库配置
```env
DB_HOST=localhost
DB_PORT=3306
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DB_NAME=your_db_name
DB_DIALECT=postgresql
```

#### LLM API 配置（必填）

项目支持多个 LLM 引擎，请至少配置一个：

**Insight Engine（推荐 Kimi）**
```env
INSIGHT_ENGINE_API_KEY=your_kimi_api_key
INSIGHT_ENGINE_BASE_URL=https://api.moonshot.cn/v1
INSIGHT_ENGINE_MODEL_NAME=kimi-k2-0711-preview
```

**Media Engine（推荐 Gemini）**
```env
MEDIA_ENGINE_API_KEY=your_gemini_api_key
MEDIA_ENGINE_BASE_URL=https://aihubmix.com/v1
MEDIA_ENGINE_MODEL_NAME=gemini-2.5-pro
```

**Query Engine（推荐 DeepSeek）**
```env
QUERY_ENGINE_API_KEY=your_deepseek_api_key
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-chat
```

**Report Engine（推荐 Gemini）**
```env
REPORT_ENGINE_API_KEY=your_gemini_api_key
REPORT_ENGINE_BASE_URL=https://aihubmix.com/v1
REPORT_ENGINE_MODEL_NAME=gemini-2.5-pro
```

#### 网络搜索配置（可选）

**Anspire API（推荐）**
```env
SEARCH_TOOL_TYPE=AnspireAPI
ANSPIRE_API_KEY=your_anspire_api_key
```

**Bocha API**
```env
SEARCH_TOOL_TYPE=BochaAPI
BOCHA_WEB_SEARCH_API_KEY=your_bocha_api_key
```

---

## 验证安装

```powershell
# 激活环境
conda activate bettafish

# 检查 Python 版本
python --version

# 检查关键依赖
python -c "import flask; import streamlit; import torch; print('依赖检查通过')"

# 检查项目模块
python -c "from InsightEngine import agent; from MediaEngine import agent; from QueryEngine import agent; print('引擎模块检查通过')"
```

---

## 运行项目

### 1. 运行主应用

```powershell
cd d:\96.qoder\BettaFish
python app.py
```

访问地址：`http://localhost:5000`

### 2. 运行独立引擎

```powershell
# Insight Engine
cd d:\96.qoder\BettaFish
streamlit run SingleEngineApp/insight_engine_streamlit_app.py

# Media Engine
cd d:\96.qoder\BettaFish
streamlit run SingleEngineApp/media_engine_streamlit_app.py

# Query Engine
cd d:\96.qoder\BettaFish
streamlit run SingleEngineApp/query_engine_streamlit_app.py
```

### 3. 生成报告

```powershell
# 生成 PDF 报告
python regenerate_latest_pdf.py

# 生成 Markdown 报告
python regenerate_latest_md.py

# 生成 HTML 报告
python regenerate_latest_html.py
```

---

## 常见问题

### Q1: Conda 激活环境失败

**解决方案**：使用 PowerShell 管理员模式运行：
```powershell
# 初始化 conda
conda init powershell

# 重新打开 PowerShell
```

### Q2: PyTorch 安装失败

**解决方案**：使用清华镜像源：
```powershell
pip install torch torchvision -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### Q3: Playwright 浏览器无法安装

**解决方案**：
```powershell
playwright install --with-deps chromium
```

### Q4: WeasyPrint PDF 导出失败

**解决方案**（Windows 环境）：
```powershell
# 安装 GTK3 运行时
# 下载地址：https://github.com/tschoonj/GTK-for-Windows-Runtime-Environment-Archive

# 或者使用 conda 安装
conda install -c conda-forge weasyprint
```

### Q5: 数据库连接失败

**解决方案**：
1. 确保数据库服务已启动
2. 检查 `.env` 文件中的数据库配置是否正确
3. 确认数据库用户权限

### Q6: LLM API 调用失败

**解决方案**：
1. 检查 API Key 是否正确配置
2. 确认 BASE_URL 是否正确
3. 查看网络是否可达
4. 检查 API 余额是否充足

---

## 环境管理命令汇总

```powershell
# 查看环境列表
conda env list

# 创建环境
conda create -n bettafish python=3.11 -y

# 激活环境
conda activate bettafish

# 退出环境
conda deactivate

# 删除环境
conda env remove -n bettafish

# 导出环境配置
conda env export > environment.yml

# 更新依赖
pip install -r requirements.txt --upgrade
```

---

## 技术支持

如遇到问题，请查看：
- 项目 GitHub Issues
- 项目 Wiki 文档
- 调试日志（logs 目录）
