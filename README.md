# loon-config

一份带详细注释、可直接读懂设计意图的 **Loon 配置**（iOS / iPadOS）。

主打：**国内直连、国外代理、不匹配也尽量直连**，附 AI 服务隔离、链式住宅 IP、分层去广告、流媒体/电竞优化。配置文件本身每个决策点都有中文注释，照着读就行；本 README 只讲整体设计与坑。

> ⚠️ **安全须知**：`loon.conf` 的 `[Proxy]` 段已**脱敏**，节点全是占位符。务必替换成你自己的，且**不要把真实密码 / UUID / 住宅代理账号提交到公开仓库**——GitHub 上有爬虫专扫代理凭证，几分钟内就会被盗用。

---

## 设计哲学

> 能走中国的全走直连，剩下的走代理。国内用得多，所以默认偏向直连。

落实成规则就是一条**解析型 GEOIP 兜底**：

```
... 显式规则 ...
DOMAIN-SUFFIX,cn,China
GEOIP,CN,China,no-resolve   # 便宜：IP 形态请求先拦
GEOIP,CN,China              # 兜底：未命中域名解析后按 IP 归属，中国 IP 一律直连
FINAL,Final                 # 走到这 = 确认是国外的 → Global 代理
```

`no-resolve` 那条只能拦 IP 形态请求；**第二条不带 `no-resolve`** 才是关键——未命中的域名会被解析、按 IP 归属判定，国内 IP 直接直连，只有解析出非中国 IP 的才落到 `FINAL → Global`。这样「国内全直连、剩下走代理」才真正成立，而不是国内冷门站漏到代理变慢。

代价：这条早于远程广告列表，**国内 IP 上的部分广告会少拦**——靠下面的分层去广告补回来。

## 去广告分三层（互不依赖路由）

1. **本地精准 REJECT 块**（`[Rule]` 最顶端）：穿山甲 / 优量汇 / 百度联盟 / 阿里妈妈等国内大广告联盟，只收「广告专用子域」，绝不碰内容主域。放最顶是为了压过 `qq.com/.cn→China` 和 GEOIP 兜底。
2. **远程广告列表**：blackmatrix7 `Advertising` / `Privacy`。
3. **MITM 重写插件**：fmz200 去广告合集（约 730 款 App，主力）+ app2smile 各 App + ddgksf2013 微博。**这层和路由无关**，直连也生效。

> 抖音去广告：本版**开启**（`[Mitm]` 未排除字节系主机，fmz200 可 MITM 抖音）。代价：抖音核心接口有证书锁定，MITM 偶发握手冲突，极端情况下抖音可能不稳/加载慢——这是「抖音去广告」的固有取舍。想要「抖音绝对稳但有广告」，把字节系主机加进 `[Mitm]` 的 `-` 强制排除即可。

## 其它要点

- **DNS**：阿里公共 DNS 打头 + DoH（阿里 / dnspod / 360），不优先吃运营商 DNS。
- **AI 隔离**：OpenAI/Claude/Gemini/Grok/DeepSeek 分别走不同出口；Claude/AI-API 经**链式住宅 IP**（设备→VPS→住宅代理→目标），对 AI 风控更友好。
- **QUIC**：YouTube 定点封 UDP 443 强制走 TCP，否则 MITM 去广告失效。
- **TikTok vs 抖音**：只收 TikTok 专用域名走代理；抖音那套 `snssdk/pstatp/bytedance` 走直连，注意 `isnssdk≠snssdk`、`ipstatp≠pstatp` 这类 i/sg 前缀变体，互不冲突。
- **WhatsApp**：已删 3 条超宽 AWS 网段（`/12`、`/15`），那是 WhatsApp 早年跑 AWS 的遗留，会误吞无关流量；现靠域名规则足够。

## 使用

1. 替换 `[Proxy]` 占位符为你自己的节点；订阅机场填 `[Remote Proxy]` 并把 `enabled` 改 `true`。
2. Loon 导入 `loon.conf`。
3. 去广告/增强需 **开启 MITM 并安装信任 CA 证书**；部分 App 要清缓存或重装才生效。
4. 想改抖音去广告/稳定性取舍，见上文「去广告分三层」。

## 致谢

规则与插件来自社区开源：[blackmatrix7](https://github.com/blackmatrix7/ios_rule_script)、[app2smile](https://github.com/app2smile/rules)、[fmz200/奶思](https://github.com/fmz200/wool_scripts)、[ddgksf2013](https://github.com/ddgksf2013/Rewrite)、[Maasea](https://github.com/Maasea/sgmodule)、[Hackl0us GeoIP2-CN](https://github.com/Hackl0us/GeoIP2-CN)、[Koolson/Qure 图标](https://github.com/Koolson/Qure)。

## 免责声明

仅供个人学习与网络优化研究。节点/订阅/破解类插件（如 Spotify 解锁）请自行评估合规与封号风险，与本仓库无关。
