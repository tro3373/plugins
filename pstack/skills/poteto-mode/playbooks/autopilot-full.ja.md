### Autopilot-full

**あなたが持つのは verdict であって、PR ではない。各 PR は 1 人の owner が build から merge まで担い、コードを書いていないあなたの swarm が、着地するコードを証明するまで何も merge されない。**

verdict が覆うのは 1 つのパッチ、green な CI が覆うのは 1 つの head だ。ステップ 4 は full swarm を 1 回走らせ、大きな変更のときだけもう 1 回走らせる。`playbooks/shipping.md` の patch-id ルールがレーンの結果を持ち越せることがあり、新しい head にはそれぞれ自前の green な CI が要る。

1. **operator のゲートが先だ。** 付与が覆うのはキューの項目だけなので、operator が自分のものと名指しした項目と、キュー外の PR は merge-ready で止まる。プランを述べるよう求められても、それは go ではない。
2. **PR ごとに owner を 1 体起動し、その作業を代替が見つけられる場所に置く。** forge はプログラムにつき一度だけ解決する。GitHub CLI (`gh`) が既定だ。`command -v origin` が成功し、Origin がそのリポジトリを解決できるなら、PR の操作はすべて `origin pr` を使う。そうでなければ `gh` のままとし、フォールバックしたことを記録する。Graphite (`gt`) を必須にしてはならない。各 owner は Cursor cloud agent で、build し、`playbooks/opening-a-pr.md` に従って PR を開き、実物のアーティファクト上で変更を証明し、`playbooks/babysit.md` に従って green まで babysit し、ステップ 4 と 5 が許可したら merge する。

   CI と bot が作業と並行して走るよう、PR は早めに開く。コードが ready になるのは、slop-strip (`cursor-team-kit` プラグインの `deslop` skill (`/deslop`)) と `/no-comments` のパスを終えた後だけで、これにより full swarm は仕上がったコードに対して走る。その後、現在の trunk へ rebase し、self-proof の前に code-ready な head を報告する。検証が、パッチの着地先のベース上で走るようにするためだ。以降の head はすべて報告し、self-proof、CI、babysit が終わったら merge-ready を報告する。fix のラウンドは、trunk が競合するか CI を壊さない限りそのベースに留まる。rebase のたびに CI が再開するからだ。対象パスに対するリポジトリのチェックは、コミット済みの head に対して、code-ready の push の前と、verifier を要する後続の push の前に走らせる。

   VM は落ちるものと想定する。開始後まもなく、**show-me-your-work** skill に従って root エージェントの store (パスはシステムプロンプトにある) に決定の trail を push して開始し、以降は残す価値のある単位ごとに push する。サブエージェントを spawn した者は、その id、状態、その種類の過去の run から見積もった stuck-after の実行時間をその store に記録する。親が死んでも root が探れるようにするためだ。force-push するのは自分のブランチだけとし、`--force-with-lease` は最後に push した、または引き継いだ SHA に固定する。自分が行っていない push があったら、止まって root に報告する。
3. **owner は trunk から並列に走らせ、決してスタックしない。** そうすれば各 PR が単独で着地する。
4. **swarm 検証は 1 回だけ。** full swarm は code-ready な head で 1 回走らせる。もう 1 回 full swarm を走らせるのは、新しい挙動、再設計、大きな新しい diff のときだけだ。rebase、merge conflict の解消、軽微な fix は、新しい head に対して verifier を 1 つ立てる。**swarm** skill に沿って、code-ready な SHA で独立した verifier を展開する。彼らはゲートを再実行し、荷重を支える挙動を実際の面の上で、その control skill (`cursor-team-kit` の `control-ui` や `control-cli` など) か名指しされたドライバを通じてライブで証明し、diff を監査する。ライブのレーンが無い verdict、または merge-ready レポートのレシートを監査する前の verdict は clean ではない。どのレーンも、確認していない主張を信用しないからだ。

   証明されたすべての finding は、レーンがどうラベル付けしていても、同じ欠陥を持つすべての箇所にまたがる red test とともに、1 回の fix-forward で owner に送る。PR が無くても trunk に現れる欠陥は trunk に対して記録し、PR のゲートにはしない。ただし PR が複製した、または新たに到達した欠陥は PR 自身のものだ。fix を自分で push してはならない。ブランチの writer は 1 人だからだ。stuck した owner は置き換える。
5. **owner が merge するのは、あなたが検証し、着地先の trunk でテストされたものだけだ。** green な CI だけでは merge は認可されない。swarm のレーンが始まった後、merge 準備は現在の trunk へ rebase し、新しい head を報告し、green な CI を待つ。

   merge の直前に trunk を fetch し、その変更がパッチに影響しうるかを問う。trunk が動くたびに rebase していては終わらないからだ。着地するコードが一度もテストされていないとき、たとえば競合、双方が変更したファイル、その PR でどのチェックが走るかを変える変更があるときは、rebase して CI を待つ。双方が変更した lockfile は、merge 結果で frozen install が通り、PR が変更するパッケージが head と同じに解決されるなら rebase は要らない。その install が着地する lockfile をテストするからであり、install が失敗するなら rebase で直さねばならない。trunk がパッチの呼び出し元、またはパッチが依存するもの (呼び出すコードや依存のバージョン) を変えたときは、merge 結果に対してパッチのチェックを走らせ、通れば merge する。それ以外は merge する。trunk が merge 準備を超えて 3 度目の rebase を強いるなら、保留して root に報告する。root が衝突する作業を順序付ける。
6. **root のレイヤーを走らせる。** ゲートを緩めるもの、たとえば引き上げた予算や allowlist のエントリは、verifier の証明を得た後にあなたの新鮮な副署が要る。green を欲しがるのは owner だからだ。承認をカバーする付与の下では、その副署が承認そのものとなり、owner の記録はレーンが確認できるようあなたの副署を指す。無効化された、または soft-fail にされたチェックは必ず外す。あらゆるケースを守らなくなるからだ。forge が強制する承認は、決して自ら与えたり回避したりしない。どの副署もそれをカバーしないからだ。

   operator の go を受けたら、委任した作業が残らなくなるまで監査するよう `/loop 1h` を装填する。長い run は drift し、エージェントは黙って stall するからだ。各 tick はこの playbook を trunk から読み直し、drift、たとえば未報告の head や未記録のエージェントを、その owner に記録させて直す。drift は stall ではなく記録の欠落だからだ。末端のレーンに至るまで全エージェントを探り、それぞれを副作用で判断する。ステータスの文面や要約は stuck した子を隠しうるからだ。エラーになる、ターンを開始できない、stuck-after の実行時間を過ぎても新しいものが何も無いエージェントは stuck であり、最初の tick までに push も trail の行の書き込みも無い owner も同じだ。あなたや operator を待っているものは stuck ではないので、答える。stuck したエージェントは、その stop が届くのを待たず直ちに置き換える。operator のために何かを変えた tick は **ketchup** skill に従って operator に追いつかせて終わり、何も変えなかった tick は静かにしておく。
7. **operator の stop で直ちに stand down する。** hold は書き込みゼロの命令として全 owner に届く。各 owner は自分の現状を報告し、operator が解放するまで brief を保持する。

**返答:** **ketchup** skill に従って operator に追いつかせる。各 PR の状態と verdict、何が merge されたか、副署とその理由、未処理の operator ゲートを含め、各 PR と trail へのリンクを付ける。
