# データ経路の運用

45m 鏡の観測データが、観測の後にどこに保管され、どう観測者とアーカイブに渡るかの、運用の規則と状態です。

- 置き場所と書く項目は、ハブの判断記録 [0013](https://github.com/nobeyama-radio-observatory/nro-context/blob/main/decisions/0013-観測系のインターフェース仕様をnro45-interfacesにまとめる.md) で決めました。
- データの形式（ファイルの中身）は書きません。形式の正本は [nro45-interfaces](https://github.com/nobeyama-radio-observatory/nro45-interfaces) です。
- 観測系の境界の全体は、nro-45m-control の [docs/interfaces.md](https://github.com/nobeyama-radio-observatory/nro-45m-control/blob/main/docs/interfaces.md)（⑤ 保管と配信）にあります。
- 接続先、計算機の上のパス、認証の方式、取得の手順は書きません。観測者向けの取得の方法の正本は、計算機グループの計算機とネットワークのシステムのページです（ハブの[非公開の一覧](https://github.com/nobeyama-radio-observatory/nro-context/blob/main/policy/non-public.md)）。

確認日：2026-11-19（同じ日の聞き取りを反映）

## 経路

いまは、SAM45 の経路と、YATSUGATAKE で SAM45 の故障した Array を肩代わりするデータ（YKSP）の暫定の経路の、二つがあります。

```
            SAM45 の経路                                YKSP の経路（暫定）
各装置の Local Controller（sw-cosmos-lc）         暫定の Local Controller（sw-sam45compat-lc）
        │ 観測時系列データ                                │ 観測時系列データ（YKSP）
        ▼                                                 ▼
マージ（sw-cosmos-merge）                          マージ（sw-cosmos-merge）
        │ 観測生データ                                    │ 観測生データ（YKSP）
        ▼                                                 ▼
NRODB（野辺山）（db-nrodb-nobeyama）               YKSP の保管と配信（db-yksp-delivery）
   │ 週1回の保守のとき                                    │
   ├──▶ NRODB（三鷹）（db-nrodb-mitaka）                  └──▶ 観測者（所内 LAN からだけ）
   ├──▶ 電波アーカイブ（db-radio-archive）：パイプラインを適用して登録
   └──▶ 観測者（Web とコマンドライン）
```

観測者から見ると、観測生データが SAM45 と YKSP の二つになります（YATSUGATAKE-NRO45-sam45compat の README）。

## 段階ごとの規則と状態

| 事柄 | SAM45 の経路 | YKSP の経路（暫定） |
|---|---|---|
| 保管するもの | マージした観測生データ（2013年度から）と、SAM45 の観測時系列データ（2015年度から）（`db-nrodb-nobeyama`） | 観測時系列データと観測生データ（YATSUGATAKE-NRO45-sam45compat の docs/plan.md の5） |
| 保管の単位 | 記録がない | グループとプロジェクトの単位。PSW と OTF を分ける（同 plan.md の5） |
| 二重化 | 野辺山と三鷹の二か所。週1回の保守のときに三鷹に送る（`db-nrodb-mitaka`） | 暫定の間はない。最終的には SAM45 と同じ経路に入る（[聞き取り 2026-11-19](../interviews/2026-11-19.md) 問1） |
| 保管の期間 | 野辺山の NRODB は数年。三鷹の NRODB と電波アーカイブは恒久（[聞き取り 2026-11-19](../interviews/2026-11-19.md) 問2） | 暫定の間は、1年を過ぎたものを消す案（同 plan.md の5-4）。最終的には左と同じ |
| アーカイブ | NRODB から取り出し、パイプラインを適用して電波アーカイブに登録する。人手がかかっており、完全な自動化が課題（`db-radio-archive`） | 最終的に登録する（[聞き取り 2026-11-19](../interviews/2026-11-19.md) 問1） |
| 観測者の取得 | Web とコマンドライン（`db-nrodb-nobeyama`） | 暫定の間は所内 LAN からだけ（同 plan.md の5-2）。最終的には、ほかのデータと同じ経路で取得する（[聞き取り 2026-11-19](../interviews/2026-11-19.md) 問3） |
| 取得できる人 | 占有期間中は、その観測者だけ（ハブの非公開の一覧） | 自分のグループのデータだけ（同 plan.md の5-2） |
| データの量 | 2021年12月の時点でおよそ23万件（`db-nrodb-nobeyama`）。1観測あたり数 GB、1年でおよそ 30 TB（[聞き取り 2026-11-19](../interviews/2026-11-19.md) 問4） | フルモードで、その 10〜20 倍程度の見込み。どのデータレートが主流になるかは予測できない（同 問4） |

## 決まっていないこと

2026-11-19 の聞き取り（[聞き取り 2026-11-19](../interviews/2026-11-19.md)）で、YKSP の最終的な経路は SAM45 と同じと決まった。残っているのは次のとおり。

1. **恒久の経路をどうするか。** NRODB を改修するか、新しいシステムを作るか。今年度中を目処に判断する。判断に合わせて、NRODB と電波アーカイブの保守の体制も見直す必要がある（問5）
2. **データの量の増え方。** YKSP のフルモードで 10〜20 倍の見込みだが、どのデータレートが主流になるかは予測できない（問4）。ストレージの費用の扱いは検討中
3. **解析ソフトの移行。** いまの解析ソフトにはデータ点数 4096 点の上限があり、YKSP のフルモードの観測生データは読めない。CASA に移る必要がある（問6）
4. SAM45 の経路の保管の単位（記録がない）

## 期限

SAM45 は一部の Array が故障しており、YATSUGATAKE への切り替えを早ければ2027年度末に終える見込みです（nro-45m-spectrometer の台帳の `be-yatsugatake` の `renewal`。仮説）。切り替えが済むと、観測データの大半が YKSP の経路を通ります。恒久の経路は、今年度中を目処に判断する予定です（聞き取り 2026-11-19 問5）。

YKSP のフルモードを提供する時期は、次の三つの期限を同時に決めます（仮説）。

- 観測時系列データと観測生データの形式の新しい版
- マージの作り直し（内製の予定。nro-45m-control の聞き取り 2026-11-19 問3）と、解析ソフトの CASA への移行
- データの量の増加に見合う保管の容量
