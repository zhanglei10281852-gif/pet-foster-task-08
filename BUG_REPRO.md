# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

普通客户虽然看不到用户管理菜单，但直接请求 `/api/user/list` 会拿到全站用户名、手机号和邮箱。这个目录只能由管理员查看，不能把前端隐藏当成权限控制。请修复服务端鉴权，让普通客户稳定收到 403；不要改动现有测试，也禁止绕过权限验证，管理员查询仍须可用。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/pet-foster-task-08
- 仓库地址：https://github.com/zhanglei10281852-gif/pet-foster-task-08.git
- parent SHA：c589284be3e822ef8c9059b77b4fd4cac2b8be59

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/pet-foster-task-08.git bug-repro
cd bug-repro
git checkout --detach c589284be3e822ef8c9059b77b4fd4cac2b8be59
go test ./internal/pet -run ^TestAnnotationUserDirectoryRequiresAdmin$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationUserDirectoryRequiresAdmin$ -count=1
--- FAIL: TestAnnotationUserDirectoryRequiresAdmin (0.26s)
    annotation_pet_behavior_test.go:172: list users error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	0.277s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationUserDirectoryRequiresAdmin$ -count=1
--- FAIL: TestAnnotationUserDirectoryRequiresAdmin (1.03s)
    annotation_pet_behavior_test.go:172: list users error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	1.367s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

普通客户直接访问 /api/user/list 时必须稳定得到 403，响应不得泄露用户名、手机号或邮箱，管理员仍能正常查询用户目录。TestAnnotationUserDirectoryRequiresAdmin 应由修复前失败转为修复后通过，权限相关包测试及全量回归通过；不得改动测试、绕过服务端鉴权或降低角色断言。
