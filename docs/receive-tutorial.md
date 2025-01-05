## 使用 Cloudflare email worker 实现的临时电子邮件服务



## 🌈 特点

+   ✨ 隐私友好，无需注册，开箱即用

+   ✨ 更好的 UI 设计，更加简洁

+   🚀 快速部署，无需服务器

原理：

+   Email worker 接收电子邮件

+   前端显示电子邮件（remix）

+   邮件存储（sqlite）

> worker接收电子邮件 -> 保存到数据库 -> 客户端查询电子邮件

## ✨效果

![1736063134197](https://cdn.illsky.com/img/2025/01/202501051545491.jpg)



## 👋  自部署教程

### **准备工作**

+   [**Cloudflare**](https://dash.cloudflare.com/) 账户与托管在 Cloudflare 上的域名

+   [**turso**](https://turso.tech/) sqlite 数据库（个人免费计划足够）

+   [**Vercel**](https://vercel.com/) 账号部署前端用户界面

+   本地安装 [Nodejs](https://nodejs.org/) 环境用于本地运行调式 (版本 >= 18.x)


### **接收邮件教程**

#### turso

注册一个 [turso](https://turso.tech/) 账户，创建一个 `vmail` 数据库，并创建一个 `emails` 表

![image-20250105153649999](https://cdn.illsky.com/img/2025/01/202501051536378.png)

选择您的数据库，您会看到“编辑表”按钮，点击并进入，继续点击左上角的 `SQL runner` 按钮，将 sql 脚本复制到终端运行

```sql
CREATE TABLE `emails` (
 `id` text PRIMARY KEY NOT NULL,
 `message_from` text NOT NULL,
 `message_to` text NOT NULL,
 `headers` text NOT NULL,
 `from` text NOT NULL,
 `sender` text,
 `reply_to` text,
 `delivered_to` text,
 `return_path` text,
 `to` text,
 `cc` text,
 `bcc` text,
 `subject` text,
 `message_id` text NOT NULL,
 `in_reply_to` text,
 `references` text,
 `date` text,
 `html` text,
 `text` text,
 `created_at` integer NOT NULL,
 `updated_at` integer NOT NULL
);
```



#### Cloudflare worker

部署 email worker，需要准备 Node 环境（推荐 18.x 及以上），并且需要安装 wrangler cli 并在本地登录，参考 https://developers.cloudflare.com/workers/wrangler/install-and-update (登录时建议开启VPN)

```
# 安装 pnpm 
npm install -g pnpm
```



```
git clone https://github.com/shuyuncong0/vmail

cd vmail

# 安装依赖
pnpm install
```



在 `vmail/apps/email-worker/wrangler.toml` 文件中填写必要的环境变量。

+   TURSO\_DB\_AUTH\_TOKEN（第1步中的turso表信息，点击“Generate Token”）

+   TURSO\_DB\_URL（例如 libsql://db-name.turso.io）

+   EMAIL\_DOMAIN (域名，如 vmail.dev)

> 如果您不执行此步骤，可以在Cloudflare的 worker settings 中添加环境变量

然后运行命令：

```
cd apps/email-worker

pnpm run deploy
```



#### 电子邮件路由

配置电子邮件路由规则，设置“Catch-all”操作为发送到 email worker：

![image-20250105153844459](https://cdn.illsky.com/img/2025/01/202501051538589.png)

#### Vercel  

**在 Vercel  上部署 Remix 应用程序**

确保在部署时准备并填写以下环境变量（`.env.example`）：

| 变量名                 | 说明                                  | 示例                        |
| ---------------------- | ------------------------------------- | --------------------------- |
| COOKIES_SECRET         | 必填，cookie加密密钥，随机字符串      | `12345abcde`                |
| TURSO_DB_RO_AUTH_TOKEN | 必填，turso数据库只读凭据             | `my-turso-db-ro-auth-token` |
| TURSO_DB_URL           | 必填，turso数据库URL                  | `libsql://db-name.turso.io` |
| EMAIL_DOMAIN           | 必填，域名后缀，支持多个              | `illsky.us.kg`              |
| EXPIRY_TIME            | 可选，邮箱过期时间，单位秒，默认86400 | `86400`                     |
| TURNSTILE_KEY          | 可选，网站验证所需的 Turnstile Key    | `my-turnstile-key`          |
| TURNSTILE_SECRET       | 可选，网站验证所需的 Turnstile Secret | `my-turnstile-secret`       |

获取 `TURNSTILE_KEY`、`TURNSTILE_SECRET` 请前往 cloudflare 控制台 https://dash.cloudflare.com/?to=/:account/turnstile

关于`TURNSTILE_KEY`、`TURNSTILE_SECRET` 配置

![image-20250105154126613](https://cdn.illsky.com/img/2025/01/202501051541883.png)



推荐使用一键部署按钮（此操作会在你的github账户中自动创建vmail仓库并关联部署到vercel）：

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Foiov%2Fvmail&env=COOKIES_SECRET&env=TURNSTILE_KEY&env=TURNSTILE_SECRET&env=TURSO_DB_RO_AUTH_TOKEN&env=TURSO_DB_URL&env=EMAIL_DOMAIN&project-name=vmail&repository-name=vmail)

或手动将代码推送到你的 Github 仓库，并在 Vercel 面板中创建项目。选择 `New project`，然后导入对应的 Github 仓库，填写环境变量，选择 `Remix` 框架，点击 `Deploy`，等待部署完成。

部署完后继续点击 Countinu to Dashboard，进入 Settings -> General，修改下面设置：

![](https://cdn.illsky.com/img/2025/01/202501051542086.png)

一般默认无需修改

![image-20250105154247945](https://cdn.illsky.com/img/2025/01/202501051542071.png)



![image-20250105152442815](https://cdn.illsky.com/img/2025/01/202501051543720.png)

验证环境变量

**然后进入 Deployments 重新部署一次，或向 github 推送代码重新触发部署**。

#### 域名解析

部署成功后在 cloudflare 添加域名解析(A记录)到对应平台，就可以愉快的玩耍了

vercel 演示如何解析：

![image-20250105152636452](https://cdn.illsky.com/img/2025/01/202501051544257.png)

![1736063113885](https://cdn.illsky.com/img/2025/01/202501051545150.jpg)

#### SSL/TLS

在CF域名控制台修改加密模式为完全（或严格）,若不修改，访问网站会出现重定向次数过多错误。

#### 本地调试

```
git clone https://github.com/oiov/vmail
cd vmail
# 安装依赖
pnpm install

# 运行端口 localhost:3000
pnpm run remix:dev
```

运行前复制 `apps/remix/.env.example` 文件并重命名为 `apps/remix/.env`，填写必要的环境变量。

