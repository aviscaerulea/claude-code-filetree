# ローカル改修の要件と更新手順

このフォークの `windows-herdr` ブランチは、上流の filetree に Windows の herdr で使うための改修を加えたものだ。
上流が更新されたら、この文書の手順で取り込む。次に要件をすべて満たすことを確かめる。

## 要件

| 番号 | 要件 |
| --- | --- |
| R1 | Windows で herdr の中で動いている間、filetree は Diff with、Open with: editor、`e`、`t` の外部コマンドを herdr のペインで起動する |
| R2 | Windows で herdr のペインへコマンドを送るとき、filetree は cmd.exe の書式（語を二重引用符で囲み、対象を `%PANE_OPEN_TARGET%` で渡す）で送る |
| R3 | herdr のペインで起動した外部コマンドが正常終了したとき、filetree はそのペインを閉じる |
| R4 | herdr のペインで起動した外部コマンドが異常終了した場合、filetree はエラーを読めるようにペインを残す |
| R5 | Diff with を設定している間、git 管理外のファイルかフォルダで `d` を押した場合、filetree は外部コマンドを起動せずに「`<パス> is not in a git repository`」のトーストを出す |
| R6 | Diff with が hunk の構成では、filetree が開いた同じリポジトリの hunk が動いている間に `d` を押したとき、filetree は新しいペインを開かずに、その hunk の表示中のファイルへ押したパスを加え、ファイルならそこへ移動する |
| R7 | Diff with が hunk の構成では、filetree が開いた同じリポジトリの hunk が動いていない場合、filetree は新しいペインで hunk を起動し、そのセッションを次の `d` のために覚える |
| R8 | filetree は、自分で開いていない hunk のセッション（hunk-watch などのリポジトリ全体の表示）を変更しない |

## 実装の場所

改修した箇所にはすべて「ローカル改修」で始まるコメントを付けてあり、`rg "ローカル改修" hooks` で一覧できる。

| 要件 | ファイル | 箇所 |
| --- | --- | --- |
| R1 | `hooks/register.tsx` | `editorHost` の win32 の判定、起動時の `inHerdr` の判定 |
| R2, R3, R4 | `hooks/open.ts` | `herdrOpenCommand` の `windows` 引数 |
| R2 | `hooks/register.tsx` | `openInHerdr` から `herdrOpenCommand` へ `windows` を渡す |
| R5 | `hooks/register.tsx` | `diffFile` の `git rev-parse --show-toplevel` の判定 |
| R6, R7, R8 | `hooks/register.tsx` | `diffFile` の herdr の分岐と、`hunkSessions`、`addToHunk`、`rememberHunk` |

上流のテスト「open with editor in an unsupported environment: terminal on win32」は、R1 で Windows を対象に含めたため失敗するが、想定どおりの失敗として扱う。

## 上流の更新の取り込み

`~/.claude/mods/filetree`（このクローン）で次の順に実行する。

1. 上流の更新を取得する：`git fetch upstream --tags`
2. 取り込む版を決める：上流の新しいリリースのタグ（例：`v0.8.3`）か `upstream/main` を選ぶ
3. 作業ブランチへマージする：`git merge <タグか upstream/main>`
4. 競合したら、上の要件を満たす形で解消する
   - 上流が同じ関数を書き換えていたら、上流の新しいコードを正として、「ローカル改修」のコメントの箇所を当て直す
   - 上流が Windows や herdr に対応して要件を満たすようになった箇所は、改修を外して上流に合わせる
5. 検証する
   - `claude plugin test .`：失敗が上記の 1 件だけであることを確かめる
   - `aubx --package typescript@5 tsc --noEmit -p tsconfig.json`：終了コードが 0 であることを確かめる
   - Claude Code で `/reload-plugins` を実行する
   - R1〜R8 を画面で確かめる
6. フォークへ送る：`git push origin windows-herdr`

マージで取り込むので、force push は要らない。
