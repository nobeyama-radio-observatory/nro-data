# データ経路の運用

45m 鏡の観測データが、観測の後にどこに保管され、どう観測者とアーカイブに渡るかの、運用の規則と状態です。

- 置き場所と書く項目は、ハブの判断記録 [0013](https://github.com/nobeyama-radio-observatory/nro-context/blob/main/decisions/0013-観測系のインターフェース仕様をnro45-interfacesにまとめる.md) で決めました。
- データの形式（ファイルの中身）は書きません。形式の正本は [nro45-interfaces](https://github.com/nobeyama-radio-observatory/nro45-interfaces) です。
- 観測系の境界の全体は、nro-45m-control の [docs/interfaces.md](https://github.com/nobeyama-radio-observatory/nro-45m-control/blob/main/docs/interfaces.md)（⑤ 保管と配信）にあります。
- 接続先、計算機の上のパス、認証の方式、取得の手順は書きません。観測者向けの取得の方法の正本は、計算機グループの計算機とネットワークのシステムのページです（ハブの[非公開の一覧](https://github.com/nobeyama-radio-observatory/nro-context/blob/main/policy/non-public.md)）。

確認日：2026-11-19

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
| 二重化 | 野辺山と三鷹の二か所。週1回の保守のときに三鷹に送る（`db-nrodb-mitaka`） | 決まっていない |
| 保管の期間 | 記録がない | 1年を過ぎたものを消す案。自動か手か、観測時系列データも同じ期間かは決まっていない（同 plan.md の5-4） |
| アーカイブ | NRODB から取り出し、パイプラインを適用して電波アーカイブに登録する。人手がかかっており、完全な自動化が課題（`db-radio-archive`） | 登録するか決まっていない |
| 観測者の取得 | Web とコマンドライン（`db-nrodb-nobeyama`） | 所内 LAN からだけ（同 plan.md の5-2）。リモートの観測者の方法は決まっていない |
| 取得できる人 | 占有期間中は、その観測者だけ（ハブの非公開の一覧） | 自分のグループのデータだけ（同 plan.md の5-2） |
| データの量 | 2021年12月の時点でおよそ23万件（`db-nrodb-nobeyama`）。容量は記録がない | 記録がない |

## 決まっていないこと

計算機グループへの聞き取りの候補です。

1. YKSP の観測生データを、電波アーカイブに登録するか。三鷹に二重化するか
2. 保管の期間を過ぎた観測生データと観測時系列データを、どう扱うか
3. リモートの観測者が YKSP のデータを取得する方法（方針の粒度で）
4. データの量の見込み。1観測あたりと1年あたり。SAM45 と YKSP のそれぞれ
5. 恒久の経路をどうするか。NRODB を改修するか、新しいシステムを作るか。判断の時期。YATSUGATAKE が稼働すると、データの形式が変わり、量も急に増える（`db-nrodb-nobeyama` の `renewal`）
6. いまの解析ソフト（NEWSTAR、NOSTAR）が、YKSP の観測生データをそのまま読めるか
7. SAM45 の経路の保管の期間と保管の単位

## 期限

SAM45 は一部の Array が故障しており、YATSUGATAKE への切り替えを早ければ2027年度末に終える見込みです（nro-45m-spectrometer の台帳の `be-yatsugatake` の `renewal`。仮説）。切り替えが済むと、観測データの大半が YKSP の経路を通ります。恒久の経路は、それまでに決める必要があります（仮説）。
