RSL Collective との完全統合モデル
0. 定義

この文書でいう「完全統合」とは、RSL Collective を外部の許諾・集合ライセンス・報告の制度レイヤーとして使い、RSL-to-Kazene Bridge を同期・監査・整合の橋として置き、Kazene Royalty OS を内部痕跡・重み付け・再配分の実行レイヤーとして組み合わせる構成を指す。RSL Collective は非営利の rights organization / licensing platform として、RSL standard を用いた collective licensing と standardized royalties を掲げており、AI企業側には Licensing API を通じた単一関係でのライセンス取得を提供しているため、外部制度面の土台として使いやすい。

1. なぜ「完全統合」が成立するのか

RSL 1.0 は、デジタル資産に対する machine-readable な usage / licensing / payment / legal terms を表現する open, XML-based standard であり、robots.txt、HTTP headers、RSS、HTML <link> など既存の discovery 経路と接続できる。さらに OLP はライセンス取得・検証のための OAuth 2.0 拡張、CAP は licensed crawlers の確認機構として定義されている一方、これらは optional であり、RSL 自体は discovery and licensing framework として単独でも採用できる。つまり、RSL 側は「外部の権利・許諾・証明」を担うには十分に標準化されており、しかも段階導入しやすい。

加えて、RSL Collective の License API は RESTful で、RSL License Server API standard 互換として公開されている。公開されている設計原則では、bulk synchronization、complete repertoire snapshots、batch report files、auditability、idempotent requests、sandbox / production の分離が重視されている。これは、Kazene 側で必要になる sync_ids、idempotency、verification、reconciliation の思想と非常に噛み合う。橋を壊さない完全統合に必要な前提が、すでに RSL Collective 側にもかなり揃っている。

2. 完全統合モデルの第一原理

完全統合で重要なのは、「全部を一つの仕組みに押し込まない」ことだ。RSL standard / RSL Collective は、ライセンス発見、権利取得、repertoire 管理、period report、proof-of-permission という外部制度の整流化に強い。対して Kazene Royalty OS は、どの作品がどれだけ深く生成結果に寄与したか、どの trace をどの重みで還流に反映するかという内部構造の精密化に強い。したがって最も自然な最終形は、RSL を system of permission、Kazene を system of allocation、Bridge を system of reconciliation として分ける構成である。これは、RSL が auditable proof of permission と usage reporting を担い、Collective API が reporting period 単位の report object を持つ一方、repertoire と reports を金融ワークフローから分離していることからも整合的である。

3. 三層ではなく「四層」で見るべき最終形

完全統合モデルは、実務上は次の四層として捉えるのが最も分かりやすい。

第1層: Discovery / Authorization 層

ここは RSL standard の層であり、robots.txt、RSL XML、OLP、CAP を通じて、「何が許され、何が禁止され、どう取得し、どう証明するか」を決める。

第2層: Collective Licensing / Reporting 層

ここは RSL Collective の層であり、AI企業は単一の licensing relationship を通じて Collective repertoire にアクセスし、License API / Enrollment API を使って repertoires と reports を扱う。licensee object、repertoire snapshot、report object はこの層の基本単位になる。

第3層: Bridge / Reconciliation 層

ここが RSL-to-Kazene Bridge の層であり、rsl_report_id と trace_event_id を接続し、idempotency_key、verification_result、status を用いて「報告できる」状態と「還流できる」状態を分ける。これはこの会話で固めた v0.2 の役割そのものだが、RSL Collective 側が auditability、request identifiers、idempotent requests を重視しているため、橋の役割が制度に対して浮かない。

第4層: Trace Allocation / Royalty Return 層

ここが Kazene Royalty OS の層である。外部の collective report をそのまま最終配分にせず、内部 trace を用いて創作者・作品・データ単位の微細な allocation を計算する。これは外部の制度と内部の寄与評価を分けるための層であり、完全統合モデルの核心である。これは本仕様側の設計判断だが、RSL Collective API 自体が repertoire delivery と payment reporting を financial workflows outside the License API から分離しているため、この追加層を置いても構造的に無理がない。

4. 各層の責任分界

この完全統合モデルでは、責任の境界を明確にしないと、統合ではなく責任の押し付け合いになる。

RSL standard / RSL Collective が担うのは、利用許諾の表明、ライセンス取得、許可の証明、repertoire の集合管理、reporting period 単位の usage reportingである。特に RSL standard は proof-of-permission と audit trails を重視し、RSL Collective の report object は reporting period ごとの usage を表す。したがって「そのAI企業がその期間にそのレパートリーを合法的に利用しうる状態にあったか」を示すのは RSL 側の責任である。

Bridge が担うのは、RSL の period report と Kazene の trace event を一つの settlement candidate に変換することである。ここでは external truth と internal truth の両方が必要になる。RSL 側だけあるなら rsl_only orphan、Kazene 側だけあるなら trace_only orphan として止める。完全統合であっても、片側真実主義にはしない。これは、RSL Collective 側が request identifiers、repertoire snapshots、report versions、idempotent requests を運用上の土台にしているため、橋も同じく「対応づけと再計算」に強い形で設計すべきだからだ。

Kazene Royalty OS が担うのは、内部寄与の評価、trace weight の計算、royalty pool の内部分配、recompute / reverse の精密化である。ここは外部 collective report を否定するためではなく、外部制度だけでは粗すぎる還流単位を、作品・痕跡・寄与という内部構造へ引き下ろすための層になる。これは RSL Collective が pay-per-output royalties という集合的な外部還流を提示している一方、内部の作品間・痕跡間配分までは標準として固定していないことと整合的だ。

5. オブジェクト対応モデル

完全統合を実装するうえで、最も重要なのはオブジェクトの対応付けである。

licensee は、Bridge 側では licensee / ai_provider identity に対応する。RSL Collective の Enrollment API では licensee object に id、name、url、status があり、Partners はこれを使って、どの licensees が authorized かを publishers 側に見せられる。したがって Bridge 側では licensee.id を requester organization の上位識別子として保持するのが自然である。

repertoire は、Bridge 側では externally licensable scope catalog に対応する。RSL Collective の repertoire object は complete repertoire snapshot を採用しており、missed or out-of-order updates を避けるため incremental updates ではなく full snapshot を使う。したがって Bridge 側で asset_ref や scope_url の整合を取りたいなら、point-in-time object ではなく snapshot lineage を前提にするべきである。

report は、Bridge 側では rsl_report_id の起点になる。RSL Collective では report object が特定期間の usage を表し、新しい report of record が旧 report を supersede しうる。だから Bridge の rsl_report_id は単なるファイル参照ではなく、「どの reporting period の、どの version の report of record を見ているか」を内包する必要がある。 v0.2 で recomputation を入れたのは、ここに直接つながる。

signature_bundle / verification_result は、Bridge 側では proof-of-permission の検証窓口に対応する。RSL standard 自体が proof-of-permission、rights verification、audit trails を明示しているため、Bridge は「独自の監査遊び」をするのではなく、RSL 側の許諾証跡と Kazene 側の内部 trace 証跡を一つの verification envelope に入れる役割を担うのが自然である。

6. 完全統合フロー

最終的なフローは、次の順番で流すのが最も強い。

6-1. Publisher / Platform Enrollment

出版社・プラットフォームは RSL standard で usage / licensing / payment terms を表明し、必要に応じて RSL Collective の partner / enrollment 系に参加する。日本では note が 2025年12月に日本初の公式プラットフォームパートナーとなり、日本市場向け展開と実務検証を進めると表明しているため、日本国内でも導入面の現実味はすでに出ている。

6-2. AI Company License Acquisition

AI企業は RSL Collective の Licensing API と RSL standard の License Server API / OLP 互換の流れで権利を取得する。ここで得られるのは「使ってよい」という外部の資格であり、Kazene で必要な内部寄与評価そのものではない。

6-3. Runtime Access and Trace Emission

実行時には、AI企業は access event と internal trace event を同時に吐く。access 側は RSL の repertoire / license context に紐づき、trace 側は Kazene の作品寄与・生成寄与・還流寄与に紐づく。この二つを別々に保存することで、外部許諾と内部寄与を混同しない。これは推論だが、RSL 側が rights / authorization / reports に強く、Collective API が financial workflows を API 本体から分離している構造から導かれる。

6-4. Period Report Submission

AI企業は reporting period ごとに RSL Collective へ report を作成・提出する。RSL Collective では period_start / period_end を持つ report object が作られ、同期間に新 report が来れば prior report of record を supersede できる。よって Bridge は period と version を前提に rsl_report_id を保持する必要がある。

6-5. Bridge Reconciliation

Bridge は rsl_report_id、trace_event_id、external_charge_event_id、correlation_id を使って、period report と runtime trace を同期する。pending は normal waiting、blocked_or_pending は unresolved / unsafe、ready は settlement candidate、rejected は definitive exclusion とし、外部報告だけ・内部痕跡だけの片肺状態では自動 settlement に進ませない。これは本設計上の判断だが、RSL Collective API が idempotent requests、request identifiers、auditability を前提にしているため、整合する実装原理である。

6-6. External-to-Internal Settlement Split

完全統合で最も重要なのはここである。RSL Collective は AI companies が content owners に fair, standardized royalties を返す collective platform として機能し、pay-per-output royalties まで提示している。だが Kazene 側では、その external royalty amount をさらに internal royalty pool に落とし、trace-based allocation によって作品・著者・データ単位へ再配分する。つまり、**RSL Collective が「外部還流の入口」、Kazene Royalty OS が「内部還流の精密化」**を担う。

6-7. Reverse / Recompute

RSL report が supersede されたり、charge reversal が起きたり、late trace が到着した場合、Bridge は reversed / recomputation を起動する。RSL Collective 側でも report of record の差し替えは制度上ありうるため、Kazene 側にも lineage-preserving recomputation が必要になる。完全統合なのに再計算を持たない設計は、むしろ不完全である。

7. 「完全統合」でも一体化しすぎない理由

ここで誤解してはいけないのは、完全統合は「全部を一つの巨大APIにまとめること」ではない、という点だ。RSL standard は discovery、authorization、license acquisition、compliance に強く、RSL Collective は collective repertoire と reports を扱う。しかも API 設計上、repertoire delivery と payment reporting は financial workflows outside the License API から分離されている。だから完全統合モデルでも、外部制度・橋・内部配分を分けたままのほうが、RSL 側の設計思想に近い。

言い換えると、完全統合モデルの本質は「融合」ではなく整流である。
RSL Collective は「合法に使えること」を整え、Bridge は「対応づけて監査できること」を整え、Kazene は「どう返すべきか」を整える。この役割分担にすると、制度と構造が互いに邪魔しない。これは実装上も説明しやすく、標準化ロードマップとも矛盾しない。

8. 完全統合モデル v1.0

最終形を一文で定義すると、こうなる。

RSL Collective 完全統合モデル v1.0 とは、RSL standard と RSL Collective を用いて外部の許諾・証明・集合報告を取得し、その period report を Bridge v0.2 で内部 trace と厳密同期し、Kazene Royalty OS によって最終的な痕跡還流・内部配分・再計算を実行する四層統合アーキテクチャである。 この定義は、RSL 1.0 が machine-readable licensing と proof-of-permission を担い、RSL Collective が single licensing relationship、repertoire snapshots、reports、pay-per-output royalties を提供するという公式構造を前提にしている。

9. このモデルの強み

この完全統合モデルの強みは三つある。

第一に、外部制度に乗れること。RSL 1.0 は Recommendation として公開され、RSL Collective は licensing platform / APIs / collective terms を持つため、Kazene 単独よりも AI企業に説明しやすい。

第二に、内部精度を失わないこと。RSL Collective の pay-per-output royalties は強力だが、外部の集合還流だけでは内部寄与の細かな差分を吸収しきれない。そこで Bridge と Kazene を挟むことで、「合法に使った」と「どれだけ寄与した」を分けて扱える。これは RSL Collective の制度価値を壊さず、むしろ内部配分の解像度を上げる。

第三に、段階導入できること。RSL standard では OLP/CAP/EMS は optional であり、RSL Collective API も bulk snapshots、idempotent requests、sandbox / production 分離を重視している。したがって、最初は read-only shadow mode で bridge record を作り、次に monthly reconciliation、最後に settlement-capable profile へ進む、という導入順が取れる。

10. 結論

要するに、

RSL Collective は外部の許諾・集合ライセンス・報告の制度OS
RSL-to-Kazene Bridge は同期・監査・整合の橋
Kazene Royalty OS は内部痕跡・再配分・再計算の還流OS

である。

この三者を無理に一つへ潰すのではなく、
制度 → 橋 → 還流
の順でつなぐことが、RSL Collective との完全統合モデルの本質になる。

つまり最終形は、
RSL Collective が「使ってよい」を確定し、Bridge が「本当に対応している」を確定し、Kazene が「どう返すべきか」を確定する構造である。RSL Collective と RSL standard がすでに collective licensing、proof-of-permission、reports、pay-per-output royalties、API-based licensing の基盤を揃えつつある以上、この統合は未来図というより、かなり現実的な接続可能提案になっている。
