# Hooks 系统

> 学习目标：使用 Hooks 实现自定义逻辑

## 目录

- [Hooks 概述](#hooks-概述)
- [Gateway Hooks](#gateway-hooks)
- [Plugin Hooks](#plugin-hooks)
- [Hook 配置](#hook-配置)
- [创建自定义 Hook](#创建自定义-hook)
- [Hook 最佳实践](#hook-最佳实践)
- [小结](#小结)

---

## Hooks 概述

### 什么是 Hook？

Hook 是在特定事件发生时自动执行的函数，允许你在 OpenClaw 的关键点插入自定义逻辑。

### Hook 生命周期

```
请求 → Hook1 → Hook2 → ... → HookN → 处理
                                      ↓
响应 ← Hook1 ← Hook2 ← ... ← HookN ← 返回
```

### Hook 类型

| 类型 | 触发时机 | 用途 |
|-----|---------|------|
| **Gateway Hooks** | Gateway 级别事件 | Agent 启动、命令执行 |
| **Plugin Hooks** | Agent 级别事件 | 模型解析、工具调用 |
| **Message Hooks** | 消息事件 | 接收、发送、发送后 |
| **Session Hooks** | 会话事件 | 开始、结束 |

---

## Gateway Hooks

### agent:bootstrap

Agent 启动时触发，用于初始化 Agent。

#### 配置

```json
{
  "hooks": {
    "gateway": {
      "agent:bootstrap": {
        "enabled": true,
        "handler": "bootstrapHandler",
        "priority": 10
      }
    }
  }
}
```

#### 实现

```javascript
// hooks/bootstrap.js
async function bootstrapHandler(agent, context) {
  // 加载用户配置
  const userConfig = await loadUserConfig(agent.id);

  // 初始化插件
  await initializePlugins(agent.plugins);

  // 设置默认值
  agent.defaults = {
    model: userConfig.model || 'claude-sonnet-4',
    temperature: userConfig.temperature || 0.7
  };

  return agent;
}

module.exports = { bootstrapHandler };
```

### command Hooks

命令执行前后触发。

#### before:command

命令执行前触发：

```json
{
  "hooks": {
    "gateway": {
      "before:command": {
        "enabled": true,
        "commands": ["message", "sessions"],
        "handler": "beforeCommand"
      }
    }
  }
}
```

```javascript
async function beforeCommand(command, args, context) {
  // 记录命令
  logCommand(command, args);

  // 验证权限
  if (!hasPermission(context.user, command)) {
    throw new Error('Unauthorized');
  }

  // 修改参数
  args.timestamp = new Date();

  return args;
}
```

#### after:command

命令执行后触发：

```javascript
async function afterCommand(command, result, context) {
  // 记录结果
  logResult(command, result);

  // 发送通知
  if (result.error) {
    await notifyError(context.user, result.error);
  }

  return result;
}
```

---

## Plugin Hooks

### before_model_resolve

模型解析前触发，可以修改或替换模型配置。

```json
{
  "hooks": {
    "plugin": {
      "before_model_resolve": {
        "enabled": true,
        "handler": "beforeModelResolve"
      }
    }
  }
}
```

```javascript
async function beforeModelResolve(modelConfig, context) {
  // 根据用户选择模型
  const userPreference = await getUserModel(context.user.id);

  if (userPreference) {
    modelConfig.model = userPreference.model;
    modelConfig.temperature = userPreference.temperature;
  }

  // 根据任务复杂度调整
  const complexity = analyzeComplexity(context.message);
  if (complexity > 0.8) {
    modelConfig.model = 'claude-opus-4';
  }

  return modelConfig;
}
```

### before_prompt_build

构建提示词前触发，可以修改或添加内容。

```json
{
  "hooks": {
    "plugin": {
      "before_prompt_build": {
        "enabled": true,
        "handler": "beforePromptBuild"
      }
    }
  }
}
```

```javascript
async function beforePromptBuild(promptContext, context) {
  // 添加系统上下文
  const systemContext = await loadSystemContext();
  promptContext.system.push(systemContext);

  // 添加用户偏好
  const userPrefs = await getUserPreferences(context.user.id);
  promptContext.user.push(`用户偏好: ${JSON.stringify(userPrefs)}`);

  // 添加项目上下文
  if (context.workspace) {
    const projectInfo = await loadProjectInfo(context.workspace);
    promptContext.project = projectInfo;
  }

  return promptContext;
}
```

### before_tool_call

工具调用前触发，可以验证、修改或阻止工具调用。

```json
{
  "hooks": {
    "plugin": {
      "before_tool_call": {
        "enabled": true,
        "tools": ["exec", "process"],
        "handler": "beforeToolCall"
      }
    }
  }
}
```

```javascript
async function beforeToolCall(toolCall, context) {
  // 验证工具权限
  if (!isToolAllowed(toolCall.name, context.user)) {
    throw new Error(`Tool ${toolCall.name} not allowed for user`);
  }

  // 对于敏感工具，要求确认
  if (['exec', 'process'].includes(toolCall.name)) {
    const confirmed = await requestConfirmation(
      context.user,
      `执行 ${toolCall.name}？`,
      toolCall
    );

    if (!confirmed) {
      throw new Error('Tool call cancelled by user');
    }
  }

  // 记录工具调用
  await auditLog('tool_call', {
    tool: toolCall.name,
    user: context.user.id,
    timestamp: new Date()
  });

  return toolCall;
}
```

### after_tool_call

工具调用后触发，可以处理结果或触发后续操作。

```javascript
async function afterToolCall(toolCall, result, context) {
  // 处理错误
  if (result.error) {
    await handleError(toolCall, result.error, context);
  }

  // 记录成功调用
  await auditLog('tool_success', {
    tool: toolCall.name,
    user: context.user.id,
    duration: result.duration
  });

  // 触发后续操作
  if (toolCall.name === 'write' && result.success) {
    await triggerFileScan(result.path);
  }

  return result;
}
```

---

## 消息和会话 Hooks

### message_received

消息接收后触发。

```json
{
  "hooks": {
    "message": {
      "received": {
        "enabled": true,
        "handler": "onMessageReceived"
      }
    }
  }
}
```

```javascript
async function onMessageReceived(message, context) {
  // 分析消息
  const analysis = analyzeMessage(message);

  // 检测垃圾信息
  if (analysis.isSpam) {
    await blockUser(context.user.id);
    throw new Error('Spam detected');
  }

  // 记录统计
  await recordMessageStats({
    user: context.user.id,
    channel: context.channel,
    type: analysis.type,
    sentiment: analysis.sentiment
  });

  // 修改消息
  message.metadata = {
    ...message.metadata,
    analyzed: true,
    type: analysis.type
  };

  return message;
}
```

### message_sending

消息发送前触发。

```javascript
async function onMessageSending(message, context) {
  // 格式化消息
  message.content = formatMessage(message.content, context.channel);

  // 添加签名
  if (context.agent.signature) {
    message.content += `\n\n— ${context.agent.signature}`;
  }

  // 检查敏感信息
  const filtered = filterSensitiveInfo(message.content);
  if (filtered !== message.content) {
    await notifySensitiveInfoLeak(context.user.id);
    message.content = filtered;
  }

  return message;
}
```

### message_sent

消息发送后触发。

```javascript
async function onMessageSent(message, context) {
  // 更新统计
  await updateMessageStats({
    agent: context.agent.id,
    channel: context.channel,
    length: message.content.length
  });

  // 触发自动化
  if (shouldTriggerAutomation(message, context)) {
    await triggerAutomation(message, context);
  }

  return message;
}
```

### session_start

会话开始时触发。

```javascript
async function onSessionStart(session, context) {
  // 初始化会话
  session.metadata = {
    ...session.metadata,
    startTime: new Date(),
    source: context.channel
  };

  // 发送欢迎消息
  if (session.agent.welcomeMessage) {
    await sendWelcomeMessage(session, context);
  }

  // 加载用户上下文
  const userContext = await loadUserContext(context.user.id);
  session.context = { ...session.context, userContext };

  return session;
}
```

### session_end

会话结束时触发。

```javascript
async function onSessionEnd(session, context) {
  // 保存会话摘要
  const summary = await generateSessionSummary(session);
  await saveSessionSummary(session.id, summary);

  // 更新统计
  await updateSessionStats({
    duration: Date.now() - session.metadata.startTime,
    messageCount: session.messages.length,
    agent: session.agent.id
  });

  // 清理资源
  await cleanupSession(session.id);

  return session;
}
```

---

## Hook 配置

### 全局配置

```json
{
  "hooks": {
    "enabled": true,
    "directory": "~/.openclaw/hooks",
    "autoLoad": true,
    "priority": {
      "default": 10,
      "override": {}
    }
  }
}
```

### Hook 链

Hooks 按优先级顺序执行：

```json
{
  "hooks": {
    "plugin": {
      "before_prompt_build": [
        {
          "handler": "addSystemContext",
          "priority": 10
        },
        {
          "handler": "addUserPreferences",
          "priority": 20
        },
        {
          "handler": "addProjectContext",
          "priority": 30
        }
      ]
    }
  }
}
```

### 条件执行

只在特定条件下执行 Hook：

```json
{
  "hooks": {
    "plugin": {
      "before_tool_call": {
        "enabled": true,
        "condition": {
          "tools": ["exec", "process"],
          "users": ["admin"],
          "channels": ["slack"]
        },
        "handler": "beforeToolCall"
      }
    }
  }
}
```

---

## 创建自定义 Hook

### Hook 结构

```
hooks/
├── package.json
├── index.js
├── hooks/
│   ├── before-tool-call.js
│   ├── after-tool-call.js
│   └── message-handler.js
└── README.md
```

### Hook 实现

```javascript
// hooks/before-tool-call.js
/**
 * 工具调用前 Hook
 */
async function beforeToolCall(toolCall, context) {
  // 实现逻辑
  console.log(`Tool ${toolCall.name} called by ${context.user.id}`);

  // 返回修改后的 toolCall 或抛出错误阻止调用
  return toolCall;
}

module.exports = beforeToolCall;
```

### Hook 导出

```javascript
// index.js
module.exports = {
  'before_tool_call': require('./hooks/before-tool-call'),
  'after_tool_call': require('./hooks/after-tool-call'),
  'message_received': require('./hooks/message-handler')
};
```

### Hook 注册

```json
{
  "hooks": {
    "custom": {
      "before_tool_call": {
        "enabled": true,
        "module": "./hooks/index.js",
        "handler": "before_tool_call"
      }
    }
  }
}
```

---

## Hook 最佳实践

### 1. 幂等性

Hook 应该是幂等的，多次执行产生相同结果：

```javascript
// ✅ 好的做法
async function beforePromptBuild(context) {
  if (!context.systemInitialized) {
    context.system = await loadSystemContext();
    context.systemInitialized = true;
  }
  return context;
}

// ❌ 不好的做法
async function beforePromptBuild(context) {
  context.system.push(await loadSystemContext()); // 重复添加
  return context;
}
```

### 2. 错误处理

妥善处理错误，避免影响整个流程：

```javascript
async function beforeToolCall(toolCall, context) {
  try {
    // 验证逻辑
    await validateToolCall(toolCall);
  } catch (error) {
    // 记录错误但不阻止执行
    logError(error);
    // 或根据严重程度决定是否抛出
    if (error.severe) {
      throw error;
    }
  }
  return toolCall;
}
```

### 3. 性能考虑

避免在 Hook 中执行耗时操作：

```javascript
// ✅ 好的做法 - 异步执行
async function message_sent(message, context) {
  // 不等待完成
  sendAnalytics(message, context).catch(logError);
  return message;
}

// ❌ 不好的做法 - 阻塞执行
async function message_sent(message, context) {
  await sendAnalytics(message, context); // 阻塞
  return message;
}
```

### 4. 优先级设置

合理设置 Hook 优先级：

```javascript
// 高优先级 - 应该先执行
addSystemContext: { priority: 10 },

// 中优先级 - 正常顺序
addUserPreferences: { priority: 50 },

// 低优先级 - 最后执行
addProjectContext: { priority: 100 }
```

### 5. 条件检查

避免不必要的 Hook 执行：

```javascript
async function beforeToolCall(toolCall, context) {
  // 只对特定工具执行
  if (!['exec', 'process'].includes(toolCall.name)) {
    return toolCall; // 快速返回
  }

  // 只对特定用户执行
  if (context.user.role !== 'admin') {
    return toolCall;
  }

  // 执行复杂逻辑
  return await validateToolCall(toolCall);
}
```

---

## 小结

本节我们学习了：

1. ✅ Hooks 概述 - 定义、生命周期、类型
2. ✅ Gateway Hooks - agent:bootstrap, command hooks
3. ✅ Plugin Hooks - before/after 模型解析、提示词构建、工具调用
4. ✅ 消息和会话 Hooks - message_received/sending/sent, session_start/end
5. ✅ Hook 配置 - 全局、链式、条件执行
6. ✅ 创建自定义 Hook - 结构、实现、注册
7. ✅ Hook 最佳实践 - 幂等性、错误处理、性能、优先级、条件检查

## Hook 调试

```bash
# 列出所有 Hooks
openclaw hooks list

# 测试 Hook
openclaw hooks test before_tool_call

# 查看 Hook 日志
openclaw hooks logs --hook before_tool_call

# 调试 Hook
openclaw hooks debug before_tool_call --verbose
```

## 下一步

[内存与上下文管理](./19-memory.md) - 优化 Agent 内存和上下文
