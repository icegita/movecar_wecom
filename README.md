# MoveCar_Wecom - 挪车通知系统（多车版）

基于 Cloudflare Workers 的智能挪车通知系统，扫码即可通知车主，保护双方隐私。

> 本版基于 [原项目](https://github.com/lesnolie/movecar) 进行优化调整，参考 [movecar2](https://github.com/Misakiuri64/movecar2) 的多车架构思路，核心架构不变，进行了多项改进。
> 
> 🙏 感谢 [lesnolie/movecar](https://github.com/lesnolie/movecar) 和 [Misakiuri64/movecar2](https://github.com/Misakiuri64/movecar2) 提供的开源基础与设计参考。

## 界面预览

|                        路人页面                         |                         车主页面                         |
| :-------------------------------------------------------: | :------------------------------------------------------: |
| [🔗 在线预览](https://htmlpreview.github.io/?https://github.com/icegita/movecar_wecom/blob/main/sender_preview.html) | [🔗 在线预览](https://htmlpreview.github.io/?https://github.com/icegita/movecar_wecom/blob/main/owner_preview.html) |

## 个人向修改:

### 界面修改

底部"一键通知车主"按钮样式修改,更醒目

### 功能修改

#### 获取位置信息
原版弹窗只有我知道了按钮,默认获取位置,调整为2个单独按钮,由用户自行选择;

#### 增加通知进度入口

原版一旦离开页面,再次扫描只能重新发起流程,增加了查看进度的入口,10分钟内可显示当前任务接口;

#### 多车支持

支持同一 Worker 实例管理多辆车，每辆车绑定独立的企业微信群 + 联系电话：

- **URL 嵌车牌**：二维码链接附带 `?plate=沪A888666`，扫码即对应到该车
- **按车牌隔离**：冷却时间、通知状态、位置信息均按车牌独立存储
- **每车独立通知**：A 车的推送发到 A 的微信群，B 车发到 B 的群
- **向下兼容**：不传 `?plate=` 参数时保持原单车逻辑不变（使用 `WECOM_WEBHOOK` + `PHONE_NUMBER`）
<img width="465" height="741" alt="232814" src="https://github.com/user-attachments/assets/0492d414-f079-4108-954d-9dff98cb894a" />
<img width="465" height="800" alt="232833" src="https://github.com/user-attachments/assets/561b6e8b-2785-441e-85ac-fbf33a65ad46" />
<img width="445" height="673" alt="PixPin_2026-05-29_15-11-54" src="https://github.com/user-attachments/assets/49621fa9-113b-4020-939c-3794fa19ae93" />







## 与原版的主要区别

### 🔔 推送服务：~~Bark → Server酱~~ 修改为企业微信

~~原版使用 Bark（仅 iOS），本版改用 **Server酱** 推送至微信，适合国内安卓用户。~~

- 企业微信适合国内用户，不区分安卓苹果，非常方便
- 推送正文支持 Markdown，确认链接在微信消息内可直接点击跳转

### ⏱️ 无位置信息时需等待30秒倒计时结束才能点击按钮

未分享位置时需等待30秒倒计时结束才能点击按钮，相比延迟发送此方式更方便用户理解。弹窗说明和 Toast 提示均已同步更新。

### ✏️ 输入框：textarea → contenteditable

将普通文本框改为富文本编辑区，支持在同一输入框内混合显示普通文本和**红色加粗的加急提示**。

### 🏷️ 标签交互逻辑重写

原版四个标签（挡路 / 临停 / 没接 / 加急）点击后直接覆盖输入内容，本版进行了完整重写：

| 功能                 | 原版                           | 个人向定制版                               |
| -------------------- | ------------------------------ | ------------------------------------ |
| 标签数量             | 4 个（含"没接"）               | 2 个                                 |
| ~~加急~~             | ~~覆盖全部内容~~               | ~~追加到文本末尾，红色加粗~~        |
| 已输入内容           | 点击标签会清空文本套用模板文字 | **在光标处插入**，不影响已有内容     |
| 先点加急再点其他标签 | 加急文本被替换为其他模板文字   | **检测光标位置**，插入到模板文字之前 |
| 选中反馈             | 无                             | 选中标签变色                         |

### 🎨 其他界面调整

- 预览链接添加交互
- 三个标签**整体居中**，间距等比自适应屏幕，支持小屏换行
- 位置获取失败时提示文案更具体："刷新页面重试或复制链接到浏览器打开"，（微信网页刷新后保持位置获取状态）
- 模板字符串加入 `/*html*/` 标记，VS Code 可对内嵌 HTML 完整高亮

## 部署步骤

### 第一步：注册 Cloudflare 账号

1. 打开 https://dash.cloudflare.com/sign-up
2. 输入邮箱和密码，完成注册

### 第二步：创建 Worker

1. 登录完成邮箱验证后，点击左侧菜单『Workers & Pages』
2. 点击『Create application』
3. 选择『Start with Hello World!』
4. 点击『Deploy』
5. 点击『Edit code』，删除默认代码
6. 复制 `movecar_android.js` 全部内容，粘贴
7. 点击右上角『Deploy』保存

### 第三步：创建 KV 存储

1. 左侧菜单『Storage & databases』→『Workers KV』
2. 点击『Create instance』
3. 名称填 `MOVE_CAR_STATUS`，点击『Create』
4. 回到你的 Worker →『Bindings』
5. 点击『Add binding』,左侧选择『KV Namespace』→『Add binding』
6. Variable name 填 `MOVE_CAR_STATUS`，KV Namespace选择刚创建的 namespace`MOVE_CAR_STATUS`，点击『Add binding』

### 第四步：配置环境变量

~~_!!! 请先前往 [sct.ftqq.com](https://sct.ftqq.com) 微信扫码登录获取 SendKey_~~

~~在 Cloudflare Worker → Settings → Variables and Secrets 中添加：~~

登录企业微信建立群，点击右上方三个点>消息推送>点击机器人复制其Webhook地址，

**单车模式（原有）：**

| Variable name | Value | 是否必填 |
|---|---|---|
| `WECOM_WEBHOOK` | 企业微信群机器人 Webhook | 必填 |
| `PHONE_NUMBER` | 车主联系电话 | 可选 |

**多车模式（推荐）：**

将上述两个变量替换为 `CAR_LIST` 一个变量：

| Variable name | Value | 是否必填 |
|---|---|---|
| `CAR_LIST` | CSV 格式的多车配置（Secret 类型） | 必填 |

`CAR_LIST` 格式：每行一辆车，三列 `车牌号,Webhook地址,联系电话(可选)`

```
沪A888666,https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxx,13800138000
苏E12345,https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=yyy,
浙B56789,https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=zzz,13900139000
```

> 原有的 `WECOM_WEBHOOK` 和 `PHONE_NUMBER` 可保留不删——带 `?plate=` 的请求走 `CAR_LIST`，不带 `?plate=` 的请求自动回退到旧变量。

### （可选项）

#### 绑定域名（建议，加快国内访问速度）

1. Worker →『Settings』→『Domains & Routes』
2. 点击『Add』→『Custom Domain』
3. 输入你的域名，按提示完成 DNS 配置

> 获得免费域名或购买域名等方法自行查找

#### 使用WAF规则防护（建议，防止境外恶意攻击）

1. 进入 Cloudflare Dashboard → 你的域名
2. 左侧菜单点击『Security』→ 『Security rules』
3. 点击『Create rule』→『Custom rules』
4. 规则设置：
   - Rule name: `Block non-CN traffic` / `阻止境外访问`
   - Field: `Country`
   - Operator: `does not equal`
   - Value: `China`
   - Choose action: `Block`
5. 点击『Deploy』

#### 修改 Worker 名称

1. Worker →『Settings』→『General』
2. 点击『Rename』，输入想更改的名称

## 生成挪车二维码（多车版）

每辆车生成不同的二维码，URL 格式为 `链接?plate=车牌号`：

```
https://movecar.你的账号.workers.dev?plate=沪A888666
https://movecar.你的账号.workers.dev?plate=苏E12345
```

1. 将对应车辆的完整 URL 复制到二维码生成工具（如 https://www.toolhelper.cn/QRCode/Generate）
2. 转换为二维码并下载
3. 分别印刷并张贴在对应车辆上
4. 可使用工具添加背景、文字等美化

## License

MIT
