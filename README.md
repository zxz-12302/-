# -人格 Persona
给 DeepSeek Harness 的助手注入一份稳定的「人格」：它只改变说话方式，不改变做事质量。

设置页：侧边栏 Settings →「人格」，直接改名字、语气、护栏开关、自定义文案，保存后下一条消息生效，不用重启。
工作优先护栏：写代码、调试、查资料、跑工具时人格自动收敛，信息密度优先。
开箱即用：装好即有默认人格「小深」；想定制就去设置页改，或改 %DSH_HOME%\persona.json。
安装
在 Harness 里装一次即可，之后每次启动自动加载：

dsh plugin --profile <profile 名> add dsh-persona-plugin
或者图形界面：设置 → 插件 → 从 npm 安装 dsh-persona-plugin。

装完刷新一次页面（Ctrl+R），设置面板左栏最底部会出现「人格」。

设置页
打开侧边栏 Settings，左栏最下面的「人格」（英文界面为 Persona）。字段：

字段	默认值	说明
启用人格	开	关掉后完全不注入人格文案
工作优先护栏	开	关掉后只剩语气，不再声明「工作优先」优先级（不建议关）
人格名	小深	出现在 ## 人格 · <名字>；60 字符上限
语气	沉稳、直接、可靠，偶尔冷幽默	一句话描述；200 字符上限
自定义人格文案	空	非空时整段替换内置文案，{name} 会替换成当前人格名；20000 字符上限
页面底部会实时显示「当前注入的人格提示词」，可以直接看到模型收到的那段文本。 「恢复默认」会删除设置文件，回到内置默认人格。

设置保存在 %DSH_HOME%\persona.json（默认 C:\Users\<你>\.dsh\persona.json），一个普通 JSON 文件，可手改、可备份、可删除。

行为
插件向系统提示词注册一个段落（local:persona，默认 order 1，位于部署人格之后、第一方指引之前），内容为：

## 人格 · <名字> — 语气、称呼、表达、情绪、语言、禁忌。
## 工作优先 — 任务、正确性、安全 > 人格表达的硬性优先级。
段落以 interpolate: false 注册，文本里的 {...} 不会被当作提示词变量，因此 custom 可以安全地包含代码片段。 段落 text 是一个函数，每次组装提示词时读取当前状态，所以保存后无需重新注册。

优先级与来源
生效值 = 内置默认 → bundle 行 config（cordis.patch.yml）→ %DSH_HOME%\persona.json（设置页写的）。
order 只从行 config 读取，设置页不提供（改排序请编辑 patch）。
作用范围
段落注册在全局提示词注册表上，因此对所有 Session 生效（包括 Agent Teams 的队友）。工作护栏保证队友的任务执行不受人格影响；如果只想对用户会话生效，可再按 agent.ctx 做 scope 化，属于后续可选增强。

自测
$node = "C:\Users\1\.dsh\dsh-runtimes\dsh-primary-runtime\dependencies\node\bin\node.exe"
& $node _tmp/persona-host-test.mjs      # 渲染、校验、保存/恢复、热更新
& $node _tmp/persona-client-test.mjs    # Client 半部分注册形状
Host 自测用临时 DSH_HOME，覆盖：默认渲染、护栏开关、{name} 替换、enabled:false 不注入、非法配置拒绝、保存落盘、重载后读回、恢复默认。
