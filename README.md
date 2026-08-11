# hayari-data

**`cloud-itonami/hayari` が観測した生データの永続層。** DataLad dataset で、実体は
git-annex 経由で `s3.kotobase.net` に置く。git に入るのはポインタとこの README だけ。

## なぜ永続させるのか（再取得できるのに）

Wikimedia の per-country pageviews は **2021-01-01 以降ならいつでも再取得できる**
（実測 2026-08-11: 2020-12-31 は 404、2021-01-01 は 200）。だから観測は原理的には
cache であって、失っても取り直せる。

それでもここに置くのは 3 つの理由による:

1. **上流は約束していない。** endpoint も保持期間も Wikimedia の運用判断で変わりうる。
   「取り直せる」は今日の観測であって、契約ではない
2. **観測した時点のバイトを残す。** 上流が後から値を訂正すると、再取得は
   *今の答え* を返す。分析の再現には *あの日そう見えた* が要る
3. **2021 より前は誰にも取れない。** その境界の外側は、失えば終わり

## 構造

```
raw/<YYYY-MM-DD>.datoms.edn   その日の生観測（annex）
corpus/hayari-content.edn     記事の冒頭抜粋（annex, CC BY-SA 4.0）
corpus/hayari-entities.edn    Wikidata entity（annex, CC0-1.0）
```

**ライセンスは混ざっている。** `corpus/hayari-content.edn` は Wikipedia 由来で
**CC BY-SA 4.0（帰属 + 継承）**、`corpus/hayari-entities.edn` と `raw/` の
Wikidata 由来部分は **CC0-1.0**。各レコードにライセンスが刻んであるので、
抜粋しても条件は失われない。

## custody

| remote | 置き場 | 独立性 |
|---|---|---|
| `kotobase` | `s3.kotobase.net` → bucket `cloud-itonami-hayari-data` | — |

⚠ **numcopies を独立性の主張に使わないこと。** ADR-2608100100 が実測したとおり、
`kotobase` remote の bytes は B2 の `s3.us-west-004` に落ちる。将来 `b2` remote を
足しても**同一プロバイダ**なので、2 コピーは bucket 削除や鍵 1 本の侵害には耐えるが、
**プロバイダ障害には耐えない**。

```bash
git annex whereis raw/2026-08-08.datoms.edn   # どこに何コピーあるか
git annex get raw/2026-08-08.datoms.edn        # 実体を取る
git annex drop raw/2026-08-08.datoms.edn       # ローカルから落とす（remote に残る）
```

## 正本はこちらではない

観測を作るのは `cloud-itonami/hayari`。ここは**その出力の保管庫**であって、
収集ロジックも判断も持たない。query 面に載るのは hayari 側の
`data/hayari-summary.edn`（要約、git 管理）で、ここの生データは載らない。
