---
name: frontend-contract-style
description: 统一前端接口契约和实现风格。适用于用户提到“接口字段改了”“前端参数要和后端一致”“改请求体/响应结构”“接口字段重命名”“去掉兼容写法”“统一提交参数”“批量调整调用点”“顺手把代码风格收敛一下”等场景，也适用于 request payload、query 参数、response handling、hook/store/component 调用链的一次性标准化改造。
---

# Frontend Contract Style

前端任务只要涉及接口契约、参数命名、响应处理或实现风格收敛，就按下面规则执行。

## 执行流程

1. 先确认唯一标准契约，再开始改代码。
2. 先全局搜索旧字段、旧方法、旧接口调用点，再做修改。
3. 只允许两种结果：
   - 所有调用点全部切到新契约。
   - 只保留新契约，并立即删除旧路径。
4. 除非用户明确要求，否则不要保留兼容映射、fallback 字段、双写参数、过渡适配层。
5. 改完后用最小必要的搜索、构建或测试做一次验证。

## 契约约束

- 用 `rg` 搜 request 方法、组件调用、store、hook、常量和同接口相关的所有入口。
- 字段改名要在调用点同步修改，不要在 API 层做“旧字段转新字段”的中间翻译。
- 请求体、query 参数、函数签名、响应字段要和后端标准契约完全一致。
- 新契约落地后，立即删除废弃分支、旧字段、旧 helper。

## 风格约束

- 保持项目现有的参数传递、命名、导入、请求封装和文件组织风格。
- 优先写短而直接的代码，不要为了兼容历史契约再加一层抽象。
- 相关常量、状态、辅助逻辑、提交方法尽量放在相邻位置，不要四处散落。
- 文件需要分段时，用简短行注释划分逻辑块。
- 在你改动的代码里，给变量声明和方法定义补简短注释；如果项目现有风格明显不这么写，则优先跟随项目。
- 注释只解释用途，不解释语法。

## 搜索方式

```powershell
rg "oldFieldName|oldMethodName|targetEndpoint|requestFunction" src
```

修改前先搜，修改后再搜一遍，确认旧名字已经清理干净。

## 推荐改法

```js
// ==================== api ====================
/**
 * Submit runner audit result.
 */
export function doAudit(data) {
  // Request payload.
  return post('/runner/admin/runner/audit/doAudit', data);
}

// ==================== submit ====================
/**
 * Submit audit action with the standard backend contract.
 */
async function handleSubmit() {
  // Audit payload.
  const payload = {
    // Record id.
    id: form.id,
    // Audit status.
    certStatus: form.certStatus,
    // Reject reason.
    rejectReason: form.rejectReason,
  };

  await doAudit(payload);
}
```

## 最终检查

- 重新执行 `rg`，确认旧字段、旧 helper、旧参数名都已移除。
- 条件允许时，对受影响页面或模块做一次最小构建或测试。
- 明确说明是否还有未验证路径。
