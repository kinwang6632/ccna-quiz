# 題庫答案檢查問題清單

檢查日期：2026-09-26　範圍：第 1–1100 題（[ccna-quiz.html](ccna-quiz.html)）

說明：

- 已在解析 `ex` 中註明「題庫給的答案是 X，改採 Y」的題目不列入本清單。
- 本清單僅為檢查結果，題庫內容尚未修改。
- 狀態欄：`[ ]` 待處理、`[x]` 已修正。

## 一、高度可疑，建議修正

| 狀態 | 題號 | 目前答案 | 建議答案 | 理由 |
|---|---|---|---|---|
| [ ] | 157 | C | D | 與第 21 題完全相同（第 21 題答案為「超過最大功率 → err-disabled」），兩題互相矛盾；依 Cisco PoE 監控行為應為 err-disabled。 |
| [ ] | 158 | C | A | 與第 821 題相同（821 答 A：predictable latency）。「加一層 leaf 解決超額訂閱」不是 spine-leaf 的正確描述。 |
| [ ] | 296 | AD | AE | 與第 229 題相同（229 答 AE）。Root ID 區塊中的 `Port 1 (FastEthernet 2/1)` 是 root port，不是 designated port。 |
| [ ] | 488 | BC | AB | 附圖：PC2～PC4 位於 R2 後方、PC5（.10）位於 R3 後方。需要 `10.10.10.0/24 → 192.168.2.3`（A）加上 `10.10.10.10/32 → 192.168.2.2`（B）；BC 兩條都只是 10.10.10.10 的主機路由，PC2～PC4 無法到達。 |
| [ ] | 490 | CE | BC | E 與 C 同樣走 R2，無法讓往 `2001:db8:23::14` 的流量改走 R3。應為 /64 走 R2（C）＋ ::14/128 走 R3（B）。 |
| [ ] | 385 | ABF | ABE | OSPF 分層設計的理由是加快收斂、降低路由負擔、將不穩定侷限在單一區域（E）；F「降低路由器設定複雜度」不成立。 |
| [ ] | 568 | D | C | D 使用不存在的 `ntp stratum 2` 指令，且在標準 ACL 10 使用延伸 ACL 語法；C 使用 `ntp master 2` 與標準 ACL，語法正確。 |
| [ ] | 197 | D | B | Cisco IP Phone 收到 PC 的未標記資料流量會原樣轉送，由交換器放入 access VLAN，電話不會替它加標籤。 |
| [ ] | 272 | 拖放 | 修正配對 | Autonomous AP 被配到「configured and managed by a WLC」，錯誤；應為「requires a management IP address」（可對照第 278 題）。 |
| [ ] | 207 | 拖放 | 修正配對 | 兩格對調：「Controller image auto deployed to access points」→ Easy upgrade process；「Controller provides centralized management of users and VLANs」→ Easy Deployment Process。 |
| [ ] | 261 | 拖放 | 修正配對 | Forwarding 狀態的答案中含「Frames received from the attached segment are discarded」，此為 blocking 狀態行為；應改為「…are processed」或「Switched frames received from other ports are advanced」。 |

## 二、有爭議，建議查證

| 狀態 | 題號 | 目前答案 | 可能答案 | 說明 |
|---|---|---|---|---|
| [ ] | 175 | 拖放 | — | 「requires IP addresses on the access point and the WLC」被放在 Layer 2 Tunnel，較像 Layer 3 LWAPP 的特性。 |
| [ ] | 159 | C | B | B（轉送到下一跳）與 C（查詢 FIB）都屬 data plane，屬雙解題；常見題庫答案為 B。 |
| [ ] | 234 | CE | DE | 手動 shutdown 的介面顯示為 administratively down，而非 down/down；err-disabled（D）才是 down/down。 |
| [ ] | 642 | D | C | 題目要求「增加違規計數並送出 SNMP trap」，一般答案為 restrict；shutdown 雖也會計數與送 trap，但會關閉埠。 |
| [ ] | 1057 | C | B | 在語音 VLAN 上手動指定 MAC 的語法為 `switchport port-security mac-address <mac> vlan voice`。 |

## 三、資料瑕疵（不影響答案）

| 狀態 | 題號 | 問題 |
|---|---|---|
| [ ] | 156 | 選項 A 與 D 內容完全相同（對照附圖計算，答案 C 正確）。 |
| [ ] | 137 | 「ff05:1:3」少一個冒號，應為「ff05::1:3」。 |
| [ ] | 957 | B、D 實際為同一設定（bootps = UDP 67），單選題有兩個正確選項；解析已說明。 |

## 附錄：已抽查附圖、確認答案正確的題目

75、90、123、156、184、186、213、269、471、621
