# 飞书成员职务字段

飞书通讯录把成员的任职信息拆成若干独立的用户属性。其中**职务**是标准化字段：标准值由管理员在管理后台维护，开放平台只读；把某个职务挂到成员身上则要走成员接口。每个成员只能有一个职务。

## 字段与开放平台能力

| 字段 | 标识 | 标准值来源 | 开放平台 |
|---|---|---|---|
| 职务 | `job_title` | 管理后台维护 | 仅查询；改人走成员接口 |
| 职级 | `job_level_id` | 后台或 API | 增删改查 |
| 序列 | `job_family_id` | 后台或 API | 增删改查 |
| 工作城市 | `city` | 管理后台维护 | 仅查询 |
| 人员类型 | `employee_type` | 后台或 API | 增删改查 |

资源接口：

- 职务 `/open-apis/contact/v3/job_titles`（单个 `/:job_title_id`）
- 职级 `/open-apis/contact/v3/job_levels`
- 序列 `/open-apis/contact/v3/job_families`
- 工作城市 `/open-apis/contact/v3/work_cities`
- 人员类型 `/open-apis/contact/v3/employee_type_enums`

## 职务：后台维护、成员单值

- 标准职务在**管理后台 › 组织架构 › 字段管理 › 字段设置 › 职务**维护，数量上限 1 万；只有「启用」的职务才能生效到成员。
- 职务资源接口**仅支持查询**，不提供增删改。
- 成员的 `job_title` 是**单值字符串**（上限 255），所以一个成员只有一个职务——「每人一个」由此而来。

## 修改成员任职信息

`PATCH /open-apis/contact/v3/users/:user_id`

请求体可写字段：`job_title`（职务名称，传入不存在的名称会自动创建，传空格则清空）、`job_level_id`、`job_family_id`、`city`、`employee_type`、`employee_no`、`leader_user_id` 等。

- 权限：`contact:contact`（更新通讯录）或 `contact:user.base`（更新用户基本信息）。
- 身份：改职务**必须用应用身份（`tenant_access_token`）**。用户身份只允许改 `name`、`en_name`、`avatar_key` 三个字段，改不了职务。
- 频率：1000 次/分钟、50 次/秒（改 `department_ids`、`is_frozen` 时降到 1 QPS）。

## CLI

`lark-cli` 的 `contact` 域只封装了查询（`+get-user`、`+search-user`、`+search-bot`、`user_profiles batch_query`），没有编辑成员的命令，需裸调：

```bash
lark-cli api PATCH /open-apis/contact/v3/users/:user_id \
  --params '{"user_id_type":"open_id"}' \
  --data '{"job_title":"高级工程师"}'
```

## 平台边界

应用 `cli_aae342e9b6389bb4` 未申请职务、职级、序列、工作城市相关 scope，读取这些资源会返回 `99991672`（`app_scope_not_applied`）；人员类型列表可读。要真正写入成员职务，需先在开发者后台补 `contact:contact` 或 `contact:user.base`，并配置通讯录数据权限范围。
