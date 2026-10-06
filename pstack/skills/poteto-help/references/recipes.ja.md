# コピーする価値のあるプロンプト

実際のパス、skill、完了のチェックに差し替えること。口語的な言い方で構わない。

## 理解する

- `/poteto-mode read <thread>. restate the underlying issue in your own words, in plain english.`
- `/poteto-mode investigate why <symptom>. give me what we know, what data you used, and your best hypotheses. don't change any code yet.`
- `use /how to understand <subsystem>. then use /why to find out why it broke recently.`
- `/recall my work on <topic> from last week, then read <issue>.`
- `/teach me why you implemented it this way and not <other way>. what did you trade off?`
- `/poteto-mode take over this branch. read the decision log, find what's done, and continue. don't redo finished work.`

## 作る

- バグ: `/poteto-mode <symptom>. repro first, then fix and verify.`
- アプリのバグ: `/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.`
- 安いテストがあるバグ: `/poteto-mode repro <bug> first. if there's a cheap test path, /tdd it. then fix and rerun.`
- 機能: `/poteto-mode add <behavior>. <current output> stays byte-identical. verify both.`
- リファクタリング: `/poteto-mode move <code> into one module, zero behavior change. record the current output first and prove it's unchanged after.`
- 性能: `/poteto-mode <operation> takes <time> on <fixture>. trace it, fix the measured cause, show me before and after.`

## 設計と計画

- `/poteto-mode prototype a few options for <feature>. take screenshots or videos for me to compare.`
- `/poteto-mode we need <feature>. /architect it first, and answer open questions with prototypes. let me review before proceeding.`
- `/poteto-mode write a tutorial for how i would use <new package> first. then /teach me why it beats the current one.`
- `ask /arena for a second opinion on this thread and our approach.`
- `/poteto-mode turn this design into a plan. small verifiable PRs, each with its own verification steps.`
- `/poteto-mode plan the migration of <library> to <target>. small verifiable PRs. the result must match the original exactly, bugs included.`

## レビューと出荷

- `/interrogate the whole branch, but skeptically. don't change anything yet. no nitpicks unless it's a real bug or regression.` 却下された指摘も読むこと。
- `/swarm check every package under <dir> against its check script. one worker per package. one report.`
- `/poteto-mode open the pr. small ordered commits, evidence in the description.`
- `/poteto-mode babysit this pr. get it green.` 状況確認だけなら: `/poteto-mode check on pr <number>. anything outstanding?`
- `/poteto-mode land the stack.`

## 離席と復帰

- `/poteto-mode im going to bed. <goal> in a fresh worktree off <base>. done means <checks>. keep a decision log. don't ask me before committing. /loop until done. if you're truly stuck after a few hours, stop and write up why.`
- `/show-me-your-work catch me up on what you did last night.` まず Attention セクションを読むこと。
- `/poteto-mode full autopilot on this queue. each item is independent.`
- `/poteto-mode autopilot these changes but stack them, don't ship. i'll land the stack.`
- `/reflect capture what we learned so the next run doesn't repeat it.` 将来の判断を変える編集だけを承認すること。
- `/bro` は直前の返答を平易な言葉で言い直す。
