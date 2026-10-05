# 运行命令

- 项目：MeloTTS（myshell-ai/MeloTTS）
$12026-09-09
- 运行方式：直接运行（官方对 Windows 建议 Docker，但本机 docker.io 超时不可达，故用 uv + Python 3.10）
- 硬件评估：满足
  - 官方：CPU 即可实时推理；未要求 16GB+ 显存
  - 本机：RTX 4080 / 16GB；当前约占用 6.3GB、空闲约 9.6GB；系统内存约 32GB
  - 结论：WebUI 会预加载多语种模型，显存/内存均够用

## 环境准备

```powershell
# 建议在仓库根目录执行（pwsh）
$env:UV_INDEX_URL = 'https://pypi.tuna.tsinghua.edu.cn/simple'
$env:HF_ENDPOINT = 'https://hf-mirror.com'
$env:Path = 'E:\Programs\ffmpeg-master-latest-win64-gpl\bin;' + $env:Path

# 使用 Python 3.10（项目按 3.9 测试；勿用本机默认 3.14）
uv venv --python 3.10 .venv
.\.venv\Scripts\Activate.ps1
uv pip install -e .
uv run python -m unidic download
```

可选预下载多语种权重（WebUI 启动时也会自动拉）：

```powershell
$env:HF_ENDPOINT = 'https://hf-mirror.com'
uv run python melo/init_downloads.py
```

## 启动

- 推荐：`pwsh -NoProfile -File .\start.ps1`（单服务 WebUI，跳过菜单）
- 服务 Id：`webui`
- 等价手动命令：

```powershell
$env:HF_ENDPOINT = 'https://hf-mirror.com'
$env:Path = 'E:\Programs\ffmpeg-master-latest-win64-gpl\bin;' + $env:Path
.\.venv\Scripts\python.exe melo\app.py --host 127.0.0.1 --port 8888
```

Docker 备选（需本机 Docker Hub / 镜像加速可用后）：`pwsh -NoProfile -File .\start.ps1 -Mode docker`

## 验证

1. 浏览器打开日志中的本地地址（默认从 8888 顺延）
2. 选语言 EN / ZH，点 Synthesize，应生成可播放 wav
3. CLI 冒烟：

```powershell
$env:HF_ENDPOINT = 'https://hf-mirror.com'
uv run melo "Hello from MeloTTS." test-en.wav --language EN --speaker EN-US
```

预期：退出码 0，生成 `test-en.wav`。

## 备注

- 镜像源：PyPI 用清华 tuna；Hugging Face 用 `hf-mirror.com`（huggingface.co 探测超时）
- Docker Hub（registry-1.docker.io）探测超时，暂不走 Docker
- ffmpeg：`E:\Programs\ffmpeg-master-latest-win64-gpl\bin`
- 建议端口起点 8888（与 Dockerfile 一致），占用则 `start.ps1` 自动 +1
- 分支：`local-custom`；交付物默认不提交

### Windows 本机踩坑（已处理）

- `setuptools>=81` 会去掉 `pkg_resources`，librosa 0.9.1 依赖它：已固定 `uv pip install "setuptools<81"`
- `python -m unidic download` 易卡住（字典源慢/不可达）。本机用 junction 把 `unidic/dicdir` 指到已装的 `unidic_lite/dicdir`（够 MeCab 初始化；日语质量可能略逊完整 UniDic）
- 重建 `.venv` 后需重做：`cmd /c mklink /J .venv\Lib\site-packages\unidic\dicdir .venv\Lib\site-packages\unidic_lite\dicdir`

- Gradio 6 兼容：melo/app.py 已去掉已删除的 show_api / pi_open 参数
- 当前验证：WebUI http://127.0.0.1:8888 返回 200；CLI 已生成 test-en.wav
