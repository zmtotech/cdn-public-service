# py

### USER
实战项目版

### ASSISTANT
可以。下面我把你第 2 周、第 3 周的知识点，整理成一个**实战项目版学习文档**。

这个项目不会一下子上真正的大模型 API，先做一个：

# Python 迷你 AI Agent 基础项目
**目标：用文件、JSON、异常处理、模块拆分、requests、dotenv、venv 串起来做一个“伪 Agent”项目。**

你学完并亲手做完这个项目后，会对后面学：

- LLM API 调用
- Tool Calling
- RAG
- AI Agent 工作流

轻松很多。

---

# 一、项目目标

我们要做一个命令行版本的“小助手”，它能：

1. 读取用户输入
2. 根据输入决定调用哪个“工具”
3. 工具可以是：
   - 天气查询（模拟）
   - 搜索请求（调用测试接口）
4. 把结果保存到 JSON 文件
5. 使用 `.env` 管理配置
6. 用 `try / except` 处理异常
7. 拆分为多个 Python 文件

---

# 二、你会练到哪些知识点

这个项目会覆盖你前两周要掌握的核心能力：

## 第 2 周
- 文件读写
- JSON 处理
- 异常处理
- 模块与代码组织

## 第 3 周
- requests 发 HTTP 请求
- dotenv 读取环境变量
- 虚拟环境与依赖管理

---

# 三、项目效果预览

运行程序后，你可以这样交互：

```bash
请输入你的问题：帮我搜索 AI Agent
```

输出可能是：

```bash
识别到任务类型：search
执行结果：
搜索结果：你输入的是“帮我搜索 AI Agent”
结果已保存到 result.json
```

或者：

```bash
请输入你的问题：上海天气怎么样
```

输出：

```bash
识别到任务类型：weather
执行结果：
上海今天天气晴朗，温度 26°C
结果已保存到 result.json
```

---

# 四、项目结构

建议你按下面结构创建项目：

```bash
mini_agent_project/
│
├── main.py
├── agent.py
├── tools.py
├── utils.py
├── config.py
├── .env
├── requirements.txt
└── result.json   # 运行后自动生成
```

---

# 五、先准备环境

## 1）创建项目文件夹
```bash
mkdir mini_agent_project
cd mini_agent_project
```

---

## 2）创建虚拟环境
```bash
python -m venv venv
```

---

## 3）激活虚拟环境

### Windows
```bash
venv\Scripts\activate
```

### macOS / Linux
```bash
source venv/bin/activate
```

---

## 4）安装依赖
```bash
pip install requests python-dotenv
```

---

## 5）导出依赖
```bash
pip freeze > requirements.txt
```

---

# 六、先理解这个项目的核心思路

这个“迷你 Agent”其实就是：

- **用户输入**
- **一个简单判断逻辑**
- **选择不同工具函数**
- **拿到结果**
- **保存结果**

这和真正 AI Agent 的结构已经很接近了：

- LLM 负责理解用户意图
- Agent 决定调用哪个工具
- Tool 执行具体操作
- 返回结果给用户
- 记录日志/结果

只是我们现在先不用大模型，先用规则判断代替。

---

# 七、逐个创建文件

---

## 1. `.env`

在项目根目录创建 `.env` 文件：

```env
SEARCH_API_URL=https://httpbin.org/post
DEFAULT_CITY=上海
```

### 作用
- 存放配置
- 模拟以后存 API 地址、API Key 的方式

---

## 2. `config.py`

负责读取环境变量。

```python
from dotenv import load_dotenv
import os

load_dotenv()

SEARCH_API_URL = os.getenv("SEARCH_API_URL")
DEFAULT_CITY = os.getenv("DEFAULT_CITY", "北京")
```

### 你学到的点
- `dotenv`
- `os.getenv()`
- 配置和业务逻辑分离

---

## 3. `utils.py`

负责保存 JSON 文件。

```python
import json

def save_json(data, filename="result.json"):
    with open(filename, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
```

### 你学到的点
- 文件写入
- JSON 保存
- 工具函数封装

---

## 4. `tools.py`

这里定义两个“工具”：

- `get_weather(city)`：模拟天气工具
- `search_web(query)`：通过请求测试接口模拟搜索工具

```python
import requests
from config import SEARCH_API_URL

def get_weather(city):
    return {
        "tool": "weather",
        "city": city,
        "result": f"{city}今天天气晴朗，温度 26°C"
    }

def search_web(query):
    payload = {
        "query": query
    }

    response = requests.post(SEARCH_API_URL, json=payload, timeout=5)
    response.raise_for_status()

    return {
        "tool": "search",
        "query": query,
        "result": f"搜索结果：你输入的是“{query}”",
        "raw_response": response.json()
    }
```

---

### 你学到的点
- requests 的 `post`
- 传 JSON
- `timeout`
- `raise_for_status()`
- 返回字典结构

---

## 5. `agent.py`

这里写“Agent 的决策逻辑”。

当前先用简单规则判断：

- 如果用户输入里有“天气”，调用天气工具
- 如果用户输入里有“搜索”，调用搜索工具
- 否则返回无法识别

```python
from tools import get_weather, search_web
from config import DEFAULT_CITY

def detect_task(user_input):
    if "天气" in user_input:
        return "weather"
    elif "搜索" in user_input:
        return "search"
    else:
        return "unknown"

def run_agent(user_input):
    task_type = detect_task(user_input)

    if task_type == "weather":
        city = DEFAULT_CITY
        for c in ["上海", "北京", "广州", "深圳", "杭州"]:
            if c in user_input:
                city = c
                break
        return {
            "task_type": task_type,
            "data": get_weather(city)
        }

    elif task_type == "search":
        return {
            "task_type": task_type,
            "data": search_web(user_input)
        }

    else:
        return {
            "task_type": "unknown",
            "data": {
                "tool": "none",
                "result": "暂时无法识别你的请求，请尝试输入包含“天气”或“搜索”的问题。"
            }
        }
```

---

### 你学到的点
- 模块导入
- 函数调用函数
- 条件判断
- 返回统一结构

---

## 6. `main.py`

主程序入口。

```python
from agent import run_agent
from utils import save_json

def main():
    user_input = input("请输入你的问题：").strip()

    try:
        result = run_agent(user_input)

        print(f"识别到任务类型：{result['task_type']}")
        print("执行结果：")
        print(result["data"]["result"])

        save_json(result)
        print("结果已保存到 result.json")

    except Exception as e:
        print("程序运行出错：", e)

if __name__ == "__main__":
    main()
```

---

### 你学到的点
- 程序入口
- `try / except`
- 用户输入
- 调用主逻辑
- 保存结果

---

# 八、完整运行流程

当你执行：

```bash
python main.py
```

程序会：

1. 等待你输入问题
2. 根据输入调用 `agent.py`
3. `agent.py` 判断应该用哪个工具
4. 调用 `tools.py`
5. 拿到结果后回到 `main.py`
6. 打印结果
7. 通过 `utils.py` 保存到 `result.json`

---

# 九、运行示例

---

## 示例 1：天气查询

输入：

```bash
请输入你的问题：上海天气怎么样
```

输出：

```bash
识别到任务类型：weather
执行结果：
上海今天天气晴朗，温度 26°C
结果已保存到 result.json
```

对应 `result.json` 可能是：

```json
{
  "task_type": "weather",
  "data": {
    "tool": "weather",
    "city": "上海",
    "result": "上海今天天气晴朗，温度 26°C"
  }
}
```

---

## 示例 2：搜索请求

输入：

```bash
请输入你的问题：帮我搜索 AI Agent
```

输出：

```bash
识别到任务类型：search
执行结果：
搜索结果：你输入的是“帮我搜索 AI Agent”
结果已保存到 result.json
```

---

## 示例 3：无法识别

输入：

```bash
请输入你的问题：你好
```

输出：

```bash
识别到任务类型：unknown
执行结果：
暂时无法识别你的请求，请尝试输入包含“天气”或“搜索”的问题。
结果已保存到 result.json
```

---

# 十、这个项目对应真实 AI Agent 的哪里？

虽然这是“简化版”，但你已经在练真实 AI Agent 的核心结构了。

## 当前这个项目
- `detect_task()`：相当于“意图识别”
- `run_agent()`：相当于“Agent 控制器”
- `get_weather()` / `search_web()`：相当于“工具函数”
- `result.json`：相当于“结果记录 / 日志”
- `.env`：相当于“配置管理”

## 以后升级
你会把：

- 规则判断  
替换成  
- LLM 判断调用哪个工具

你会把：

- 模拟天气结果  
替换成  
- 真正调用天气 API

你会把：

- 简单搜索接口  
替换成  
- 真正的搜索 API / 企业知识库 / 向量检索

所以这一步非常值得做。

---

# 十一、你在这个项目里必须掌握的关键点

---

## 1）文件与 JSON
你要理解：

- 为什么 `save_json()` 要单独放在 `utils.py`
- 为什么 `result.json` 是结构化保存，而不是只写一行文本

---

## 2）异常处理
你要理解：

- 为什么 `main()` 里要加 `try / except`
- 为什么网络请求可能失败
- 为什么程序不能一出错就直接崩掉

---

## 3）模块拆分
你要理解：

- `main.py` 不要什么都写
- `agent.py` 负责决策
- `tools.py` 负责执行工具
- `config.py` 负责读取配置
- `utils.py` 负责通用功能

这就是工程化思维的开始。

---

## 4）requests
你要理解：

- `requests.post()` 是怎么发 JSON 的
- `response.json()` 为什么能变成字典
- `timeout` 和 `raise_for_status()` 的意义

---

## 5）dotenv
你要理解：

- 为什么 URL、API Key 不要写死
- `.env` 的作用是什么

---

# 十二、建议你亲手做的 4 个小改造

完成基础版后，不要立刻停。  
你最好做这 4 个增强练习。

---

## 改造 1：支持更多城市
把城市列表扩展成：

```python
["上海", "北京", "广州", "深圳", "杭州", "南京", "成都"]
```

如果输入包含这些城市，就返回对应城市天气。

---

## 改造 2：增加一个新工具
比如增加一个 `get_todo()` 工具：

```python
def get_todo():
    return {
        "tool": "todo",
        "result": "你今天的待办事项：学习 Python、练习 Agent 项目"
    }
```

然后在 `detect_task()` 中支持“待办”。

这一步非常接近真实 Agent 扩展工具的方式。

---

## 改造 3：把结果追加保存到日志文件
现在是覆盖写入 `result.json`，你可以增加一个 `log.txt`：

```python
def save_log(text, filename="log.txt"):
    with open(filename, "a", encoding="utf-8") as f:
        f.write(text + "\n")
```

每次运行都写一条日志。

---

## 改造 4：单独处理 requests 错误
你可以把 `search_web()` 改成：

```python
import requests
from config import SEARCH_API_URL

def search_web(query):
    try:
        payload = {"query": query}
        response = requests.post(SEARCH_API_URL, json=payload, timeout=5)
        response.raise_for_status()

        return {
            "tool": "search",
            "query": query,
            "result": f"搜索结果：你输入的是“{query}”",
            "raw_response": response.json()
        }
    except requests.exceptions.RequestException as e:
        return {
            "tool": "search",
            "query": query,
            "result": f"搜索请求失败：{e}"
        }
```

这会更接近真实项目。

---

# 十三、建议你按这个顺序练习

## 第一步
先把我给你的基础代码完整敲一遍，确保能运行。

## 第二步
自己不看文档，重新写一次：

- `main.py`
- `agent.py`
- `tools.py`

## 第三步
做 2 个增强功能：
- 新增一个工具
- 新增错误处理

## 第四步
自己画出调用流程图：

```text
用户输入 -> agent判断 -> 工具执行 -> 返回结果 -> 保存文件
```

如果你能自己讲清楚这个流程，说明你真的理解了。

---

# 十四、你做完这个项目后，算达到什么水平？

做完这个项目后，你已经具备了进入下一阶段的基础：

## 你已经会
- 基础项目结构
- 配置管理
- 工具函数封装
- 文件与 JSON 读写
- requests 请求
- 异常处理
- 基础“Agent 控制器”思维

## 你下一步就可以开始接触
- 调用真正的 LLM API
- 使用大模型决定工具调用
- RAG 文档读取
- LangChain / LangGraph 的基础概念

---

# 十五、下一步升级方向

这个项目做完后，最自然的升级有 3 条路：

## 路线 A：接入真正的大模型 API
把 `detect_task()` 从关键词判断升级成：
- 调用 LLM 判断用户意图

## 路线 B：接入真实天气 / 搜索 API
把模拟工具升级成真实工具

## 路线 C：增加多轮对话记录
把用户历史消息保存起来，模拟“记忆”

---

# 十六、一句话总结

这个项目本质上是在训练你：

> **用 Python 把“输入 → 判断 → 调工具 → 返回结果 → 保存输出”这一条 Agent 基础链路走通。**

这正是你后面转向 AI Agent 开发最需要的底层能力。

---

如果你愿意，我下一步可以直接继续给你两种内容之一：

## 方案 A：把这个项目升级成“接入真实 OpenAI API 的第一版”
适合你开始接触真正 LLM 调用。

## 方案 B：给你一份“实战练习清单 + 标准答案”
你可以像做作业一样一步一步练。  

如果你想，我建议下一步直接做：  
**“把这个迷你 Agent 升级成接入真实 LLM API 的版本”**。