# 纵深防御式校验

## 概述

修复无效数据引起的缺陷时，只在一处加校验看起来就够了。不同的代码路径、重构或 mock 都可能绕过这一处检查。

**核心原则** 在数据经过的每一层都进行校验，从结构上杜绝缺陷。

## 多层校验的作用

单处校验解决当前复现方式，多层校验可以阻断更多触发路径。每一层能发现的问题各有侧重。

- 入口校验拦截大多数无效输入。
- 业务逻辑检查边界情况。
- 环境保护阻止特定环境下的危险操作。
- 其他层失效时，调试日志可帮助定位。

## 四层校验

### 第 1 层：入口校验

**目的** 在 API 边界拒绝明显无效的输入。

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ……继续处理
}
```

### 第 2 层：业务逻辑校验

**目的** 确认数据满足当前操作的要求。

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ……继续处理
}
```

### 第 3 层：环境保护

**目的** 阻止在特定环境中执行危险操作。

```typescript
async function gitInit(directory: string) {
  // 在测试中，拒绝在临时目录之外运行 git init
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ……继续处理
}
```

### 第 4 层：调试日志

**目的** 捕获现场信息以供排查。

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ……继续处理
}
```

## 应用方法

发现缺陷时：

1. **追踪数据流。** 无效值来自哪里？用在何处？
2. **找出所有检查点。** 列出数据流经的每一处。
3. **在各层补齐检查。** 依次覆盖入口、业务逻辑、环境保护和调试信息。
4. **测试每一层。** 尝试绕过第 1 层，验证第 2 层能否拦截，依此类推。

## 会话示例

缺陷：空的 `projectDir` 导致 `git init` 在源码目录运行。

**数据流**

1. 测试设置 → 空字符串
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init` 在 `process.cwd()` 中运行

**新增的四层措施**

- 第 1 层。`Project.create()` 校验目录非空、存在且可写。
- 第 2 层。`WorkspaceManager` 校验 `projectDir` 非空。
- 第 3 层。`WorktreeManager` 拒绝在测试的临时目录之外运行 `git init`。
- 第 4 层。在 `git init` 之前记录堆栈。

**结果** 全部 1847 项测试通过，缺陷无法复现。

## 各层发现的问题

测试中，四层各自发现了其他层遗漏的问题。

- 不同代码路径绕过了入口校验。
- mock 绕过了业务逻辑检查。
- 不同平台的边界情况需要环境保护。
- 调试日志揭示了代码结构上的误用。

**在每一层补齐检查。**
