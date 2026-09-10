フォークの構造を見ました（`plugins/pstack/skills/no-comments/SKILL.md`、marketplace 名は `open-pstack`、プラグイン名は `pstack`）。できます。ただし upstream を追従したいなら、**ディレクトリ削除ではなく frontmatter 1行追加**にするのをおすすめします。

## 1. no-comments をオフにする（差分を最小に）

`plugins/pstack/skills/no-comments/SKILL.md` の先頭をこうする:

```yaml
---
name: no-comments
description: "Spawn the comment-sicko subagent, ..."
disable-model-invocation: true
---
```

これで description がコンテキストに載らず、Claude は自動起動できなくなります（`/pstack:no-comments` と手で打てば動く）。差分が1行なので、upstream がそのファイルの frontmatter を触らない限りマージ衝突しません。

削除を勧めない理由がもう一つ。`poteto-mode/SKILL.md` の28行目に「Before review → the no-comments skill」とあり、playbooks 数本からも参照されています。消すと poteto-mode が存在しないスキルを呼びに行きます。frontmatter 方式なら Claude Code が呼び出しをブロックして「別の手段で再現するな」と Claude に指示するので、poteto-mode は単にそのステップを飛ばす形になります。リポジトリの `tests/skill-collision-repro.sh` もこのフラグを検査するのは `principle-*` と how/why/unslop/typescript-best-practices だけなので、CI も通ります。

## 2. upstream の追従

```bash
git remote add upstream https://github.com/ericlitman/open-pstack.git
git fetch upstream
git merge upstream/main
git push origin main
```

自分の差分が1行だけなので、衝突があってもその1ファイルで済みます。

## 3. フォークを marketplace として登録

```
/plugin uninstall pstack@open-pstack        # 本家版を入れていれば
/plugin marketplace add seiei-sogen/open-pstack-sogen
/plugin install pstack@open-pstack
```

注意点: marketplace.json の `name` が `open-pstack` のままなので、本家の登録を**置き換える**形になります（同名 marketplace は1つしか登録できない）。本家と並存させたいなら `name` を `open-pstack-sogen` などに変えてください。これも upstream がほぼ触らないフィールドなので衝突リスクは低いです。

## 4. 以降の更新フロー

upstream をマージして push したら:

```
/plugin marketplace update open-pstack
/plugin update pstack@open-pstack
```

プラグインの版は `plugin.json` の `version` で固定されているので、upstream のバージョンアップに乗るぶんには自動で反映されます。upstream の bump なしに自分だけ何か変えたときは、同じ version だとキャッシュが更新されないので、uninstall → install し直すか、`version` を `1.4.0-sogen.1` のように一時的に上げてください。