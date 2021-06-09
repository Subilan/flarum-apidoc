# Flarum RESTful API

下面就是对 API 介绍的正文。

## TL;DR

根据原 api.md 的开头，

> Every Flarum forum exposes a publicly-accessible JSON API that can read and write forum data. It conforms to the JSON-API 1.0 specification.

该 API 实际上遵循了 [JSON-API 1.0 规范](https://jsonapi.org/)。如果你不想阅读下面的注解，那么可以从该规范看起，同样可以摸清 API 接受的参数和返回值结构等等。_不过可能需要阅读一些较长的英文文章。_

以下是对该 API 各个方面的详细介绍。

---

## 认证 Authorization

在请求每一个接口之前，都需要带上一个 `Authorization` 信息。有些页面不需要该信息即可访问，那么这时它展示的内容就是针对访客的；而附带认证信息以后，它展示的内容就是针对用户的。

### POST `/api/token`

#### 发送

```json
{
  "identification": "用户名",
  "password": "密码明文"
}
```

#### 返回

```json
{
  "token": "Token 内容",
  "userId": "登入用户的 ID"
}
```

在获取到 Token 以后，若要附带 Authorization 信息，请在 Headers 信息里加入 `Authorization: Token <token>`。以 Java 的 `org.apache.httpcomponents.httpasyncclient` 为例：

```java
var token = "...";
var client = HttpAsyncClients.custom()
                .setDefaultRequestConfig(RequestConfig.custom().setCookieSpec(CookieSpecs.STANDARD).build())
                .setDefaultCookieStore(new BasicCookieStore()).build();
client.start();
var get = new HttpGet("/api/forum");
get.addHeader("Content-Type", "application/vnd.api+json; charset=UTF-8");
get.addHeader("Authorization", "Token " + token);
client.execute(get, new FutureCallback<HttpResponse>() { /* ... */ })
```

## 论坛 Forum

此 API 用于获取和更改论坛的基础信息。

### GET `/api`

返回的是一个描述论坛本身信息的对象。信息包含论坛的名称、介绍、欢迎消息、欢迎标题、网址、LOGO 地址、版本号以及所有可用的标签、用户组等等。

#### 返回

```jsonc
{
  "data": {
    "type": "forums",
    "id": "1",
    "attributes": {
      "title": "...",
      "description": "..."
      // ...
    },
    "relationships": {
      "groups": {
        "data": []
      },
      "tags": {
        "data": []
      }
    }
  },
  "included": [
    {
      "type": "...",
      "id": "...",
      "attributes": {}
    }
  ]
}
```

### POST `/api/settings`

修改论坛的配置。

#### 发送

仅需发送你想要改变的部分。

```jsonc
{
  // 来自 https://论坛/admin#/basics
  "forum_title": "Flarum", // 论坛标题
  "forum_description": "Flarum is ...", // 论坛介绍
  "default_locale": "zh-hans", // 默认语言
  "show_language_selector": 0, // 是否显示语言选择菜单？填 1 或 0
  "welcome_title": "Welcome!", // 论坛欢迎横幅 - 标题
  "welcome_message": "This is my forum.", // 论坛欢迎横幅 - 正文信息
  // 来自 https://论坛/admin#/mail
  "mail_driver": "smtp", // 论坛邮箱驱动
  "mail_from": "noreply@myforum.com", // 发件地址
  "mail_host": "smtp.mailserver.com", // SMTP - 主机
  "mail_port": "465", // SMTP - 端口
  "mail_encryption": "ssl", // SMTP - 加密方式
  "mail_username": "noreply@myforum.com", // SMTP - 用户名
  "mail_password": "123456789", // SMTP - 密码
  "mail_mailgun_secret": "xxx",
  "mail_mailgun_domain": "mailserver.com",
  "mail_mailgun_region": "api.mailgun.net", // 或者 api.eu.mailgun.net
  "mail_mandrill_secret": "xxx",
  // 来自 https://论坛/admin#/appearance
  "theme_primary_color": "#000", // 论坛主色调
  "theme_secondary_color": "#000", // 论坛副色调
  "theme_dark_mode": false, // 是否启用夜间模式
  "theme_colored_header": false, // 是否启用彩色页眉
  "custom_header": "...", // 自定义页眉内容（HTML 代码）
  "custom_footer": "...", // 自定义页脚内容（HTML 代码）
  "custom_less": "..." // 自定义样式内容（LESS 或 CSS 代码）
}
```

#### 返回

不会返回任何值，状态码为 `204 No Content`。如果参数有问题，状态码为 `422 Unprocessable Entity`。

## 主题 Discussions

用于获取和发布主题。

### GET `/api/discussions`

获取主题列表。

#### 参数

- `include=param1,param2,...` **指定包含返回值中的 `discussion` 对象的 `relationships` 包含哪些内容。** 例如，指定 `include=user,lastPostedUser,tags,firstPost`，那么返回值的 `relationships` 项内就会包含这四项内容。
- `filter[q]=?` **指定一个过滤器。** 根据该过滤器来限制返回值。目前已知的过滤器有
  - `is:following` 正在关注的主题（根据 Token 代表的用户而定）。
  - `tag:tagname` 含有指定标签的主题。例如指定 `filter[q]=tag:activity` 那么就只会返回 activity 标签下的内容。注意此处 `tag:tagname` 中的 `tagname` 是当初设置该 tag 时候写的英文名称（设置 tag 的时候会要求填一个标签名称和一个英文名称，标签名称是对外显示出来的，英文名称则用于此处）。
- `page[limit]=?` **设置单页获取数限制。** 由于此接口返回的是整个论坛按时间顺序排列的主题列表，如果主题数目较多，则需要分页。该值用于确定一页内获取多少个主题，默认情况下是 `20`，最大值是 `50`，超过 50 按 50 计。
- `page[offset]=?` **设置页面偏移量。** 利用此值可实现分页，此值应结合 `limit` 使用。此值代表从第一个主题开始要跳过的主题数量。例如指定 `page[offset]=50&page[limit]=50` 则代表从第 51 个主题开始，获取 50 个。再如指定 `page[offset]=200&page[limit]=2` 则代表从 201 个主题开始，获取 2 个，也就是获取第 201 和 202 个主题。

提示：返回的数据中会包含基于当前请求参数的上一页地址、下一页地址和第一页地址，因此不需要手动组合这些地址。

**例**

```
GET /api/discussions?include=user&filter[q]=tag:activity&page[limit]=5&page[offset]=20
```

#### 返回

```jsonc
{
  "links": {
    "first": "https://myforum.com/api/discussions", // 第一页地址
    "next": "https://myforum.com/api/dicussions?xxx", // 下一页地址
    "prev": "https://myforum.com/api/dicussions?xxx" // 上一页地址（当处于第一页时没有此项）
  },
  "data": [
    {
      "type": "discussions",
      "id": "X",
      "attributes": {
        "title": "..."
        // ...
      },
      "relationships": {
        "user": {
          "data": {
            "type": "users",
            "id": "X"
          }
        },
        "lastPostedUser": {
          "data": {
            "type": "users",
            "id": "X"
          }
        }
        // ...
      }
    }
  ]
}
```

### POST `/api/discussions`

创建一个新的主题。

#### 发送

```jsonc
{
  "data": {
    "type": "discussions",
    "attributes": {
      "title": "...", // 标题
      "content": "..." // 正文 Markdown
      // ...
    },
    "relationships": {
      "tags": {
        "data": [
          {
            "type": "tags",
            "id": "X"
          }
          // ...
        ]
      }
      // ...
    }
  }
}
```

#### 返回

```jsonc
{
  "data": {
    "type": "discussions",
    "id": "X",
    "attributes": {
      "title": "xxx",
      "slug": "xxx",
      "createdAt": "1970-01-01T00:00:00+00:00"
    },
    "relationships": {
      // ...
    }
  },
  "included": [
    {
      "type": "posts",
      "id": "X",
      "attributes": {
        // ...
      }
    },
    {
      "type": "users",
      "id": "X",
      "attributes": {
        // ...
      }
    },
    {
      "type": "tags",
      "id": "X",
      "attributes": {
        // ...
      }
    }
  ]
}
```

返回的值就相当于是获取了这个新发的帖子，包含该帖子的完整信息。

### GET `/api/discussions/:id`

获取对应 ID 的主题。

#### 返回

结构与发布后返回的信息相同。

### DELETE `/api/discussions/:id`

删除对应 ID 的主题。

#### 返回

不会返回任何值，状态码为 `204 No Content`。如果不存在，状态码为 `404 Not Found`。

### PATCH `/api/discussions/:id`

修改对应 ID 的主题内容。

#### 发送

结构与创建时的相同，只需要发送你要改变的内容即可。

#### 返回

如果正常，返回的值就相当于是获取了这个修改后的帖子，包含该帖子的完整信息，状态码 `200 OK`。

---

还没写完，有时间再更。
