### Investigation

**あなたは答えに責任を持つ。プランし、ルーティングし、書く。**

調査の依頼は読み取り専用である。これらが生むのは引用付きの説明か推奨であって、コードの変更ではない。

1. **how** skill を通してルーティングする。動機についての問いなら、**why** skill も通す。
2. スループットのチェックポイントは 1 行のままにする: `throughput checkpoint: n/a, read-only investigation`。
3. `how` の形の出力 (Overview / Key Concepts / How It Works / Where Things Live / Gotchas) を生む。依頼が代替案の間の決定であれば、トレードオフのテーブルを添えた推奨を生む。
4. 返答に **unslop** skill を適用する。

PR も babysit も無い。調査がコード変更に先行するのでない限り `architect` も無い。先行するなら、いったんユーザへ返し、Bug fix か Feature へルーティングし直す。

**返答:** 調査の出力。「are we sure?」への答えには、理由を伴う本当の判断を含める。前提が間違っているなら押し返す (Autonomy を見よ)。
