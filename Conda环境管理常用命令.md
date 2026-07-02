# Conda 环境管理常用命令

> 命令中的 `<...>` 表示占位内容。实际执行时，请替换为本机真实路径、环境名称或包名，并删除尖括号。路径包含空格时，建议使用英文双引号包裹完整路径。

## 1. 配置 Conda 环境默认存储目录

该配置用于指定新建 Conda 环境的默认保存位置。通常只需配置一次。

```bat
<Conda安装目录>\Scripts\activate.bat
conda config --prepend envs_dirs <Conda环境目录>
```

验证环境目录配置：

```bat
conda config --show envs_dirs
```

预期输出中，`<Conda环境目录>` 应位于 `envs_dirs` 列表首位。

如需同时指定 Conda 包缓存目录，可配置 `pkgs_dirs`：

```bat
conda config --prepend pkgs_dirs <Conda包缓存目录>
```

查看包缓存目录配置：

```bat
conda config --show pkgs_dirs
```

固定路径示例：

```bat
D:\miniconda3\Scripts\activate.bat
conda config --prepend envs_dirs D:\conda_envs
conda config --prepend pkgs_dirs D:\conda_pkgs
```

> 建议 Conda 安装目录、环境目录、包缓存目录和项目目录均使用纯英文路径。部分 Python 包、编译工具、PyInstaller、Qt 相关组件对中文路径和路径编码兼容性较差，可能导致安装、构建或运行异常。

## 2. 创建 Conda 环境的标准流程

### 2.1 打开 CMD

打开 Windows `cmd` 命令提示符。

### 2.2 激活 Conda 基础环境

执行 Conda 激活脚本：

```bat
<Conda安装目录>\Scripts\activate.bat
```

固定路径示例：

```bat
D:\miniconda3\Scripts\activate.bat
```

激活成功后，命令提示符前通常会显示：

```text
(base)
```

### 2.3 创建环境

命令格式：

```bat
conda create -n <环境名称> python=<Python版本>
```

示例：

```bat
conda create -n GPEnv python=3.10
```

如果已配置 `envs_dirs`，该环境默认创建到 `<Conda环境目录>` 下。

### 2.4 激活环境

```bat
conda activate <环境名称>
```

示例：

```bat
conda activate GPEnv
```

### 2.5 切换到项目根目录

```bat
cd /d <项目根目录>
```

示例：

```bat
cd /d D:\projects\demo
```

`/d` 参数用于在切换目录时同时切换磁盘分区。

### 2.6 安装 requirements 依赖

如果项目根目录包含 `requirements.txt`：

```bat
pip install -r requirements.txt
```

如需单独使用清华 PyPI 镜像源：

```bat
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

永久更换清华源命令：

```bat
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple/
```

## 3. 创建环境相关命令

创建指定 Python 版本的环境：

```bat
conda create -n myenv python=3.10
```

创建环境并安装指定包：

```bat
conda create -n myenv python=3.10 numpy pandas
```

在指定路径创建环境：

```bat
conda create --prefix <Conda环境目录>\myenv python=3.10
```

基于 `environment.yml` 创建环境：

```bat
conda env create -f environment.yml
```

基于 `environment.yml` 在指定路径创建环境：

```bat
conda env create --prefix <Conda环境目录>\myenv -f environment.yml
```

## 4. 激活和退出环境

按环境名称激活环境：

```bat
conda activate myenv
```

按环境路径激活环境：

```bat
conda activate <Conda环境目录>\myenv
```

退出当前 Conda 环境：

```bat
conda deactivate
```

如果存在多层环境激活状态，可重复执行 `conda deactivate`，直到退出全部 Conda 环境：

```bat
conda deactivate
conda deactivate
```

## 5. 查看环境信息

查看所有 Conda 环境：

```bat
conda env list
```

或：

```bat
conda info --envs
```

查看当前 Conda 配置和环境信息：

```bat
conda info
```

查看当前 `python` 可执行文件路径：

```bat
where python
```

查看当前 `pip` 可执行文件路径：

```bat
where pip
```

查看 Python 版本：

```bat
python --version
```

查看 Conda 版本：

```bat
conda --version
```

## 6. 删除环境

按环境名称删除：

```bat
conda remove -n myenv --all
```

也可以使用 `conda env remove` 删除环境：

```bat
conda env remove -n myenv
```

按环境路径删除：

```bat
conda remove --prefix <Conda环境目录>\myenv --all
```

按环境路径使用 `conda env remove` 删除：

```bat
conda env remove -p <Conda环境目录>\myenv
```

删除前应先确认目标环境，避免误删：

```bat
conda env list
```

## 7. 克隆和迁移环境

将旧环境克隆到新的环境目录：

```bat
conda create --prefix <Conda环境目录>\OldEnv --clone <旧环境路径或旧环境名称>
```

如果 Conda 能识别原环境名称，也可使用环境名作为克隆源：

```bat
conda create --prefix <Conda环境目录>\OldEnv --clone OldEnv
```

固定路径示例：

```bat
conda create --prefix D:\conda_envs\OldEnv --clone C:\Users\用户名\.conda\envs\OldEnv
```

验证新环境：

```bat
conda activate <Conda环境目录>\OldEnv
python --version
```

确认新环境可正常使用后，再删除旧环境：

```bat
conda remove --prefix <旧环境路径> --all
```

重新查看环境列表：

```bat
conda env list
```

## 8. 安装包

优先使用 Conda 安装可由 Conda 管理的包；仅当 Conda channel 中没有目标包，或项目明确要求 pip 安装时，再使用 pip。混用 Conda 和 pip 时，建议先安装 Conda 包，再安装 pip 包。

使用 Conda 安装包：

```bat
conda install numpy
```

安装多个包：

```bat
conda install numpy pandas matplotlib
```

安装指定版本：

```bat
conda install numpy=1.26
```

从指定 channel 安装：

```bat
conda install -c conda-forge <包名>
```

使用 `conda-forge` 安装多个包：

```bat
conda install -c conda-forge numpy pandas
```

使用 pip 安装包：

```bat
pip install <包名>
```

基于 `requirements.txt` 安装 pip 依赖：

```bat
pip install -r requirements.txt
```

如果需要确保 pip 对应当前环境的 Python，可使用：

```bat
python -m pip install -r requirements.txt
```

以可编辑模式安装本地项目：

```bat
pip install -e .
```

## 9. 卸载包

使用 Conda 卸载包：

```bat
conda remove <包名>
```

使用 pip 卸载包：

```bat
pip uninstall <包名>
```

## 10. 更新包和 Conda

更新指定包：

```bat
conda update <包名>
```

更新当前环境中的所有 Conda 包：

```bat
conda update --all
```

更新 Conda：

```bat
conda update conda
```

更新 pip：

```bat
python -m pip install --upgrade pip
```

## 11. 查看包信息

查看当前环境中的所有 Conda 包：

```bat
conda list
```

查看指定 Conda 包：

```bat
conda list numpy
```

查看当前环境中的 pip 包：

```bat
pip list
```

查看指定 pip 包详情：

```bat
pip show <包名>
```

查看包的导入路径：

```bat
python -c "import package_name; print(package_name.__file__)"
```

示例：

```bat
python -c "import numpy; print(numpy.__file__)"
```

## 12. 导出和恢复环境

导出完整 Conda 环境：

```bat
conda env export > environment.yml
```

导出环境时去除 build 信息，便于跨设备复用：

```bat
conda env export --no-builds > environment.yml
```

仅导出显式安装过的 Conda 包：

```bat
conda env export --from-history > environment.yml
```

使用 pip 导出依赖清单：

```bat
pip freeze > requirements.txt
```

基于 `environment.yml` 更新当前环境，并移除文件中未声明的依赖：

```bat
conda env update -f environment.yml --prune
```

基于 `requirements.txt` 安装 pip 依赖：

```bat
pip install -r requirements.txt
```

## 13. Channel 管理

查看 channel 配置：

```bat
conda config --show channels
```

添加 `conda-forge`：

```bat
conda config --add channels conda-forge
```

将 `conda-forge` 调整到 channel 列表首位：

```bat
conda config --prepend channels conda-forge
```

删除指定 channel：

```bat
conda config --remove channels conda-forge
```

启用严格 channel 优先级：

```bat
conda config --set channel_priority strict
```

恢复灵活 channel 优先级：

```bat
conda config --set channel_priority flexible
```

## 14. 配置查看和修改

查看全部 Conda 配置：

```bat
conda config --show
```

查看配置来源文件：

```bat
conda config --show-sources
```

查看环境目录配置：

```bat
conda config --show envs_dirs
```

添加环境目录：

```bat
conda config --add envs_dirs <Conda环境目录>
```

将环境目录调整到列表首位：

```bat
conda config --prepend envs_dirs <Conda环境目录>
```

删除环境目录配置：

```bat
conda config --remove envs_dirs <Conda环境目录>
```

查看包缓存目录配置：

```bat
conda config --show pkgs_dirs
```

添加包缓存目录：

```bat
conda config --prepend pkgs_dirs <Conda包缓存目录>
```

关闭启动终端时自动激活 `base` 环境：

```bat
conda config --set auto_activate_base false
```

恢复自动激活 `base` 环境：

```bat
conda config --set auto_activate_base true
```

## 15. 清理缓存

清理未使用的包缓存：

```bat
conda clean --packages
```

清理压缩包缓存：

```bat
conda clean --tarballs
```

清理索引缓存：

```bat
conda clean --index-cache
```

清理所有可清理缓存：

```bat
conda clean --all
```

自动确认清理操作：

```bat
conda clean --all -y
```

## 16. 在指定环境中运行命令

不激活环境，直接在指定环境中运行 Python：

```bat
conda run -n myenv python --version
```

按环境路径运行命令：

```bat
conda run --prefix <Conda环境目录>\myenv python --version
```

在指定环境中运行脚本：

```bat
conda run -n myenv python main.py
```

## 17. Jupyter 相关命令

在当前环境安装 Jupyter kernel：

```bat
pip install ipykernel
python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
```

查看 Jupyter kernel 列表：

```bat
jupyter kernelspec list
```

删除指定 Jupyter kernel：

```bat
jupyter kernelspec uninstall myenv
```

## 18. 常见问题处理

### 18.1 `conda` 不是内部或外部命令

该错误表示当前 CMD 无法找到 Conda 命令。可先执行 Conda 激活脚本：

```bat
<Conda安装目录>\Scripts\activate.bat
```

再执行 Conda 命令：

```bat
conda env list
```

也可以直接使用 Conda 的完整路径：

```bat
<Conda安装目录>\condabin\conda.bat env list
```

固定路径示例：

```bat
D:\miniconda3\condabin\conda.bat env list
```

### 18.2 环境被创建到非预期目录

先查看当前环境目录配置：

```bat
conda config --show envs_dirs
```

再将目标环境目录调整到列表首位：

```bat
conda config --prepend envs_dirs <Conda环境目录>
```

### 18.3 环境目录没有写入权限

如果创建环境时报 `Access is denied`，表示当前用户对目标目录缺少写入权限。可使用管理员 CMD 执行：

```bat
icacls <Conda环境目录> /grant <用户名>:(OI)(CI)M
```

示例：

```bat
icacls D:\conda_envs /grant demo_user:(OI)(CI)M
```

### 18.4 中文路径导致异常

Conda 环境目录不建议放在中文路径下。推荐使用英文路径，例如：

```text
D:\conda_envs
D:\projects\demo\.venv
```

## 19. 推荐日常流程

新项目环境创建流程：

```bat
<Conda安装目录>\Scripts\activate.bat
conda create -n ProjectEnv python=3.10
conda activate ProjectEnv
cd /d <项目根目录>
pip install -r requirements.txt
python --version
pip list
```

固定路径示例：

```bat
D:\miniconda3\Scripts\activate.bat
conda create -n ProjectEnv python=3.10
conda activate ProjectEnv
cd /d D:\projects\demo
pip install -r requirements.txt
python --version
pip list
```

旧环境迁移流程：

```bat
<Conda安装目录>\Scripts\activate.bat
conda create --prefix <Conda环境目录>\OldEnv --clone <旧环境路径>
conda activate <Conda环境目录>\OldEnv
python --version
conda remove --prefix <旧环境路径> --all
conda env list
```
