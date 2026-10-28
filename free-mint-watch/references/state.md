# 配置与去重状态

按任务保存配置，不将个人地址写死在可复用技能。配置字段：walletAddress、preferredChains、intervalMinutes、useAgenticWallet（监测固定false）、walletForInteractiveMint（可为true）、autoMint（默认false）、knownMinted、ignored、publicRpc。knownMinted 存链、合约、轮次和Token ID，不屏蔽有证据的新轮次。

walletAddress 为空时只做项目级检查，禁止在检查阶段连接钱包或读取钱包地址。RPC 仅发送核查所必需的公开地址与方法，不携带凭证。

初始状态：

```json
{"version":1,"lastRunAt":null,"candidates":{},"notified":{},"sourceFailures":{},"walletSessionNoticeSent":false}
```

candidates[key] 字段：name、chainId、contract、round、officialUrl、discoverySources、verificationSources、checkedAt、blockNumber、status（upcoming/open/closed/soldout/unverified）、eligibility（eligible/ineligible/unknown）、eligibilityReason、mintPriceWei、gasEstimateWei、otherFees、depositAndRefund、startAt、endAt、transferable、marketStatus、liquidityEvidence、riskNotes。未知值null，不用0；wei用十进制字符串精确存储。

notified[key] 存 lastNotifiedAt、reason、status、eligibility、round、关键费用、endAt、deadlineReminderSent，比较最近已通知快照去重。已领同轮次或免费名额用完仍保留记录，不重复推荐。失败时保留已有记录并标 stale，不伪装成新鲜数据。

## 推送名单分组

- 构建通知前，依据本轮写入前的 notified 主键判断分组：从未通知的主键为“新增项目”，已通知且本轮仍符合推荐条件的为“已有项目”。不要以首次发现时间或上次轮询是否出现来判定新增。
- 每次实际推送都带上当前完整推荐名单，包括没有变化的已有项目。首次推送全部为新增；已通知项目重新开放仍为已有，独立新轮次的新主键为新增。
- 已移除项目保留 notified 历史，避免重新出现时误标新增；移除原因和 stale 待复核项目单列，不混入当前推荐名单。
- 完成分组后再更新本次实际通知所含项目的快照；已有项目重列不重置 deadlineReminderSent，也不改写为新增。通知触发仍与最近通知快照比较，重列名单本身不触发通知。

状态不可存任何钱包认证材料。执行一次核查不等于通知已送达；仅在本轮确定发出用户通知时更新 notified。旧监测状态文件与本任务分开保存。

## 时间参数

schedule 字段如下，intervalMinutes 保留为旧配置兼容字段，与 schedule.intervalMinutes 同步。

```json
{"schedule":{"mode":"interval","intervalMinutes":10,"times":[],"timezone":"Asia/Shanghai","activeWindow":null},"notifyOnlyOnChange":true}
```

- mode=interval：intervalMinutes 为正整数；mode=daily：times 为24小时 HH:MM 数组。两种模式只能选一种。
- activeWindow=null 表示全天，否则为 {"start":"09:00","end":"23:00"}，起点包含、终点不包含，start>end 跨午夜；start=end 应明确确认含义，不自行当成全天。
- IANA timezone；先按该时区判断时段，再搜索，时段外直接静默。修改时段应保留最近通知快照。
- 定时能力须真实配置到宿主。宿主不支持精确频率时说明限制，不悄悄降频。时间窗口可由调度器表达，也可每轮入口先过滤。
- 计算通知差异时排除 checkedAt、blockNumber、供应量、普通 Gas、汇率、展示顺序等噪声，只比较用户可操作的语义字段。窗口到期状态变化可通知；仅“又过10分钟”不是新信息。

## 推送附带登录询问

askWalletLoginOnNotify=true 表示每次实际发出的变化通知都附带“要不要登录 Agentic Wallet？”。不因问题相同或登录过期而额外推送。问题写作“要不要登录 Agentic Wallet 进行铸造？”。仅在用户明确同意后进入独立的交互钱包流程，已连接不重新扫码。此选项不改变 autoMint=false，也不授权任何交易。
