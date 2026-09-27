### 複数フェーズ / 複数 PR のプラン

**あなたが持つのはプランであってコードではない。プランとは、オーナーが 1 box ずつ実行し、オペレータが証拠から監査するチェックリストである。** プランが成果物である。実装してはならない。

1. 変更が 1 つか 2 つのファイルで、アプローチが自明なら、プランは省く。そう述べて止まる。
2. 書き始める前に、未解決の問いはプロトタイプで決着させる。それぞれについて `playbooks/prototype.md` を実行する。ブランチ、SHA、スクリーンショットを Appendix A のために保持する。オペレータに聞くのは、どんな実行でも決着しないプロダクト上・好み上の判断だけにする。選択肢を提示する (**never-block-on-the-human** の principle skill)。
3. `subagent_type: "poteto-agent"` のサブエージェントで、Subagents セクションに従って明示的にモデルを指定して探索する (**guard-the-context-window** の principle skill)。各サブエージェントはファイルへのポインタ、規約、テストコマンド、エントリポイントを返す。インライン展開したダンプは返さない。
4. 下のスケルトンをプランファイルへコピーし、すべてのプレースホルダを埋める。オペレータがパスを指定しない限り、ファイルはエージェントストアの `docs/` 配下に書く。すべての見出しとすべてのサブブロックを、示された順序のまま保つ。1 PR につき 1 セクション。1 PR は、それ自身の証拠を伴う 1 つの変更である (**sequence-verifiable-units** の principle skill)。**How to read this** に実行 playbook を名指しする。`playbooks/autopilot-full.md` と `playbooks/autopilot-stack.md` のどちらを選ぶかは、`playbooks/autopilot-stack.md` の末尾にあるルールに従う。常設プログラムは `playbooks/orchestrate.md` を取る。
5. `/technical-writing` に従って全文を書き、その後 `/unslop` をかける。本文は Diátaxis の 1 モード、how-to である。付録が explanation と reference を持つ。各見出しはタスクか発見を述べる。長いダッシュを使わない。文中のコロンを使わない。
6. `node pstack/skills/poteto-mode/scripts/check-plan.mjs <plan.md>` を実行し、出力された行をすべて修正する (**encode-lessons-in-structure** の principle skill)。
7. 引き渡す。プランのパスとスクリプトの出力を投稿し、そこで止まる。実行が始まるのはオペレータの明示的な go の後であり、プランが名指しした実行 playbook のもとで行う。

**検証。** テストだけでは十分な検証にならない。PR が検証済みになるのは、その unit、live、perf の box がすべてチェックされたときだけである (**prove-it-works** の principle skill)。この一文が検証ルールである。すべての verification ブロックはこの一文で始まる。live ブロックは必須である。PR の head における 10 レーンが、その control skill を通して実際の表面を駆動する。**swarm** skill に従い、`swarm workers` モデル (デフォルト `grok-4.7-xhigh-fast`) 上で行う。各レーンは 1 box であり、具体的なシナリオ、保存するスクリーンショット、合格判定の述語を持つ。1 レーンは **trunk に対する回帰レーン** とする。同じ load-bearing なシナリオを trunk と head の両方で実行する。trunk にその機能が無い場合、レーンはその事実を記録し、trunk の結果をでっち上げる代わりに、diff が追加する挙動とユーザが待つ最終状態をゲートする。perf ゲートは両側で見る。trunk と head の両方が、名指しした指標を出さなければならない。trunk に機能が無い場合は、diff が追加する作業を切り出して、その作業に絶対的な予算を置き、さらにユーザが待つエンドツーエンドの状態も置く。異質なシナリオ同士の比率を主張してはならない。perf ブロックは、指標、インターリーブしたプローブ、最初に測る trunk のベースライン、そして不合格になる数値を伴うルールを名指しする。インタラクションを変える PR は review ゲート付きである。オペレータがマージ前にチャットでスクリーンショットと動画を見てレビューする。インタラクションを変えない PR は `**Review gate.** None. <PR id> is not review-gated.` と書き、その下に box を置かない。

**Control skill。** 表面によって選ぶ。ブラウザ、Electron、Web UI は `cursor-team-kit` の `control-ui` を使う。CLI と TUI は `cursor-team-kit` の `control-cli` を使う。ネイティブモバイルは、そのリポジトリが持つシミュレータ駆動 skill を何であれ使う。2 つの表面に触れる PR は両方にレーンを持つ。control skill が無い表面は Appendix C のリスクとして扱い、その live ブロックは各レーンがどう駆動するかを依然として名指しする。

````markdown
# <Program> plan

<Under ten lines. What changes, for whom, the rule the program enforces, and the PR ids in order.>

## How to read this

One box is one unit of work. Every box names the evidence that checks it. A nested box is a sub-step of the box above it. Check a box only when its evidence exists, a file, a log line, a screenshot, a test run, or a SHA. The body is a how-to. The appendices explain and record.

The program runs `pstack/skills/poteto-mode/playbooks/<execution playbook>.md`. <Who merges, and which PR ids are the operator's items that stop at merge-ready.>

Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked.

## Program checklist

### Arm the program

- [ ] State the protocol and this plan to the operator, then stop. Start execution only on the operator's explicit go.
- [ ] On the operator's go, arm a `/goal` with this exact text. "<The plan path, the PR ids in order, the verification rule, who merges, and the done condition.>"
- [ ] Read these from trunk at program start. Re-read them at every tick.
  - [ ] `git show origin/main:pstack/skills/poteto-mode/playbooks/<execution playbook>.md`
  - [ ] `git show origin/main:pstack/skills/swarm/SKILL.md`
  - [ ] `git show origin/main:<control skill path>`
  - [ ] `git show origin/main:pstack/skills/poteto-mode/playbooks/opening-a-pr.md`
  - [ ] `git show origin/main:pstack/skills/<each other leaf skill the program uses>`
- [ ] Arm the 30-minute audit tick. In a local session, a real terminal `/loop`. In a cloud root, a cloud-sleeper wake chain. Never leave the cadence to memory.
- [ ] Use this tick prompt, verbatim. "Re-read the execution playbook from trunk and the armed /goal. Audit the operation against both and fix drift in this tick. Probe every active lane and judge progress by side effects only. Stand down a stuck lane and dispatch its replacement now. Then post a short status message to the operator in chat only when the audit found a tracked change that no earlier status message reported, such as a PR opened, a code-ready head, a round launched or closed, a verdict, a merge, a stuck agent and the action taken, a blocker added or cleared, or a decision only the operator can make. Name every such change and nothing else. Do not repeat a table, the merged list, or an unchanged blocker. If the audit found none, end the turn with no reply text. Either way, log this tick's row in your decision trail. The row names the items reported, or none."
- [ ] On the operator's hold or stand-down, send every owner a zero-writes order at once.

### Spawn owners

- [ ] Spawn one owner per PR with the full lifecycle the execution playbook names.
- [ ] Follow this dependency graph. Start dependent work only after its parent merges, or base it on the parent branch when the execution playbook stacks.
  - [ ] <PR id> and <PR id> are independent and first. Both branch from `main`.
  - [ ] <PR id> after <PR id>.
- [ ] Hold the file boundaries. <PR id or class> touches only `<glob>`.
- [ ] Hold the review gate. <PR ids> change an interaction. They wait for the operator's review in chat with screenshots and a video before merge.

### PR mechanics, for every PR

- [ ] Resolve the forge once. Default to `gh`; if `command -v origin` succeeds and Origin can resolve the repository, use `origin pr` for every PR operation. Record any fallback to `gh`. Never require `gt`.
- [ ] Open the PR ready, never draft, with `origin pr create --status open --base <base-branch>` or `gh pr create --base <base-branch>` according to the resolved forge. A stack child targets its parent branch.
- [ ] Run the repo's lint and typecheck once before the PR-facing push. Push with hooks on.
- [ ] Run `/deslop` before each commit and `/no-comments` before review.
- [ ] Triage every Bugbot and security-reviewer comment per `../references/bugbot-triage.md`.
- [ ] Rebase onto current trunk before the code-ready report and babysit. Keep that merge base in fix rounds. Rebase again only at merge prep, on a `git merge-tree` conflict with trunk, or on a CI failure that comes from a change on trunk.

### Verdict and merge, for every PR

- [ ] At the code-ready head SHA and at each later push that changes the patch, run the swarm per `pstack/skills/swarm/SKILL.md`. One gates lane. The ten live lanes from the PR's **Verify, live** block. The perf lane from its **Verify, perf** block. Two or more audit lanes, each with its own focus, that read the diff and the receipts and distrust the PR body. The root audits the receipts in the merge-ready report before the verdict.
- [ ] Clean only when every lane is `PASS`. Findings go back to the owner, including a defect that a lane filed as a note. A new head gets a fresh swarm and a fresh verdict, except for results that stay valid under the patch-id rule in `playbooks/shipping.md`.
- [ ] <The merge or append rule from the execution playbook, with the patch-id rule from `playbooks/shipping.md`.>

### Boot recipe, for every live lane

Each live lane runs on its own cloud VM at the PR head. Drive through `control-ui` or `control-cli` from `cursor-team-kit`.

- [ ] `git fetch origin <head-branch> && git checkout <head SHA>`.
- [ ] <Start the backend and the surface. Wait for ready.>
- [ ] <Deliver input only through the control skill's commands. Name the read-only diagnostics.>
- [ ] Save every screenshot to `/tmp/swarm-<pr-id>/worker-<n>/<slug>.png` and return the paths with the report.

## <Task as a verb phrase> (<PR id>)

**Depends on.** <PR id, or None.>

**Files.**

- [ ] Edit `<path>`.
- [ ] Create `<path>`.
- [ ] Delete `<path>`.

**Build.**

- [ ] <One change. Name the symbol and the file.>

**You see.**

- [ ] <One observable result, with the exact log line or screen state.>

**Verify, unit.** Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked.

- [ ] <Test file and the case it gains.> Run `<command>`.

**Verify, live.** Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked. Ten lanes on `<swarm workers model>` at the PR head, per the boot recipe.

- [ ] Lane 1. Regression lane against trunk. Run <the same load-bearing scenario> at trunk and head. If trunk lacks the feature, record that and gate <the behavior the diff adds plus the end state the user waits for>. Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 2. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 3. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 4. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 5. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 6. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 7. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 8. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 9. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 10. <Scenario.> Save `<slug>.png`. Pass when <predicate>.

**Verify, perf.** Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked.

- [ ] Metric. <What is measured at both trunk and head. If trunk lacks the feature, also name the diff-added work and the end-to-end state the user waits for.>
- [ ] Probe. <The command or procedure, run at trunk and at the head, interleaved. Both sides must produce the metric.>
- [ ] Baseline. Record the trunk <value> first.
- [ ] Rule. <Head against trunk, with the number that fails. If the scenarios differ, add absolute budgets for the diff-added work and the user-visible end state instead of an invalid ratio.>

**Review gate.** The operator reviews before merge.

- [ ] Copy lane <n> screenshots into `<media path>/<pr-id>-review-<slug>.png`.
- [ ] Record a 30 to 60 second video of the change on a lane VM. Save it as `<media path>/<pr-id>-review.mp4`.
- [ ] Post the screenshots and the video in chat. Stop at merge-ready. Wait for the operator's click.

**Merge.**

- [ ] Root's clean verdict at the exact head SHA.
- [ ] Bugbot triage done.
- [ ] Rebased onto current trunk after the verdict, patch-id unchanged.
- [ ] <The owner squash-merges its own PR, or the root appends it to the base-branch stack and the operator lands it bottom-up.>

## Close the program

- [ ] Every box above is checked with its evidence.
- [ ] Reply to the operator with the report the execution playbook names.

## Appendix A. Prototype evidence

<Each open question a prototype answered, with the branch, the SHA, and the artifact links. Each question that stays unproven.>

## Appendix B. Alternatives rejected

<Each approach weighed and why it lost.>

## Appendix C. Risks

<Each risk with the PR it lands in and what the owner watches.>

## Appendix D. Links and reading list

<Docs to read before editing. Which PRs get `pstack/skills/how/SKILL.md` and `pstack/skills/interrogate/SKILL.md`. The trail per `pstack/skills/show-me-your-work/SKILL.md`.>
````

**返答:** プランのパス、依存関係付きの PR id と review ゲート付きの集合、プロトタイプが証明したことと未証明のまま残ったこと、そしてチェックスクリプトの出力。
