# REST API

Paperless-ngx 现在附带了一个完整文档化的 REST API 和一个可浏览的 Web 界面以供探索。API 的可浏览界面位于 `/api/schema/view/`。

本文档进一步提供了一些端点和功能的说明。

## 授权

REST API 提供四种不同的身份验证方式。

1.  基本身份验证

    通过提供以下形式的 HTTP 头进行授权：

    ```
    Authorization: Basic <credentials>
    ```

    其中 `credentials` 是 `<username>:<password>` 的 base64 编码字符串。

2.  会话身份验证

    当您在浏览器中登录 Paperless 时，您也会自动登录到 API，无需提供任何授权头。

3.  令牌身份验证

    您可以通过在 Web 界面用户下拉菜单中打开“我的个人资料”链接并点击圆形箭头按钮来创建（或重新创建）API 令牌。

    Paperless 还提供了一个端点来获取身份验证令牌。

    将用户名和密码作为表单或 JSON 字符串 POST 到 `/api/token/`，如果登录数据正确，Paperless 将响应一个令牌。此令牌可用于通过以下 HTTP 头对其他请求进行身份验证：

    ```
    Authorization: Token <token>
    ```

    令牌也可以在 Django 管理界面中管理。

4.  远程用户身份验证

    如果启用（请参阅[配置](configuration.md#PAPERLESS_ENABLE_HTTP_REMOTE_USER_API)），您可以使用远程用户身份验证来验证 API。

## 搜索文档

全文搜索在 `/api/documents/` 端点上可用。两个特定的查询参数会导致 API 返回全文搜索结果：

-   `/api/documents/?query=your%20search%20query`：使用全文查询搜索文档。有关语法详情，请参阅[基本用法 - 搜索](usage.md#basic-usage_searching)。
-   `/api/documents/?more_like_id=1234`：搜索与 ID 为 1234 的文档相似的文档。

分页的工作方式与此端点的普通请求完全相同。

此外，每个返回的文档都有一个额外的 `__search_hit__` 属性，其中包含有关搜索结果的各种信息：

```
{
    "count": 31,
    "next": "http://localhost:8000/api/documents/?page=2&query=test",
    "previous": null,
    "results": [

        ...

        {
            "id": 123,
            "title": "title",
            "content": "content",

            ...

            "__search_hit__": {
                "score": 0.343,
                "highlights": "text <span class="match">Test</span> text",
                "rank": 23
            }
        },

        ...

    ]
}
```

-   `score` 表示此文档相对于其他搜索结果与查询的匹配程度。
-   `highlights` 是文档内容的摘录，并使用 `<span>` 标签高亮显示搜索词，如上所示。
-   `rank` 是搜索结果的索引。第一个结果的排名为 0。

### 按自定义字段筛选

您可以通过指定 `custom_field_query` 查询参数来按自定义字段值筛选文档。以下是一些常见用例的示例：

1.  自定义字段 "due"（日期）在 2024 年 8 月 1 日至 2024 年 9 月 1 日（含）之间的文档：

    `?custom_field_query=["due", "range", ["2024-08-01", "2024-09-01"]]`

2.  自定义字段 "customer"（文本）等于 "bob"（区分大小写）的文档：

    `?custom_field_query=["customer", "exact", "bob"]`

3.  自定义字段 "answered"（布尔值）设置为 `true` 的文档：

    `?custom_field_query=["answered", "exact", true]`

4.  自定义字段 "favorite animal"（选择）设置为 "cat" 或 "dog" 的文档：

    `?custom_field_query=["favorite animal", "in", ["cat", "dog"]]`

5.  自定义字段 "address"（文本）为空的文档：

    `?custom_field_query=["OR", [["address", "isnull", true], ["address", "exact", ""]]]`

6.  没有名为 "foo" 的字段的文档：

    `?custom_field_query=["foo", "exists", false]`

7.  具有指向文档 3 和 7 的文档链接 "references" 的文档：

    `?custom_field_query=["references", "contains", [3, 7]]`

所有字段类型都支持基本操作，包括 `exact`、`in`、`isnull` 和 `exists`。字符串、URL 和货币字段支持不区分大小写的子字符串匹配操作，包括 `icontains`、`istartswith` 和 `iendswith`。整数、浮点数和日期字段支持算术比较，包括 `gt` (>)、`gte` (>=)、`lt` (<)、`lte` (<=) 和 `range`。最后，文档链接字段支持 `contains` 操作符，其行为类似于“是超集”检查。

### `/api/search/autocomplete/`

获取部分搜索词的自动补全建议。

查询参数：

-   `term`：不完整的词。
-   `limit`：结果数量。默认为 10。

端点返回的结果按词在文档索引中的重要性排序。第一个结果是索引中具有最高 [Tf/Idf](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) 分数的词。

```json
["term1", "term3", "term6", "term4"]
```

## 上传文档 {#file-uploads}

API 提供了一个用于文件上传的特殊端点：

`/api/documents/post_document/`

向此端点 POST 一个多部分表单，其中表单字段 `document` 包含要上传到 Paperless 的文档。文件名会被清理，然后用于将文档存储在临时目录中，消费者将被指示从那里消费该文档。

该端点支持以下可选表单字段：

-   `title`：指定消费者应用于文档的标题。
-   `created`：指定文档创建的日期时间（例如 "2016-04-19" 或 "2016-04-19 06:15:00+02:00"）。
-   `correspondent`：指定消费者应用于文档的通信人 ID。
-   `document_type`：类似于通信人。
-   `storage_path`：类似于通信人。
-   `tags`：类似于通信人。多次指定此字段可为文档添加多个标签。
-   `archive_serial_number`：要设置的可选档案序列号。
-   `custom_fields`：要分配给文档的自定义字段 ID 数组（值为空）或字段 ID -> 值的映射对象。

如果文档消费过程成功启动，端点将立即返回 HTTP 200，数据为消费任务的 UUID。由于消费过程发生在不同的进程中，因此无法立即获得有关消费过程本身的额外状态信息。但是，使用返回的 UUID 查询任务端点，例如 `/api/tasks/?task_id={uuid}`，将提供有关消费状态的信息，包括消费成功时创建的文档 ID。

## 权限

所有对象（文档、标签等）都允许使用可选的 `owner` 和/或 `set_permissions` 参数设置对象级权限，其格式如下：

```
"owner": ...,
"set_permissions": {
    "view": {
        "users": [...],
        "groups": [...],
    },
    "change": {
        "users": [...],
        "groups": [...],
    },
}
```

!!! note

    数组应包含用户或组的 ID 号。

如果提供了这些参数，对象的权限将被覆盖，前提是经过身份验证的用户有权这样做（用户必须是对象所有者或超级用户）。

### 检索完整权限

默认情况下，API 将返回对象级权限的截断版本，返回 `user_can_change` 指示当前用户是否可以编辑对象（因为他们要么是对象所有者，要么被授予了权限）。您可以将参数 `full_perms=true` 传递给 API 调用，以查看对象的完整权限，其格式与上面的 `set_permissions` 参数类似。

## 批量编辑

API 支持各种异步执行的批量编辑操作。

### 文档

对于文档的批量操作，请使用端点 `/api/documents/bulk_edit/`，它接受以下格式的 JSON 负载：

```json
{
  "documents": [LIST_OF_DOCUMENT_IDS],
  "method": METHOD, // 见下文
  "parameters": args // 见下文
}
```

支持以下方法：

-   `set_correspondent`
    -   需要 `parameters`：`{ "correspondent": CORRESPONDENT_ID }`
-   `set_document_type`
    -   需要 `parameters`：`{ "document_type": DOCUMENT_TYPE_ID }`
-   `set_storage_path`
    -   需要 `parameters`：`{ "storage_path": STORAGE_PATH_ID }`
-   `add_tag`
    -   需要 `parameters`：`{ "tag": TAG_ID }`
-   `remove_tag`
    -   需要 `parameters`：`{ "tag": TAG_ID }`
-   `modify_tags`
    -   需要 `parameters`：`{ "add_tags": [LIST_OF_TAG_IDS] }` 和 `{ "remove_tags": [LIST_OF_TAG_IDS] }`
-   `delete`
    -   不需要 `parameters`
-   `reprocess`
    -   不需要 `parameters`
-   `set_permissions`
    -   需要 `parameters`：
        -   `"set_permissions": PERMISSIONS_OBJ`（参见上面的[格式](#permissions)）和/或
        -   `"owner": OWNER_ID or null`
        -   `"merge": true or false`（默认为 false）
    -   `merge` 标志决定提供的权限是覆盖所有现有权限（包括删除它们）还是与现有权限合并。
-   `edit_pdf`
    -   需要 `parameters`：
        -   `"doc_ids": [DOCUMENT_ID]` 要编辑的单个文档 ID 列表。
        -   `"operations": [OPERATION, ...]` 要在文档上执行的操作列表。每个操作都是一个字典，包含以下键：
            -   `"page": PAGE_NUMBER` 要编辑的页码（从 1 开始）。
            -   `"rotate": DEGREES` 可选旋转角度（90、180、270）。
            -   `"doc": OUTPUT_DOCUMENT_INDEX` 拆分操作的输出文档的可选索引。
    -   可选 `parameters`：
        -   `"delete_original": true` 在编辑后删除原始文档。
        -   `"update_document": true` 用编辑后的 PDF 更新现有文档。
        -   `"include_metadata": true` 将元数据从原始文档复制到编辑后的文档。
-   `remove_password`
    -   需要 `parameters`：
        -   `"password": "PASSWORD_STRING"` 要从 PDF 文档中移除的密码。
    -   可选 `parameters`：
        -   `"update_document": true` 用无密码的 PDF 替换现有文档。
        -   `"delete_original": true` 在编辑后删除原始文档。
        -   `"include_metadata": true` 将元数据从原始文档复制到新的无密码文档。
-   `merge`
    -   不需要额外的 `parameters`。
    -   合并文档的顺序由 ID 列表决定。
    -   可选 `parameters`：
        -   `"metadata_document_id": DOC_ID` 将此文档的元数据（标签、通信人等）应用于合并后的文档。
        -   `"delete_originals": true` 删除原始文档。这要求调用用户是所有被合并文档的所有者。
-   `split`
    -   需要 `parameters`：
        -   `"pages": [..]` 该列表应是一个页面和/或范围的列表，用逗号分隔，例如 `"[1,2-3,4,5-7]"`
    -   可选 `parameters`：
        -   `"delete_originals": true` 在消费后删除原始文档。这要求调用用户是该文档的所有者。
    -   拆分操作只接受单个文档。
-   `rotate`
    -   需要 `parameters`：
        -   `"degrees": DEGREES`。必须是整数，即 90、180、270。
-   `delete_pages`
    -   需要 `parameters`：
        -   `"pages": [..]` 该列表应是一个整数列表，例如 `"[2,3,4]"`
    -   delete_pages 操作只接受单个文档。
-   `modify_custom_fields`
    -   需要 `parameters`：
        -   `"add_custom_fields": { CUSTOM_FIELD_ID: VALUE }`：由自定义字段 id:value 对组成的 JSON 对象，用于添加到文档，也可以是要添加的空值自定义字段 ID 列表。
        -   `"remove_custom_fields": [CUSTOM_FIELD_ID]`：要从文档中移除的自定义字段 ID。

### 对象

对象（标签、文档类型等）的批量编辑目前支持设置权限或删除操作，使用端点：`/api/bulk_edit_objects/`，它需要以下格式的 JSON 负载：

```json
{
  "objects": [LIST_OF_OBJECT_IDS],
  "object_type": "tags", "correspondents", "document_types" or "storage_paths",
  "operation": "set_permissions" or "delete",
  "owner": OWNER_ID, // 可选
  "permissions": { "view": { "users": [] ... }, "change": { ... } }, // （参见上面的 'set_permissions' 格式）
  "merge": true / false // 默认为 false，参见上文
}
```

## API 版本控制

自 Paperless-ngx 1.3.0 起，REST API 已进行版本控制。

-   版本控制确保对 API 的更改不会破坏旧客户端。
-   客户端在每个请求中指定他们希望使用的 API 特定版本，Paperless 将使用指定的 API 版本处理请求。
-   即使底层数据模型发生变化，较旧的 API 版本也将始终提供兼容的数据。
-   如果未指定版本，Paperless 将提供版本 1，以确保与未请求特定 API 版本的旧客户端兼容。

API 版本通过在每个请求中提交一个额外的 HTTP `Accept` 头来指定：

```
Accept: application/json; version=6
```

如果指定了无效版本，Paperless 1.3.0 将响应“406 Not Acceptable”并在正文中显示错误消息。早期版本的 Paperless 将提供 API 版本 1，无论是否通过 `Accept` 头指定了版本。

如果客户端希望验证其是否与任何给定服务器兼容，应执行以下过程：

1.  对任何 API 端点执行*经过身份验证的*请求。如果服务器是 1.3.0 或更高版本，服务器将向响应添加两个自定义头：

    ```
    X-Api-Version: 2
    X-Version: 1.3.0
    ```

2.  根据这些头的存在/不存在以及它们的存在时的值来确定客户端是否与此服务器兼容。

### API 版本弃用策略

较旧的 API 版本保证在新 API 版本发布后至少支持一年。之后，对较旧 API 版本的支持可能会（但不保证）被取消。

### API 变更日志

#### 版本 1

初始 API 版本。

#### 版本 2

-   添加了字段 `Tag.color`。此读/写字符串字段包含十六进制颜色，例如 `#a6cee3`。
-   添加了只读字段 `Tag.text_color`。此字段包含用于特定标签的文本颜色，根据 `Tag.color` 的亮度，该颜色为黑色或白色。
-   移除了字段 `Tag.colour`。

#### 版本 3

-   添加了权限端点。
-   `/api/ui_settings/` 的格式已更改。

#### 版本 4

-   消费模板被重构为工作流，API 端点也相应更改。

#### 版本 5

-   添加了文档和对象的批量删除方法。

#### 版本 6

-   将确认任务端点移至 `/api/tasks/acknowledge/`。

#### 版本 7

-   选择类型自定义字段的格式已更改为将选项作为具有 `id` 和 `label` 字段的对象数组返回，而不是简单的字符串列表。为选择类型自定义字段创建或更新文档的自定义字段值时，该值应为选项的 `id`，而以前是选项的索引。

#### 版本 8

-   文档注释的用户字段现在返回一个简化的用户对象，而不仅仅是用户 ID。

#### 版本 9

-   文档的 `created` 字段现在是日期，而不是日期时间。`created_date` 字段被视为已弃用，并将在未来版本中移除。