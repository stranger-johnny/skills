---
name: writing-implementation-issues
description: Writes one GitHub issue that specifies the implementation plan in enough detail for any AI coding agent to carry it out. The issue splits the work into numbered steps that are each implemented separately. It includes a table that maps Japanese terms to names in each naming case the repository uses, a file tree of changes, Mermaid diagrams that treat the system as a black box and show which final actions each user operation causes, outline code examples, and the test conditions to cover. It follows the existing coding style and structure of the repository. Use when the user asks to create, write, or draft an issue, plan a feature or refactoring as an issue, break down a task for an AI agent, or write an implementation plan. Issue を作成、Issue を書いて、Issue にまとめて、実装計画、タスクを分解、エージェントに任せる作業を整理、と依頼されたときにも使います。
---

# 実装用 Issue の作成

このスキルは、実装の方針を細部まで決めた GitHub の Issue を1つ書く手順です。
Issue を読めば、どの AI エージェントでも同じ実装ができる状態を目指します。

## 方針

Issue は、どれほど大きなタスクでも1つにします。
方針、用語、作業ステップの順序が1か所にあれば、エージェントは他の Issue を探さずに済みます。

作業は作業ステップに分けます。
1つの作業ステップは、1回の実装と1つのプルリクエストで終わる大きさにします。
エージェントは一度に多くを実装すると、テストと規約の確認を省きがちだからです。

Issue は、筆者が作成時点で実装の中身を判断できるように書きます。
筆者が Issue を読んで方針の誤りに気づければ、実装の後で手戻りが起きません。

Issue の中の名前とコードは、既存のリポジトリの規約に合わせます。

## ワークフロー

次のチェックリストをコピーして、終わった項目に印を付けてください。

```text
- [ ] 手順 1: 依頼の目的と完了条件を確かめる
- [ ] 手順 2: コードベースの規約を調べる
- [ ] 手順 3: 用語の対応表を作る
- [ ] 手順 4: 変更ファイルのツリーを書く
- [ ] 手順 5: 作業を作業ステップに分ける
- [ ] 手順 6: 処理の図を描く
- [ ] 手順 7: コード例とテストの観点を書く
- [ ] 手順 8: 下書きを読み直す
- [ ] 手順 9: ユーザーに確認してから Issue を作る
```

### 手順 1: 依頼の目的と完了条件を確かめる

次の2点を、会話とリポジトリから拾います。
足りない点だけをユーザーに尋ねます。

1. 何のための変更か。利用者や開発者にとって何が変わるか。
1. 何ができたら完了か。

完了条件は、確かめられる文で書きます。
「招待の機能ができている」ではなく、
「登録済みのメールアドレスに招待を送ると、画面にエラーが出る」と書きます。

### 手順 2: コードベースの規約を調べる

Issue を書く前に、次の順にファイルを読みます。

1.  `CLAUDE.md`、`AGENTS.md`、`README.md`、`CONTRIBUTING.md`。
1.  リンターとフォーマッターの設定。
    `.eslintrc*`、`biome.json`、`pyproject.toml`、`.editorconfig` などです。
1.  今回の変更に最も近い既存の機能。
    同じ層のファイル、そのテスト、呼び出し元を読みます。

読んだ結果から、次の点を書き出します。

-   ディレクトリの分け方と、新しいファイルを置く場所。
-   ファイル名、関数名、型名、変数名の書き方。
    `kebab-case` か `snake_case` か、`UserRepository` のような接尾辞があるか、などです。
-   エラーの扱い方。例外を投げるか、`Result` 型を返すか、などです。
-   テストのフレームワーク、テストファイルの置き場所と名前。
-   依存の注入やモックの作り方。

規約ごとに、根拠にしたファイルを1つ以上添えます。
エージェントは、そのファイルを開いて書き方を真似できます。
既存のコードの中で書き方が揃っていない場合は、最近追加されたファイルに多い書き方を選び、
そう決めたことを Issue に書きます。

### 手順 3: 用語の対応表を作る

Issue に出てくる概念を、日本語と、コードで使う表記の表にまとめます。
表は Issue の前半に置き、以降の文、図、コード例の名前はすべて表に揃えます。
名前が揃っていれば、レビューで「この変数は何か」と迷う箇所が減ります。

表の列は、1列目を日本語にし、2列目以降にリポジトリで使う表記を並べます。
どの表記を列にするかは、手順 2 で調べた規約で決めます。
列の見出しには、表記の名前と、その表記を使う場所を書きます。
規約で使わない表記は、列にしません。

TypeScript のリポジトリで、変数と関数が camelCase、型が PascalCase、
ファイル名が kebab-case の場合は、次の表になります。

```markdown
| 日本語 | camelCase | PascalCase | kebab-case |
| --- | --- | --- | --- |
| 招待 | `invitation` | `Invitation` | `invitation` |
| 招待コード | `invitationCode` | `InvitationCode` | `invitation-code` |
| 招待を作る | `createInvitation` | `CreateInvitation` | `create-invitation` |
| 招待を送る | `sendInvitation` | `SendInvitation` | `send-invitation` |
| 招待リポジトリ | `invitationRepository` | `InvitationRepository` | `invitation-repository` |
| 招待の有効時間 | `invitationTtlHours` | `InvitationTtlHours` | `invitation-ttl-hours` |
| 登録済みエラー | `alreadyRegisteredError` | `AlreadyRegisteredError` | `already-registered-error` |
| メール送信サービス | `mailer` | `Mailer` | `mailer` |
```

1行の中のすべての列を埋めます。
どの表記も揃っていれば、エージェントは同じ概念に別の語を当てません。

既存のコードにすでに名前がある概念は、その名前を使います。
新しい名前は、手順 2 で調べた規約に従って付けます。
1つの概念には1つの名前だけを使い、`invite` と `invitation` を混ぜません。

### 手順 4: 変更ファイルのツリーを書く

変更するファイルを、ディレクトリのツリーで示します。
ツリーには、変更に関係するディレクトリだけを載せます。
各ファイルの行末に、`[追加]`、`[変更]`、`[削除]` の印と、変更の内容を書きます。
印の後ろに、そのファイルを変更する作業ステップの番号を書きます。
番号は手順 5 で作業ステップを決めてから書き足します。

```text
src/
├── domain/
│   ├── already-registered-error.ts [追加] Step 1 登録済みエラー
│   └── invitation.ts               [追加] Step 1 Invitation 型と有効期限の判定
├── repositories/
│   ├── invitation-repository.ts    [追加] Step 2 招待の保存と取得
│   └── user-repository.ts          [変更] Step 2 findByEmail を追加
└── usecases/
    └── send-invitation.ts          [追加] Step 3 招待を作ってメールを送る
tests/
├── domain/
│   └── invitation.test.ts          [追加] Step 1
└── usecases/
    └── send-invitation.test.ts     [追加] Step 3
```

### 手順 5: 作業を作業ステップに分ける

作業を、順に実装できる作業ステップに分けます。
分け方は、次の規則で決めます。

-   各作業ステップの後で、ビルドとテストが通る状態にします。
    途中で止まっても、マージ済みの作業ステップは壊れていません。
-   1つの作業ステップで変更するファイルは、テストを除いて5つ程度までにします。
    これを超えると、1回のレビューで差分を読み切れません。
-   他のファイルが依存するファイルから順に並べます。
    型と定数、データの読み書き、処理、画面や API の入り口の順が基本です。

Issue の完了条件の後に、作業ステップの一覧をチェックボックスで書きます。
各作業ステップに書く項目は、[Issue のテンプレート](assets/issue-template.md) にあります。

### 手順 6: 処理の図を描く

システムをブラックボックスとして外から見て、
どの操作をすると最終的にどのアクションが行われるかを、Mermaid の `sequenceDiagram` で示します。
UseCase や Repository のような内部のクラスと、その依存関係は描きません。
描き方は [処理の図の描き方](references/diagrams.md) に従います。

### 手順 7: コード例とテストの観点を書く

各作業ステップに、主要な機能の大枠が分かるコード例を載せます。
筆者は Issue を読む時点で、処理の流れを確かめられます。
細部まで書くとコード量が増え、Issue を読む負担が増えるので、細部は省きます。

コード例は、次の規則で書きます。

-   作業ステップの主要な関数やクラスだけを載せます。
    `import`、補助の関数、定数の定義は載せません。
-   コードブロックの1行目に、ファイルのパスをコメントで書きます。
-   関数のシグネチャと、処理の大枠を書きます。
    処理の大枠とは、主要な呼び出し、分岐、エラーの返し方です。
-   入力の検証、値の計算、ログのような細部は、`// ...` に続けて、
    そこで何をするかを1行で書いて省きます。
-   1つのコード例は、20行程度までにします。
    これを超えるなら、省ける細部が残っています。
-   既存の関数やクラスを変更する場合は、`diff` のコードブロックで変更箇所を書きます。
    変更箇所の前後の既存のコードを数行ずつ含め、どこに入るかが分かるようにします。
-   コード例の書き方は、手順 2 で調べた根拠のファイルに合わせます。

新しく作る関数は、次のように書きます。

````markdown
```ts
// src/usecases/send-invitation.ts
export async function sendInvitation(email: string, now: Date): Promise<Invitation> {
  // ... email の形式を検証する
  if (await userRepository.findByEmail(email)) {
    throw new AlreadyRegisteredError(email);
  }

  const invitation = createInvitation(email, now); // ... 有効期限は now の72時間後
  await mailer.send(email, invitation.invitationCode);
  await invitationRepository.save(invitation);
  return invitation;
}
```
````

既存のクラスを変更する場合は、次のように書きます。

````markdown
```diff
// src/repositories/user-repository.ts
 export class UserRepository {
   constructor(private readonly db: Database) {}

   async findById(id: string): Promise<User | null> {
     return this.db.user.findUnique({ where: { id } });
   }
+
+  async findByEmail(email: string): Promise<User | null> {
+    return this.db.user.findUnique({ where: { email } });
+  }
 }
```
````

テストは、ケースの数ではなく、確かめる観点を揃えます。
観点ごとに、入力と期待する結果を1行で書きます。
加えて、テストしないことと、その理由を書きます。
テストしないことを書いておくと、エージェントがライブラリの動作や getter のような
意味の薄いテストを足しません。

```markdown
#### Step 1 のテストの観点

- [ ] 有効期限ちょうどの招待は期限切れと判定する。`expiresAt` と `now` が同じ時刻で `true`。
- [ ] 有効期限の1ミリ秒前の招待は期限内と判定する。`false` を返す。

#### Step 1 でテストしないこと

- `invitationTtlHours` の値そのもの。定数を写すだけのテストになるからです。
- `Invitation` 型のフィールド。型チェックで確かめられるからです。
```

### 手順 8: 下書きを読み直す

下書きをリポジトリのルートの `issue-draft.md` に書き、次のチェックリストと照らし合わせます。
下書きはコミットしません。
問題があれば直し、すべての項目を満たすまで繰り返します。

```text
- [ ] 完了条件が、確かめられる文で書いてある
- [ ] 規約ごとに、根拠のファイルパスが書いてある
- [ ] 文、図、コード例の名前が、すべて用語の対応表と一致する
- [ ] ツリーのファイルがすべてどれかの作業ステップに出てきて、その逆も成り立つ
- [ ] 各作業ステップの後で、ビルドとテストが通る
- [ ] 各作業ステップに、テストの観点と完了条件がある
- [ ] 各作業ステップに、主要な機能の大枠が分かるコード例がある
- [ ] 図に内部のクラスが出てこず、操作とアクションと、結果を左右する判定だけが描いてある
- [ ] 図のアクションが、コードを読んで確かめたものである
- [ ] テンプレートの TODO がすべて埋まっているか、削除してある
```

用語の一致は、対応表の日本語以外の列の各語を、
下書きの中で `grep` して確かめます。
表にない名前がコード例に出てきたら、表に足すか、コード例を直します。

### 手順 9: ユーザーに確認してから Issue を作る

下書きのパスと、作業ステップの一覧をユーザーに見せます。
Issue を作るとリポジトリの参加者に通知が届き、通知は取り消せないからです。
ユーザーが了承したら、下書きの H1 の行をタイトルにし、本文から H1 の行を削除して、
次のコマンドで Issue を作ります。

```bash
gh issue create \
    --title "招待メールでユーザーを登録できるようにする" \
    --body-file issue-draft.md
```

作成した Issue の URL をユーザーに伝え、`issue-draft.md` を削除します。
`gh` が使えない場合は、下書きのパスを伝え、ユーザーに貼り付けてもらいます。

## 詳細資料

-   [Issue のテンプレート](assets/issue-template.md) は、Issue の本文の出発点です。
    手順 3 から手順 7 で、このテンプレートを埋めます。
-   [処理の図の描き方](references/diagrams.md) は、手順 6 で読みます。
