# 智能行政助手 (Executive AI Assistant)

智能行政助手（EAIA）是一个尝试完成行政助理（EA）工作的AI代理。

如需使用EAIA的托管版本，请参阅[此处](https://mirror-feeling-d80.notion.site/How-to-hire-and-communicate-with-an-AI-Email-Assistant-177808527b17803289cad9e323d0be89?pvs=4)的文档。

目录

- [基本设置](#基本设置)
  - [环境](#环境)
  - [凭证](#设置凭证)
  - [配置](#配置)
- [本地运行](#本地运行)
  - [设置EAIA](#本地设置eaia)
  - [导入邮件](#本地导入邮件)
  - [连接到Agent Inbox](#将本地eaia与agent-inbox连接)
  - [使用Agent Inbox](#使用agent-inbox)
- [生产环境运行 (LangGraph Cloud)](#在生产环境中运行-langgraph-cloud)
  - [在LangGraph Cloud上设置EAIA](#在langgraph-cloud上设置eaia)
  - [手动导入](#手动导入)
  - [设置定时任务](#设置定时任务)

## 基本设置

### 环境

1. 先Fork然后克隆此仓库。注意：确保先进行Fork操作，因为部署时需要使用您自己的仓库。
2. 创建Python虚拟环境并激活它（例如 `pyenv virtualenv 3.11.1 eaia`，`pyenv activate eaia`）
3. 运行 `pip install -e .` 安装依赖项和软件包

### 设置凭证

1. 导出OpenAI API密钥（`export OPENAI_API_KEY=...`）
2. 导出Anthropic API密钥（`export ANTHROPIC_API_KEY=...`）
3. 启用Google服务
   1. [启用API](https://developers.google.com/gmail/api/quickstart/python#enable_the_api)
      - 如果尚未启用Gmail API，请点击蓝色按钮 `Enable the API`
   2. [为桌面应用程序授权凭证](https://developers.google.com/gmail/api/quickstart/python#authorize_credentials_for_a_desktop_application)
      1. 下载客户端密钥。之后，运行以下命令：
      2. `mkdir eaia/.secrets` - 创建一个存放密钥的文件夹
      3. `mv ${PATH-TO-CLIENT-SECRET.JSON} eaia/.secrets/secrets.json` - 将刚刚创建的客户端密钥移动到该密钥文件夹
      4. `python scripts/setup_gmail.py` - 这将在 `eaia/.secrets/token.json` 生成另一个用于访问Google服务的文件。
4. 导出LangSmith API密钥（`export LANGSMITH_API_KEY`）

### 配置

EAIA的配置可以在 `eaia/main/config.yaml` 中找到。其中的每个键都是必需的。以下是配置选项：

- `email`：要监控和发送邮件的电子邮箱。这应与您上面加载的凭证匹配。
- `full_name`：用户全名
- `name`：用户名字
- `background`：用户的基本信息
- `timezone`：用户所在的默认时区
- `schedule_preferences`：日历会议安排的任何偏好。例如会议长度、名称等
- `background_preferences`：回复邮件时可能需要的任何背景信息。例如需要抄送的同事等
- `response_preferences`：在邮件中包含哪些信息的任何偏好。例如是否发送calendly链接等
- `rewrite_preferences`：邮件语调的任何偏好
- `triage_no`：应忽略邮件的指导方针
- `triage_notify`：应通知用户但EAIA不应尝试起草回复的邮件指导方针
- `triage_email`：EAIA应尝试起草回复的邮件指导方针

## 本地运行

您可以在本地运行EAIA。
这对于测试很有用，但如果要实际使用，您需要让它一直运行（以运行检查邮件的定时任务）。
有关如何在生产环境（在LangGraph Cloud上）运行的说明，请参阅[此部分](#在生产环境中运行-langgraph-cloud)

### 本地设置EAIA

1. 安装开发服务器 `pip install -U "langgraph-cli[inmem]"`
2. 运行开发服务器 `langgraph dev`

### 本地导入邮件

现在让我们启动一个导入任务，导入一些邮件并通过本地EAIA处理它们。

保持 `langgraph dev` 命令运行，并打开一个新终端。从那里，回到此目录和虚拟环境。要启动导入任务，请运行：

```shell
python scripts/run_ingest.py --minutes-since 120 --rerun 1 --early 0
```

这将导入过去120分钟内的所有邮件（`--minutes-since`）。如果看到已经处理过的邮件，它不会提前中断（`--early 0`），并且会重新运行之前已经处理过的邮件（`--rerun 1`）。它将针对我们正在运行的本地实例运行。

### 将本地EAIA与Agent Inbox连接

在[本地运行](#本地运行)之后，我们可以与任何结果进行交互。

1. 前往 [Agent Inbox](https://dev.agentinbox.ai/)
2. 将其连接到您本地运行的EAIA代理：
   1. 点击 `Settings`
   2. 输入您的LangSmith API密钥。
   3. 点击 `Add Inbox`
      1. 将 `Assistant/Graph ID` 设置为 `main`
      2. 将 `Deployment URL` 设置为 `http://127.0.0.1:2024`
      3. 给它一个名称，如 `Local EAIA`
      4. 按 `Submit`

现在您可以在Agent Inbox中与EAIA交互。

## 在生产环境中运行 (LangGraph Cloud)

这些说明将介绍如何在LangGraph Cloud中运行EAIA。
您需要一个LangSmith Plus账户才能访问[LangGraph Cloud](https://langchain-ai.github.io/langgraph/concepts/langgraph_cloud/)

如果需要，您始终可以使用LangGraph Platform的[Lite](https://langchain-ai.github.io/langgraph/concepts/self_hosted/#self-hosted-lite)或[Enterprise](https://langchain-ai.github.io/langgraph/concepts/self_hosted/#self-hosted-enterprise)版本以自托管方式运行EAIA。

### 在LangGraph Cloud上设置EAIA

1. 确保您拥有LangSmith Plus账户
2. 导航到LangSmith中的部署页面
3. 点击 `New Deployment`
4. 将其连接到包含此代码的GitHub仓库。
5. 给它一个名称，如 `Executive-AI-Assistant`
6. 添加以下环境变量
   1. `OPENAI_API_KEY`
   2. `ANTHROPIC_API_KEY`
   3. `GMAIL_SECRET` - 这是 `eaia/.secrets/secrets.json` 中的值
   4. `GMAIL_TOKEN` - 这是 `eaia/.secrets/token.json` 中的值
7. 点击 `Submit` 并观察您的EAIA部署

### 手动导入

现在让我们启动一个手动导入任务，导入一些邮件并通过我们的LangGraph Cloud EAIA处理它们。

首先，获取您的 `LANGGRAPH_CLOUD_URL`

要启动导入任务，请运行：

```shell
python scripts/run_ingest.py --minutes-since 120 --rerun 1 --early 0 --url ${LANGGRAPH-CLOUD-URL}
```

这将导入过去120分钟内的所有邮件（`--minutes-since`）。如果看到已经处理过的邮件，它不会提前中断（`--early 0`），并且会重新运行之前已经处理过的邮件（`--rerun 1`）。它将针对我们正在运行的生产实例运行（`--url ${LANGGRAPH-CLOUD-URL}`）

### 将LangGraph Cloud EAIA与Agent Inbox连接

在[部署](#在langgraph-cloud上设置eaia)之后，我们可以与任何结果进行交互。

1. 前往 [Agent Inbox](https://dev.agentinbox.ai/)
2. 将其连接到您的EAIA代理：
   1. 点击 `Settings`
   2. 点击 `Add Inbox`
      1. 将 `Assistant/Graph ID` 设置为 `main`
      2. 将 `Deployment URL` 设置为您的部署URL
      3. 给它一个名称，如 `Prod EAIA`
      4. 按 `Submit`

### 设置定时任务

您可能不想一直手动运行导入。使用LangGraph Platform，您可以轻松设置一个定时任务，按照某个计划运行以检查新邮件。您可以通过以下方式设置：

```shell
python scripts/setup_cron.py --url ${LANGGRAPH-CLOUD-URL}
```

## 高级选项

如果您想控制EAIA的更多方面，而不仅仅是配置允许的内容，您可以修改代码库的部分内容。

**反思逻辑**
要控制用于反思的提示（例如填充记忆），您可以编辑 `eaia/reflection_graphs.py`

**分类逻辑**
要控制用于分类邮件的逻辑，您可以编辑 `eaia/main/triage.py`

**日历逻辑**
要控制用于查看日历上可用时间的逻辑，您可以编辑 `eaia/main/find_meeting_time.py`

**语调和风格逻辑**
要控制用于邮件语调和风格的逻辑，您可以编辑 `eaia/main/rewrite.py`

**邮件草稿逻辑**
要控制用于起草邮件的逻辑，您可以编辑 `eaia/main/draft_response.py`
