# 中文去AI味

给 [DeepSeek Harness](https://github.com/deepseek-ai) 用的一个插件。装上以后多一个专门写中文的 agent，默认的编程 agent 遇到要写给人看的正文，会把活交给它。

名字叫「中文去AI味」，是想说它针对的不是"AI 写得好不好"，是"AI 写出来一眼就能认出来"。

## 它治什么

编程能力强的模型写中文，有几样毛病，靠一句"请写得自然一点"治不好：

**句式上的模板**

- 翻案腔：先立一个读者没有的误解再推翻。「不是 A 而是 B」「看似……实则……」「你以为……其实……」
- 抬价：「不仅……而且……」「不只是……更是……」、三个形容词并排、一个分句三个顿号
- 情绪壳：「带着一丝……」「眼中闪过一丝……」「心中涌起一股……」
- 解释腔：「之所以……是因为……」「这意味着」「值得注意的是」
- 文档腔：「以下是……」「首先……其次……最后……」、正文夹「注：」「示例：」、开头预告全文
- 假深刻收尾：「总之」「综上」「未来可期」

**翻译腔**

长定语、「当……的时候」、「对于……而言」、被动句、动词名词化（"进行了优化"）。

**语气上的毛病**

- 太肯定：把话说满，「本质上」「从来」「所有人」，不给读者留判断的余地
- 太硬：形容不好的地方直接给「矮」「短」「差」「穷」，不知道中文说「不高」「不长」「不太好」
- 太干：删光虚词和废话，一句闲话都不留，每句都要给个结果
- 太碎：句号密，逗号少，八到十二个字一句，读起来像清单

**结构和载体**

- 每段「观点—解释—小结」，每节都收一次尾
- 提示词本身写成编号手册，模型照着手册的节奏写，越治越像

## 怎么装

### 交给 AI 装（推荐）

把下面这一段整个复制给你的 DSH agent：

```text
请帮我安装 DSH 插件「中文去AI味」。

仓库：https://github.com/moxiaolong/chinese-deai

步骤：
1. 调用 plugin_manager，action=install_bundle，target 填上面的仓库地址（也可以写成 github:moxiaolong/chinese-deai）。
2. 装完调用 plugin_manager，action=list_plugins，确认 preset-deai 和 preset-standard 两行的 fiberPhase 都是 active。
3. 有 diagnostic 就原样贴给我，先别继续。
4. 装好后提醒我新开一个会话——旧会话在启动时就锁定了插件版本，拿不到新东西。

如果默认 agent 起不来（新建会话失败或没有工具），回滚：
plugin_manager，action=remove_bundle，target=dsh-plugin-chinese-deai
```

### 手动装

```bash
dsh plugin --profile desktop add github:moxiaolong/chinese-deai
```

`desktop` 换成你的 profile 名：桌面应用是 `desktop`，网页版是 `web`，命令行是 `tui`。装完重启应用。

## 装完是什么样

- 新建会话时，agent 列表里多一个「文学专家」
- 默认 agent 的 persona 多一句：要写给人看的正文（文章、文案、小说、长回复、邮件正文）时，调用 `deai_writer` 交给文学专家；聊天回复、代码注释不算
- 写代码、改文件、跑命令，还是默认 agent，不受影响

## 它改了什么

插件是三个东西：

1. **新增预设 `deai`**（显示名「文学专家」）。persona 换成写作身份，工具只给读写文件、检索、提问、交付文件，不挂 shell。
2. **新增工具 `deai_writer`**。子 agent 默认用 `deai` 预设，白名单也只有它一个。
3. **整份覆盖 `preset-standard`**。在原样照抄的基础上改两处：persona 加一句路由规则，delegation 组里多挂 `deai_writer`。

第 3 条是它能在默认 agent 里自动生效的原因，也是它唯一的代价。

## 注意

**它跟 DSH 的版本绑着。** `preset-standard` 的内容是照着 DSH 0.2.x 自带的预设抄的。DSH 升级后如果默认 agent 行为异常、或者少了工具，把 `cordis.patch.yml` 末尾那段 `- id: preset-standard` 整段删掉，重新装一次。删了以后文学专家照常可用，只是要手动选，不会自动交接。

**它不会自动分流。** 是不是交给文学专家，由默认 agent 自己判断。判断依据就是 persona 里那一句，你可以按自己的习惯改。

**提示词是可以改的。** 正文在 `cordis.patch.yml` 里 `prefix: |-` 下面那一大段，是散文体写的，不是清单。改的时候保持这个文体——写成编号手册，模型会照着学，等于换个方式的 AI 味。

## 目录

```
chinese-deai/
├── package.json        插件清单，dsh.bundle.patch 指向下面的 patch
├── cordis.patch.yml    全部实现：预设、工具、对 standard 的覆盖
├── PROMPT.md           提示词的可读版本（以 patch 里为准）
└── README.md
```

## 许可

MIT
