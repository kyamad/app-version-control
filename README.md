# app-version-control

iOSアプリの強制アップデート設定を、GitHub Pages 上の 1 つの `version.json` で一元管理するためのリポジトリです。

---

## 1. このリポジトリの目的

当社が公開している **48 本の iOS アプリ** について、以下の情報をまとめて管理します。

- App Store で公開中の最新バージョン（`latestVersion`）
- 利用を許可する最低バージョン（`minimumVersion`）
- 強制アップデートの有効・無効（`forceUpdateEnabled`）
- App Store のダウンロード URL（`appStoreURL`）
- 強制アップデート画面に表示するメッセージ（`message`）

各 iOS アプリは起動時に GitHub Pages 上の `version.json` を HTTPS で取得し、自分の Bundle Identifier に対応する設定を参照します。
インストールされているアプリのバージョンが `minimumVersion` 未満であれば、アプリ内で強制アップデート画面を表示します。

このリポジトリは **静的ファイルのみ** で構成されています。
サーバー、データベース、バックエンド API、Firebase、GitHub Actions などは一切使用しません。

**`version.json` が唯一の管理ファイルです。** 更新はこのファイルを直接編集して commit・push します。

---

## 2. 公開URL

| 種別 | URL |
| --- | --- |
| GitHub リポジトリ | https://github.com/kyamad/app-version-control |
| GitHub Pages | https://kyamad.github.io/app-version-control/ |
| version.json 直接URL | https://kyamad.github.io/app-version-control/version.json |

iOS アプリから参照するのは **version.json 直接URL** です。

---

## 3. version.json の各項目

```json
{
  "schemaVersion": 1,
  "updatedAt": "2026-08-06T07:45:00Z",
  "apps": {
    "SheepGame.app": {
      "appName": "メェメェスリープ",
      "platform": "iOS",
      "minimumVersion": "9.3",
      "latestVersion": "9.3",
      "forceUpdateEnabled": true,
      "appStoreURL": "https://apps.apple.com/jp/app/タップ連打で眠らせろ-メェ-メェ-スリープ/id6754670480",
      "message": "新しいバージョンへのアップデートが必要です。App Storeから最新版へ更新してください。"
    }
  }
}
```

### トップレベル

| 項目 | 型 | 説明 |
| --- | --- | --- |
| `schemaVersion` | 数値 | この JSON の構造バージョン。構造を変更しない限り `1` のまま変更しません。 |
| `updatedAt` | 文字列 | この JSON を最後に更新した UTC 日時（ISO 8601 形式、例: `2026-08-06T07:45:00Z`）。編集のたびに更新します。 |
| `apps` | オブジェクト | アプリ設定の集合。**キーは各アプリの Bundle Identifier** です。 |

### apps 配下の各アプリ

| 項目 | 型 | 説明 |
| --- | --- | --- |
| `appName` | 文字列 | アプリ名（日本語表記）。人間が識別するための項目です。 |
| `platform` | 文字列 | プラットフォーム。すべて `"iOS"` です。 |
| `minimumVersion` | 文字列 | **利用を許可する最低バージョン。** これ未満のバージョンが強制アップデート対象になります。 |
| `latestVersion` | 文字列 | App Store で公開中の最新バージョン。表示・参考用です。 |
| `forceUpdateEnabled` | 真偽値 | 強制アップデート機能の有効・無効。`true` で有効。 |
| `appStoreURL` | 文字列 | App Store の該当アプリページ URL。強制アップデート画面の遷移先に使用します。 |
| `message` | 文字列 | 強制アップデート画面に表示するメッセージ。 |

**バージョンは必ず文字列（ダブルクォート付き）で記述してください。**
`"6.0"` を `6.0` や `6` と書くと、値が壊れて判定が正しく動作しません。

---

## 4. minimumVersion と latestVersion の違い

- **`latestVersion`** は、App Store で公開中の最新バージョンです。
- **`minimumVersion`** は、利用を許可する最低バージョンです。
- **最新版ではないというだけで、強制アップデートにする必要はありません。**
- **強制アップデートの判定に使用するのは `minimumVersion` のみです。** `latestVersion` は判定に使用しません。

### 例

```json
{
  "minimumVersion": "1.1",
  "latestVersion": "1.3"
}
```

この場合の挙動は以下のとおりです。

| インストール中のバージョン | 判定 |
| --- | --- |
| 1.0 | 強制アップデート対象 |
| 1.1 | 利用可能 |
| 1.2 | 利用可能 |
| 1.3 | 利用可能（最新版） |

軽微な修正リリースでは `latestVersion` だけを上げ、`minimumVersion` は据え置くのが基本です。
重大な不具合の修正やサーバー仕様変更など、旧バージョンを使わせられない場合にだけ `minimumVersion` を上げます。

---

## 5. アプリ更新時の基本手順

> **注意:** 現在は「12. バージョン同期の自動化」により、手順4〜10は自動で行われます。
> 通常は App Store に新バージョンが並べば、最大6時間以内に `version.json` へ自動反映されます。
> 以下は自動化を停止した場合や、手動で先に反映したい場合の手順です。

1. 新しいバージョンを App Store Connect へ提出する
2. Apple の審査を通過する
3. **App Store で実際に新バージョンをダウンロードできることを確認する**
4. `version.json` の対象アプリ（該当する Bundle Identifier のブロック）を編集する
5. `latestVersion` を新しいバージョンへ変更する
6. **強制更新させる場合だけ** `minimumVersion` も変更する
7. `updatedAt` を現在の UTC 日時へ更新する
8. JSON 構文が壊れていないことを確認する（カンマ・ダブルクォート・波括弧）
9. commit・push する
10. GitHub Pages 上の `version.json` をブラウザで開き、内容が反映されているか確認する
11. 実機でアプリを起動し、意図した動作になっているか確認する

手順 3 を飛ばさないでください。App Store に新版が並ぶ前に `minimumVersion` を上げると、ユーザーは更新できないのにアプリを使えなくなります。

---

## 6. 強制アップデートを有効にする方法

以下の **両方** の条件を満たしたときに、強制アップデート画面が表示されます。

1. `forceUpdateEnabled` が `true` である
2. アプリの現在バージョンが `minimumVersion` 未満である

つまり、強制アップデートを実施したいときは、対象アプリの `forceUpdateEnabled` を `true` にしたうえで、`minimumVersion` を「使わせたい最低バージョン」へ引き上げます。

```json
"SheepGame.app": {
  "minimumVersion": "9.4",
  "latestVersion": "9.4",
  "forceUpdateEnabled": true
}
```

上記の場合、9.3 以前を使っているユーザーが強制アップデート対象になります。

---

## 7. 強制アップデートを解除する方法

強制アップデートを誤って有効にしてしまった場合など、緊急時には対象アプリの `forceUpdateEnabled` を `false` へ変更します。

```json
{
  "forceUpdateEnabled": false
}
```

`false` にすると、`minimumVersion` の値にかかわらず、そのアプリでは強制アップデート画面が表示されなくなります。

変更後は **GitHub へ push しただけで終わりにせず**、GitHub Pages 上の `version.json` を開いて、公開内容が実際に `false` へ変わっていることを必ず確認してください。GitHub Pages への反映には数十秒〜数分かかる場合があります。CDN キャッシュの影響を受けることがあるため、確認時はブラウザのスーパーリロードや、URL 末尾に `?t=20260806` のようなクエリを付けた再読み込みが有効です。

---

## 8. 別ユーザーとの共同運用

このリポジトリは複数人で更新できます。

### 共同編集者（Collaborator）の追加手順

1. GitHub でこのリポジトリを開く
2. **Settings** を開く
3. 左メニューの **Collaborators and teams** を開く
4. **Add people** をクリック
5. 追加したい相手の GitHub ユーザー名またはメールアドレスを入力して招待する
6. 権限は、`version.json` を直接編集してもらう場合は **Write** を選択する

Organization 配下のリポジトリの場合は、Collaborator 追加の代わりに Organization の Team へ追加し、Team に対してリポジトリ権限を付与する運用も可能です。実際に選択できる権限は、GitHub のプランや Organization の設定によって異なります。

招待された側は、GitHub から届く招待メールまたは通知から招待を承諾する必要があります。

### 複数人で作業する場合の注意

**編集を始める前に、必ず最新の main を取得してください。**

```
git pull origin main
```

これを怠ると、他のメンバーの変更を上書きしたり、コンフリクトが発生したりします。
`version.json` は 1 ファイルに 48 本すべてが入っているため、同時編集はコンフリクトの原因になります。更新作業は「取得 → 編集 → すぐ push」を短時間で行うことを推奨します。

---

## 9. Git 操作例

```
git pull origin main
git status
git diff
git add version.json
git commit -m "Update app version configuration"
git push origin main
```

- `git pull origin main` … 作業開始前に最新状態を取得する
- `git status` … 変更されたファイルを確認する
- `git diff` … 変更内容（差分）を確認する。push 前に必ず目視確認する
- `git add version.json` … 変更をステージする
- `git commit -m "..."` … コミットする
- `git push origin main` … GitHub へ反映する

---

## 10. GitHub 上で直接編集する場合

ローカルに clone せず、GitHub の Web 画面から編集することもできます。

1. GitHub で `version.json` を開く
2. 右上の鉛筆アイコン（Edit this file）をクリック
3. 対象アプリの `latestVersion` などを編集する
4. `updatedAt` を更新する
5. 画面下部の **Commit changes** をクリックしてコミットする

**注意点**

- JSON の **カンマ（`,`）** と **ダブルクォート（`"`）** を壊さないでください。1 文字でも欠けると JSON 全体が読み込めなくなり、全 48 本のアプリに影響します。
- 各アプリブロックの最後の項目（`message`）の行末にはカンマを付けません。
- バージョンは必ずダブルクォートで囲んでください。
- 編集画面の **Preview** タブや、コミット前の差分表示で変更箇所を確認してください。
- 不安な場合は、編集前の内容をコピーして手元に控えておいてください。

---

## 11. 重要な注意事項

- **App Store で新版が公開される前に `minimumVersion` を上げないでください。**
- 公開前に上げると、**ユーザーが更新できないのにアプリを使えなくなります。**
- **Bundle Identifier を変更しないでください。** アプリ側が自分の Bundle Identifier で設定を探すため、変更すると設定が見つからなくなります。
- **バージョンを数値として保存しないでください。** 必ず文字列で記述します。
- **`"6.0"` を `6` にしないでください。** 値が変わってしまいます。
- **App Store URL を勝手に変更しないでください。** URL 内の日本語やハイフンは App Store が生成したものです。
- **API キーや秘密鍵をこのリポジトリに置かないでください。** Public リポジトリのため、誰でも閲覧できます。
- **JSON 構文が壊れた状態で push しないでください。** push 前に必ず構文を確認します。
- **編集したら `updatedAt` を更新してください。**
- **push 後は必ず GitHub Pages の公開内容を確認してください。** push 完了は反映完了ではありません。
- **段階リリース（Phased Release）中は自動同期に注意してください。** 配信が行き渡る前に `minimumVersion` が上がる可能性があります（「12. バージョン同期の自動化」参照）。
- **手動で `minimumVersion` / `latestVersion` を下げても、次回の自動同期で App Store の値へ戻ります。**

---

## 12. バージョン同期の自動化（GitHub Actions）

`.github/workflows/sync-app-versions.yml` により、**App Store の最新バージョンを自動で `version.json` へ反映**しています。

### 仕組み

Apple が公開している Lookup API（`https://itunes.apple.com/lookup`）に、各アプリの App Store ID を問い合わせ、公開中の最新バージョンを取得します。認証情報・APIキー・外部サービス・追加パッケージは一切不要です。App Store ID は各アプリの `appStoreURL` 末尾（`/idNNNNNNNNNN`）から自動で取り出します。

つまり **App Store 自体が唯一の情報源**です。スプレッドシートやCSVを更新する必要はありません。

### 実行タイミング

- **6時間ごと**に自動実行（01:00 / 07:00 / 13:00 / 19:00 UTC = 日本時間 **10:00 / 16:00 / 22:00 / 翌04:00**）
- GitHub の仕様上、混雑時は予定時刻から数分〜十数分遅れて実行されることがあります
- リポジトリの **Actions** タブ →「Sync app versions from App Store」→ **Run workflow** で手動実行も可能
- 手動実行時に `dry_run` を `true` にすると、**差分の確認だけ行い commit しません**。挙動を確かめたいときに使ってください

### 更新される項目

差分があったアプリについて、以下を App Store の最新バージョンへ変更します。

```
minimumVersion
latestVersion
```

加えてルートの `updatedAt` を実行時刻（UTC）へ更新します。1件も差分がなければ、**何もcommitしません**。

`appName` / `platform` / `forceUpdateEnabled` / `appStoreURL` / `message` は自動更新の対象外で、一切変更されません。

### ⚠️ 運用上もっとも重要な点

この設定では **`minimumVersion` も同時に引き上げられます**。つまり、

> App Store に新バージョンが並ぶ → **最大6時間以内に自動反映** → **旧バージョンの利用者は起動時に強制アップデート対象になる**

という運用です。段階リリース（Phased Release）を使っている場合、まだ全ユーザーに配信が行き渡っていない段階で強制アップデートがかかる可能性があります。段階リリース中のアプリがある場合は、その期間だけワークフローを一時停止するか、対象アプリの `forceUpdateEnabled` を `false` にしてください。

### 安全装置

誤作動でユーザーがアプリを使えなくなる事故を防ぐため、以下を組み込んでいます。

- **バージョンを下げない** — App Store 側が `version.json` より古い値を返した場合は変更せず、警告として記録します
- **取得失敗が多いと中断** — 全体の25%（最低3件）を超えるアプリで取得に失敗した場合、`version.json` を一切書き換えずにワークフローを失敗させます。Apple 側の障害時に設定が壊れることを防ぎます
- **取得できなかったアプリは据え置き** — 失敗が閾値以内なら、そのアプリだけ変更せずに実行結果へ一覧表示します
- **書き出し前にJSON構文を検証** — 壊れたJSONがcommitされることはありません
- **アプリを削除しない** — `version.json` にあって App Store 側で見つからないアプリも、そのまま残ります
- **バージョン表記を解釈できない場合はスキップ** — 数字とドット以外を含むバージョンは触りません

### 実行結果の確認

**Actions** タブから各実行を開くと、サマリーに以下が表示されます。

- 対象アプリ数 / 更新件数 / 変更なし件数 / 取得失敗件数 / スキップ件数
- 更新したアプリの一覧（アプリ名・Bundle Identifier・更新前・更新後）
- 取得できなかったアプリ、スキップしたアプリ、Bundle Identifier が App Store と一致しないアプリの一覧

自動commitは `github-actions[bot]` 名義で行われ、GitHub Pages へは従来どおり自動反映されます。

### 一時停止・再開

**Actions** タブ → 左メニューの「Sync app versions from App Store」→ 右上の「…」→ **Disable workflow** で停止できます。再開は同じ場所から **Enable workflow** です。

### 手動編集との併用

従来どおり `version.json` を手で編集しても構いませんが、**`minimumVersion` / `latestVersion` を手動で下げても、次回の同期で App Store の値へ戻ります**。恒久的に自動更新から外したい場合は、ワークフローを停止するか、対象アプリの `forceUpdateEnabled` を `false` にしてください。

### 注意

GitHub の仕様上、**リポジトリに60日間まったく更新がないとスケジュール実行は自動的に無効化されます**。その場合は Actions タブから手動で再有効化してください（アプリを更新していれば自動commitが入るため、通常は発生しません）。

---

## ファイル構成

```
app-version-control/
├── .github/
│   └── workflows/
│       └── sync-app-versions.yml   … App Store バージョン自動同期（12章）
├── README.md                        … このファイル
└── version.json                     … 唯一の管理ファイル
```

新しいアプリを追加する場合は、`apps` オブジェクトへ同じ構造のブロックを 1 つ追加します。キーの並び順（`appName` / `platform` / `minimumVersion` / `latestVersion` / `forceUpdateEnabled` / `appStoreURL` / `message`）は全アプリで統一してください。
