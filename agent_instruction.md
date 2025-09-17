# Agent.py 技术栈和函数调用逻辑总结

## 项目概述
这是一个基于Google Gemini API的IMO（国际数学奥林匹克）问题求解智能代理系统，采用迭代改进的方式通过AI模型解决数学问题。

## 技术栈

### 核心技术
- **编程语言**: Python 3
- **AI模型**: Google Gemini 2.5 Pro
- **API接口**: Google Generative Language API v1beta

### 依赖库
- **requests**: HTTP客户端，用于API调用
- **json**: JSON数据处理
- **argparse**: 命令行参数解析
- **datetime**: 时间处理
- **os/sys**: 系统操作和文件I/O

### 架构特点
- **异步迭代**: 通过多次迭代改进解决方案
- **自我验证**: 内置验证机制检查解决方案正确性
- **记忆系统**: 支持状态保存和恢复
- **日志系统**: 自定义日志输出到控制台和文件

## 函数调用逻辑

### 1. 主入口流程
```
__main__ → agent() → [init_explorations() | verify_solution()]
```

### 2. 核心函数说明

#### agent() - 主控制器
- **功能**: 整体流程控制，管理迭代改进过程
- **参数**:
  - `problem_statement`: 数学问题描述
  - `other_prompts`: 额外提示列表
  - `memory_file`: 记忆文件路径
  - `resume_from_memory`: 是否从记忆文件恢复
- **流程**:
  1. 检查是否需要从记忆文件恢复
  2. 初始化或恢复状态
  3. 进入迭代循环（最多30次）
  4. 验证解决方案
  5. 根据验证结果进行改进或确认完成

#### init_explorations() - 初始探索
- **功能**: 进行初始问题求解和自我改进
- **流程**:
  1. 构建初始提示（使用step1_prompt）
  2. 调用Gemini API生成第一版解决方案
  3. 使用self_improvement_prompt进行自我改进
  4. 调用verify_solution()验证改进后的解决方案
  5. 返回提示、解决方案、验证结果

#### verify_solution() - 解决方案验证
- **功能**: 使用独立的验证器检查解决方案的数学正确性
- **流程**:
  1. 提取解决方案的详细部分
  2. 构建验证提示（使用verification_system_prompt）
  3. 调用Gemini API进行验证
  4. 解析验证结果，判断是否通过
  5. 返回验证报告和通过状态

### 3. API交互函数

#### build_request_payload() - 构建API请求
- **功能**: 构造Gemini API的JSON请求负载
- **特点**:
  - 支持多轮对话格式
  - 包含系统指令和用户提示
  - 配置生成参数（temperature=0.1, thinkingBudget=32768）

#### send_api_request() - 发送API请求
- **功能**: 执行HTTP POST请求到Gemini API
- **错误处理**: 捕获网络异常和API错误

#### extract_text_from_response() - 提取响应文本
- **功能**: 从API响应JSON中提取生成的文本内容

### 4. 辅助函数

#### 记忆管理
- `save_memory()`: 保存当前状态到JSON文件
- `load_memory()`: 从JSON文件恢复状态

#### 日志系统
- `log_print()`: 自定义打印函数，同时输出到控制台和日志文件
- `set_log_file()`: 设置日志文件
- `close_log_file()`: 关闭日志文件

#### 文件操作
- `read_file_content()`: 读取问题文件内容

#### 文本处理
- `extract_detailed_solution()`: 从解决方案中提取详细部分

### 5. 提示模板系统

#### 核心提示模板
- **step1_prompt**: 主要求解指令，要求严格的数学证明格式
- **self_improvement_prompt**: 自我改进指令
- **correction_prompt**: 基于验证反馈的修正指令
- **verification_system_prompt**: 验证器角色定义和规则
- **verification_remider**: 验证任务提醒

### 6. 工作流程

#### 标准执行流程
1. **启动**: 命令行参数解析，读取问题文件
2. **初始化**: 调用agent()函数
3. **探索**: init_explorations()生成初始解决方案
4. **验证**: verify_solution()检查解决方案正确性
5. **迭代改进**: 如果验证失败，基于反馈重新生成解决方案
6. **收敛判断**: 连续5次验证通过或10次失败后停止
7. **输出**: 返回最终解决方案或失败状态

#### 记忆恢复流程
- 如果提供记忆文件且启用恢复模式
- 从JSON文件加载上次状态
- 从上次迭代位置继续执行

### 7. 错误处理和鲁棒性

#### 异常处理
- API请求异常捕获和重试
- 文件操作异常处理
- JSON解析错误处理

#### 容错机制
- 最大迭代次数限制（30次）
- 验证通过阈值（连续5次）
- 失败容忍度（最多10次验证失败）

### 8. 配置和常量

#### API配置
- `MODEL_NAME`: "gemini-2.5-pro"
- `API_URL`: Google Generative Language API端点
- `generationConfig`: 温度0.1，确保输出一致性

#### 提示模板
- 内置多个预定义提示模板
- 支持动态添加额外提示

## 详细运行流程

### 程序启动流程

#### 1. 命令行解析阶段 (`__main__`)
```
程序启动 → 解析命令行参数 → 设置配置变量
```

**具体步骤：**
1. **参数解析**: 使用`argparse`解析命令行参数
   - `problem_file`: 问题文件路径（必需）
   - `--log/-l`: 日志文件路径（可选）
   - `--other_prompts/-o`: 额外提示，逗号分隔（可选）
   - `--max_runs/-m`: 最大运行次数，默认10（可选）
   - `--memory/-mem`: 记忆文件路径（可选）
   - `--resume/-r`: 是否从记忆文件恢复（可选）

2. **配置设置**:
   ```python
   max_runs = args.max_runs           # 获取最大运行次数
   memory_file = args.memory          # 获取记忆文件路径
   resume_from_memory = args.resume   # 获取恢复标志
   other_prompts = args.other_prompts.split(',') if args.other_prompts else []
   ```

3. **日志初始化**:
   ```python
   if args.log:
       set_log_file(args.log)  # 设置日志文件，开启双重输出（控制台+文件）
   ```

4. **问题文件读取**:
   ```python
   problem_statement = read_file_content(args.problem_file)
   ```

#### 2. 主循环执行阶段
```
for i in range(max_runs):  # 默认最多10次运行
    尝试调用agent() → 如果成功找到解决方案则退出 → 否则继续下一次运行
```

### Agent函数核心流程

#### 3. Agent初始化阶段 (`agent()函数开始`)
```
检查恢复模式 → 状态初始化 → 解决方案生成/恢复
```

**恢复模式判断：**
```python
if resume_from_memory and memory_file:
    memory = load_memory(memory_file)  # 从JSON文件加载状态
    if memory:
        # 恢复所有状态变量
        problem_statement = memory.get("problem_statement", problem_statement)
        other_prompts = memory.get("other_prompts", other_prompts)
        current_iteration = memory.get("current_iteration", 0)
        solution = memory.get("solution", None)
        verify = memory.get("verify", None)
    else:
        # 恢复失败，重新开始
        current_iteration = 0, solution = None, verify = None
else:
    # 全新开始
    current_iteration = 0, solution = None, verify = None
```

#### 4. 初始解决方案生成阶段
```
if solution is None:  # 没有现成解决方案
    调用init_explorations() → 生成初始解决方案
else:  # 从记忆恢复的解决方案
    重新验证解决方案
```

**init_explorations()详细流程：**
```
第1步：构建初始提示 → 第2步：API调用生成解决方案 → 第3步：自我改进 → 第4步：验证解决方案
```

1. **构建初始提示**:
   ```python
   p1 = build_request_payload(
       system_prompt=step1_prompt,      # 主要求解指令
       question_prompt=problem_statement, # 数学问题
       other_prompts=other_prompts      # 用户自定义提示
   )
   ```

2. **第一次API调用** - 生成初始解决方案:
   ```python
   response1 = send_api_request(get_api_key(), p1)
   output1 = extract_text_from_response(response1)
   ```

3. **自我改进** - 第二次API调用:
   ```python
   p1["contents"].append({"role": "model", "parts": [{"text": output1}]})
   p1["contents"].append({"role": "user", "parts": [{"text": self_improvement_prompt}]})
   response2 = send_api_request(get_api_key(), p1)
   solution = extract_text_from_response(response2)
   ```

4. **初始验证**:
   ```python
   verify, good_verify = verify_solution(problem_statement, solution, verbose)
   ```

#### 5. 迭代改进阶段 (最多30次迭代)
```
for i in range(current_iteration, 30):
    验证解决方案 → 根据验证结果进行处理 → 保存状态 → 检查退出条件
```

**迭代循环详细流程：**

1. **状态显示**:
   ```python
   print(f"Number of iterations: {i}, number of corrects: {correct_count}, number of errors: {error_count}")
   ```

2. **验证失败处理**:
   ```python
   if("yes" not in good_verify.lower()):
       correct_count = 0      # 重置正确计数
       error_count += 1       # 增加错误计数
       # 基于验证反馈重新生成解决方案
   ```

3. **解决方案修正** (验证失败时):
   ```python
   # 构建包含问题、当前解决方案和验证反馈的新提示
   p1 = build_request_payload(
       system_prompt=step1_prompt,
       question_prompt=problem_statement,
       other_prompts=other_prompts
   )
   p1["contents"].append({"role": "model", "parts": [{"text": solution}]})
   p1["contents"].append({"role": "user", "parts": [
       {"text": correction_prompt},
       {"text": verify}  # 验证反馈
   ]})
   
   # API调用生成修正解决方案
   response2 = send_api_request(get_api_key(), p1)
   solution = extract_text_from_response(response2)
   ```

4. **重新验证**:
   ```python
   verify, good_verify = verify_solution(problem_statement, solution)
   ```

5. **验证成功处理**:
   ```python
   if("yes" in good_verify.lower()):
       correct_count += 1    # 增加正确计数
       error_count = 0       # 重置错误计数
   ```

6. **状态保存**:
   ```python
   if memory_file:
       save_memory(memory_file, problem_statement, other_prompts, i, 30, solution, verify)
   ```

7. **退出条件检查**:
   ```python
   if(correct_count >= 5):      # 连续5次验证通过
       return solution          # 返回成功解决方案
   elif(error_count >= 10):     # 连续10次验证失败
       return None              # 返回失败
   ```

### 验证子系统流程

#### 6. verify_solution()详细流程
```
提取详细解决方案 → 构建验证提示 → API验证调用 → 解析验证结果 → 二次确认
```

1. **提取详细解决方案**:
   ```python
   dsol = extract_detailed_solution(solution)  # 提取"### Detailed Solution ###"后的内容
   ```

2. **构建验证提示**:
   ```python
   newst = f"""
   ======================================================================
   ### Problem ###
   {problem_statement}
   ======================================================================
   ### Solution ###
   {dsol}
   {verification_remider}
   """
   ```

3. **验证API调用**:
   ```python
   p2 = build_request_payload(
       system_prompt=verification_system_prompt,  # 验证器角色定义
       question_prompt=newst
   )
   res = send_api_request(get_api_key(), p2)
   out = extract_text_from_response(res)
   ```

4. **二次确认** - 判断验证结果是否为正面:
   ```python
   check_correctness = """Response in "yes" or "no". Is the following statement saying the solution is correct, or does not contain critical error or a major justification gap?""" + "\n\n" + out
   prompt = build_request_payload(system_prompt="", question_prompt=check_correctness)
   r = send_api_request(get_api_key(), prompt)
   o = extract_text_from_response(r)  # 得到"yes"或"no"
   ```

5. **生成错误报告** (如果验证失败):
   ```python
   if("yes" not in o.lower()):
       bug_report = extract_detailed_solution(out, "Detailed Verification", False)
   ```

### 关键数据流

#### 7. 数据传递链路
```
命令行参数 → agent()函数参数 → init_explorations() → verify_solution() → 迭代循环
     ↓
problem_statement + other_prompts + memory_file + resume_flag
     ↓
API Payload构建 → Gemini API调用 → 响应解析 → 解决方案文本
     ↓
验证子系统 → 验证报告 + 通过状态 → 迭代决策
     ↓
状态保存 → 记忆文件 + 日志文件
```

#### 8. 状态变量追踪
- **current_iteration**: 当前迭代次数 (0-29)
- **correct_count**: 连续验证通过次数 (目标: ≥5)
- **error_count**: 连续验证失败次数 (限制: <10)
- **solution**: 当前解决方案文本
- **verify**: 当前验证报告
- **good_verify**: 验证是否通过 ("yes"/"no")

### 程序终止条件

#### 9. 成功退出条件
1. **连续5次验证通过**: `correct_count >= 5`
2. **任一运行成功**: 在`max_runs`次运行中有一次成功

#### 10. 失败退出条件
1. **连续10次验证失败**: `error_count >= 10`
2. **达到最大迭代次数**: 30次迭代后仍未成功
3. **达到最大运行次数**: `max_runs`次运行后仍未成功
4. **初始解决方案生成失败**: `init_explorations()`返回None

#### 11. 异常处理
- **API调用异常**: 捕获并继续下一次运行
- **文件读取异常**: 程序退出
- **JSON解析异常**: 抛出异常并处理

### 输出和日志

#### 12. 输出格式
- **控制台输出**: 实时显示进度和状态
- **日志文件**: 完整记录所有输出 (如果指定)
- **记忆文件**: JSON格式保存状态 (如果指定)
- **最终结果**: 成功时输出完整解决方案，失败时输出None

## 使用方法

### 基本用法
```bash
python agent.py problems/imo01.txt
```

### 高级用法
```bash
python agent.py problems/imo01.txt --log solution.log --memory memory.json --resume
```

### 参数说明
- `problem_file`: 问题文件路径
- `--log`: 日志文件路径
- `--memory`: 记忆文件路径
- `--resume`: 从记忆文件恢复
- `--max_runs`: 最大运行次数
- `--other_prompts`: 额外提示（逗号分隔）

### 典型执行时序
```
[启动] → [参数解析] → [日志设置] → [问题读取]
   ↓
[运行1] → [Agent初始化] → [解决方案生成] → [验证循环] → [成功/失败]
   ↓ (如果失败)
[运行2] → [重新开始] → ... → [成功/失败]
   ↓ (重复直到成功或达到max_runs)
[程序结束] → [清理资源]
```

## 设计特点

### 智能化
- 基于AI模型的迭代改进
- 自动验证和错误检测
- 自我学习和修正

### 可靠性
- 多重验证机制
- 状态持久化
- 详细的日志记录

### 扩展性
- 模块化函数设计
- 可配置的提示模板
- 支持自定义验证规则

这个系统展示了如何将大型语言模型应用于复杂数学问题的求解，通过系统化的验证和迭代改进机制提高解决方案的质量和可靠性。
