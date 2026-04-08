# robot-paper-daily

这个项目用于抓取 arXiv `cs.RO` 分类下的近期论文，调用 LLM 生成中文总结和相关性评分，并输出两份展示结果：

- `README.md`：Markdown 论文列表
- `index.html`：静态网页展示页

注意：仓库现有的 `README.md` 是脚本自动生成的结果文件，不适合作为项目说明文档；本文件 `readme.md` 才是使用说明。

## 项目结构

- `paper_daily.py`：抓取 arXiv、提取摘要/引言/相关工作、调用 LLM、写入 JSON 和 `README.md`
- `create_index.py`：把 JSON 渲染为 `index.html`
- `template.html`：网页模板
- `arxiv_cs_ro_papers_final.json`：抓取和总结后的原始数据
- `.github/workflows/paper.yml`：GitHub Actions 定时任务

## 运行逻辑

1. 从 `https://arxiv.org/list/cs.RO/recent?show=100` 抓取最新论文
2. 进入论文 HTML 页面提取摘要、引言、相关工作
3. 调用 LLM 生成中文总结和 1-5 分相关性评分
4. 输出到 `arxiv_cs_ro_papers_final.json`
5. 生成 `README.md`
6. 再生成 `index.html`

## 本地使用

### 1. 安装依赖

```bash
cd /home/hanyu/Codes/robot-paper-daily
python -m pip install --upgrade pip
pip install arxiv requests beautifulsoup4 friendly-traceback
```

### 2. 配置环境变量

```bash
export LLM_API_KEY='你的 API Key'
export LLM_PROMPT='你的提示词'
```

当前代码默认请求：

- Host: `api.chatanywhere.org`
- Endpoint: `/v1/chat/completions`
- Model: `gpt-4o-mini`

如果 `LLM_API_KEY` 没有设置，`paper_daily.py` 会直接退出。

### 3. 运行抓取和生成

```bash
cd /home/hanyu/Codes/robot-paper-daily
python paper_daily.py
python create_index.py
```

运行完成后，仓库根目录会更新：

- `arxiv_cs_ro_papers_final.json`
- `README.md`
- `index.html`

### 4. 本地查看网页

```bash
cd /home/hanyu/Codes/robot-paper-daily
python -m http.server 8000
```

然后打开：

`http://127.0.0.1:8000/index.html`

## GitHub Actions 自动运行

工作流文件：

- `.github/workflows/paper.yml`

当前流程会：

- 每天定时运行一次
- 或者手动触发 `workflow_dispatch`
- 安装 Python 依赖
- 运行 `paper_daily.py` 和 `create_index.py`
- 把生成后的文件提交回仓库

你需要在 GitHub 仓库设置里配置：

### Secrets

- `LLM_API_KEY`
- `GIT_TOKEN`

### Variables

- `LLM_PROMPT`

## 建议你 fork 后顺手修改的内容

在 `.github/workflows/paper.yml` 里，原作者留下了默认身份：

- `GITHUB_USER_NAME`
- `GITHUB_USER_EMAIL`

建议改成你自己的 GitHub 用户名和邮箱，否则自动提交会继续显示原作者身份。

## 常见问题

### 1. 为什么仓库里的 `README.md` 不是说明文档？

因为这个项目把论文日报直接输出到了 `README.md`，这样仓库首页就会直接显示最新论文列表。

### 2. 为什么有时候会看到“大模型总结失败”？

通常是以下原因之一：

- `LLM_API_KEY` 无效
- LLM 接口超时或限流
- 上游接口返回非 200 状态码

### 3. 为什么跑完后只有部分论文？

当前代码里 `MAX_CRAWL_PAGES = 1`，意味着只抓取有限页数；如需更多数据，需要改脚本配置。

## 当前仓库路径

本地路径：

`/home/hanyu/Codes/robot-paper-daily`
