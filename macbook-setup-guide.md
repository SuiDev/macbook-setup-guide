# MacBook セットアップガイド

2026/4 時点の、新しいMacBookの開発環境を整える手順をまとめています。

本ドキュメントは個人用のセットアップメモであり、いかなる組織・所属に関係しません。

---

## 目次

- [0. MacBook 初期セットアップ](#0-macbook-初期セットアップ)
- [1. macOS システム設定](#1-macos-システム設定)
- [2. Homebrew とパッケージ](#2-homebrew-とパッケージ)
- [3. Karabiner-Elements（キーボードカスタマイズ）](#3-karabiner-elementsキーボードカスタマイズ)
- [4. フォント設定](#4-フォント設定)
- [5. GUIアプリケーション](#5-guiアプリケーション)
- [6. プログラミング言語とランタイム](#6-プログラミング言語とランタイム)
- [7. Docker](#7-docker)
- [8. Git / GitHub](#8-git--github)
- [9. AWS CLI](#9-aws-cli)
- [10. Terraform関連](#10-terraform関連)
- [11. Visual Studio Code](#11-visual-studio-code)
- [12. Claude Code](#12-claude-code)
- [13. Codex CLI (OpenAI)](#13-codex-cli-openai)
- [14. その他のツール](#14-その他のツール)
- [15. シェル環境 (zsh)](#15-シェル環境-zsh)
- [16. SSH鍵と設定](#16-ssh鍵と設定)
- [17. 旧MacBookからの設定ファイル引き継ぎ](#17-旧macbookからの設定ファイル引き継ぎ)
- [18. ユーザーデータの移行](#18-ユーザーデータの移行)

---

## 0. MacBook 初期セットアップ

開発環境の構築に入る前に、macOS の基本的なアカウント設定とセキュリティ設定を済ませておく。

### 0.1 Appleアカウントのサインインと「探す」の有効化

1. 「システム設定」→ 画面上部の「Appleアカウントにサインイン」からサインインする
2. サインイン後、「システム設定」→「(自分の名前)」→「探す」を開く
3. 「Macを探す」を有効にする

これにより、紛失時のリモートロック・消去や位置情報の確認が可能になる。

### 0.2 パスワードの変更

初期設定時に設定したパスワードを変更する場合:

1. 「システム設定」→「Touch IDとパスワード」を開く
2. 「パスワード」セクションの「変更...」をクリックする
3. 現在のパスワードを入力し、新しいパスワードを設定する

### 0.3 Touch ID（指紋認証）の設定

1. 「システム設定」→「Touch IDとパスワード」を開く
2. 「指紋を追加」をクリックし、画面の指示に従って指紋を登録する
3. 必要に応じて複数の指を登録しておくと便利
4. 以下の用途でTouch IDを有効にする:
   - Macのロック解除
   - パスワードの自動入力

### 0.4 Macの名前（AirDrop表示名）の変更

AirDropで相手に表示されるMac名は、先に分かりやすい名前へ変更しておく。

1. 「システム設定」→「一般」→「情報」を開く
2. 「名前」をクリックする
3. AirDropで表示したい名前（例: `MacBook`）に変更する

ターミナルから変更する場合は以下でもよい。

```bash
sudo scutil --set ComputerName "MacBook"
sudo scutil --set LocalHostName "MacBook"
sudo scutil --set HostName "MacBook"
```

---

## 1. macOS システム設定

PCの基本操作に関わる設定を先に整えておく。

### 1.1 外観モード

「システム設定」→「外観」→ 外観モード: `ライト` を選択。
「システム設定」→「外観」→ Liquid Glass: `色合い調整` を選択。

### 1.2 ディスプレイ・システムのスリープ

ディスプレイの自動オフ、システムスリープ、ディスクスリープ、スクリーンセーバーをすべて無効にする。

```bash
# ディスプレイ/システム/ディスクのスリープを無効化 (-b: バッテリー駆動時, -c: 電源接続時)
sudo pmset -b displaysleep 0
sudo pmset -c displaysleep 0
sudo pmset -b sleep 0
sudo pmset -c sleep 0
sudo pmset -b disksleep 0
sudo pmset -c disksleep 0

# スクリーンセーバーを起動しない
defaults -currentHost write com.apple.screensaver idleTime -int 0
```

設定後は `pmset -g` で電源管理設定を確認できる。

GUIから設定する場合:

- 「システム設定」→「ロック画面」→「使用していない場合はディスプレイをオフにする」を「しない」
- 「システム設定」→「ロック画面」→「非アクティブ後スクリーンセーバーを開始」を「しない」
- 「システム設定」→「バッテリー」→「オプション...」→「ディスプレイがオフのときに自動でスリープさせない」を有効化（電源アダプタ接続時）

### 1.3 メニューバーの表示項目

メニューバーに以下が常時表示される状態にする。

- サウンド（音量コントロール）
- Bluetooth
- おやすみモード / 集中モード
- 入力ソース（あ/A）
- バッテリー残量（数値表示）+ 充電インジケータ

設定手順は「システム設定」→「コントロールセンター」を開き、各項目ごとに以下のように設定する。

| 項目         | 設定                                                |
| ------------ | --------------------------------------------------- |
| サウンド     | 「メニューバーに常に表示」                          |
| Bluetooth    | 「メニューバーに表示」                              |
| 集中モード   | 「メニューバーに表示」                              |
| 入力メニュー | 「メニューバーに表示」                              |
| バッテリー   | 「メニューバーに表示」 + 「割合を表示」を有効にする |

入力ソース (あ/A) が表示されない場合は「システム設定」→「キーボード」→「入力ソース」→「編集...」を開き、メニューバー上のステータスメニュー表示を有効にする。

#### フルスクリーンモードでのメニューバー常時表示

macOS のデフォルトでは、アプリをフルスクリーンにするとメニューバーが自動的に隠れてしまい、時刻やバッテリー残量などが見えなくなる。マウスカーソルを画面上端に持っていかなくても常に確認できるようにしておく。

「システム設定」→「コントロールセンター」→「メニューバー」セクション →「メニューバーを自動的に表示/非表示」を「しない」に変更する。

コマンドで設定する場合:

```bash
defaults write NSGlobalDomain AppleMenuBarVisibleInFullscreen -bool true
```

### 1.4 トラックパッド

GUIで合わせる場合の目安:

- `軌跡の速さ`: 最大
- `タップでクリック`: 有効
- `副ボタンのクリック`: `2本指でクリックまたはタップ`
- `ナチュラルなスクロール`: 有効

```bash
# タップでクリック: 有効
defaults write com.apple.AppleMultitouchTrackpad Clicking -bool true
defaults write com.apple.driver.AppleBluetoothMultitouch.trackpad Clicking -bool true

# 軌跡の速さ (スケーリング): 3 (最大)
defaults write NSGlobalDomain com.apple.trackpad.scaling -float 3

# ナチュラルスクロール: 有効
defaults write NSGlobalDomain com.apple.swipescrolldirection -bool true

# 強めのクリックと触覚フィードバック: 無効 (Force Click抑制)
defaults write com.apple.AppleMultitouchTrackpad ForceSuppressed -bool true
defaults write com.apple.AppleMultitouchTrackpad ActuateDetents -int 0

# クリックの強さ: 弱い (FirstClickThreshold = 0)
defaults write com.apple.AppleMultitouchTrackpad FirstClickThreshold -int 0
defaults write com.apple.AppleMultitouchTrackpad SecondClickThreshold -int 0

# 副ボタンのクリック (右クリック): 2本指でクリックまたはタップ
defaults write com.apple.AppleMultitouchTrackpad TrackpadRightClick -bool true
defaults write com.apple.AppleMultitouchTrackpad TrackpadCornerSecondaryClick -int 0

# USBマウス接続時のトラックパッド無効化: 無効 (常にトラックパッド使用可)
defaults write com.apple.AppleMultitouchTrackpad USBMouseStopsTrackpad -int 0
```

GUIから調整する場合は「システム設定」→「トラックパッド」で `軌跡の速さ`、`タップでクリック`、`副ボタンのクリック`、`ナチュラルなスクロール` を確認する。

### 1.5 キーボード

「システム設定」→「キーボード」で以下を設定する。`キーのリピート` は内部値が macOS バージョンによって見え方が分かりにくいため、コマンドではなくGUIで合わせる。

- `キーのリピート`: 最大
- `リピート入力認識までの時間`: 最短
- `環境光が暗い場合にキーボードの輝度を調整`: オフ

音声入力（Dictation）を使う場合のショートカット:

- 「システム設定」→「キーボード」→「音声入力」セクション → `ショートカット` を `Controlキーを2回押す` に変更する

#### 1.5.1 HHKB の引き継ぎ

外付けキーボード HHKB Professional HYBRID Type-S をメインで使用している。macOS の標準ドライバで動作するため、Mac 用ドライバのインストールは不要。

参考: https://happyhackingkb.com/jp/download/macdownload.html

HHKB は最大4台までの Bluetooth 接続先を本体に登録でき、`Fn + Ctrl + 1〜4` で登録済みスロットを切り替える。

##### 手順A: USB 接続で使う場合（もっとも簡単）

1. HHKB を USB-C ケーブルでMacBookに接続する
2. キーボード設定アシスタント（後述の「JIS/ANSI 判定」）を実行する

##### 手順B: Bluetooth で使う場合（スロット1を新MacBookに上書き）

1. HHKB 電源スイッチをオフ → 一度オンに戻す
2. HHKB 上で `Fn + Q` を押下してペアリング待機モードに入る
3. 続けて `Fn + Ctrl + <登録先スロット(1-4)>` を押して登録先スロットを指定（青 LED が高速点滅）
4. 新MacBookの「システム設定」→「Bluetooth」を開き、一覧に現れた `HHKB-Hybrid_1` の「接続」を押す
5. 画面に表示される数字（PINコード）を HHKB で入力し `Enter` を押す

##### JIS/ANSI 判定（初回接続時）

最近の macOS では初回接続時にキーボード設定アシスタントが自動起動しないため、以下のいずれかで手動起動する。

- 「システム設定」→「キーボード」→ `キーボードの種類を変更...` ボタン（HHKB 接続中に表示される）
- Spotlight (`⌘ + Space`) で `キーボード設定アシスタント` または `Keyboard Setup Assistant` と入力
- Finder で `/System/Library/CoreServices/KeyboardSetupAssistant.app` を直接開く

アシスタント起動後、画面の指示に従って左 `Shift` キーのすぐ右隣のキー、次に右 `Shift` キーのすぐ左隣のキーを押下すると、JIS/ANSI判定が完了する。

##### 修飾キー設定

「システム設定」→「キーボード」→「キーボードショートカット...」→「修飾キー...」で、対象キーボードに HHKB が選択されていることを確認する。本ガイドでは HHKB 側のキーマップで完結させるため、Mac 側の修飾キー入れ替えは既定のままとする。

PCでキーマップを変更したい場合のみ、PFU公式の `hhkb-keymap-tool` をインストールして編集する。Mac 側に設定ファイルは残らないため、必要に応じて書き出した設定ファイルを別途バックアップしておくこと。

### 1.6 入力ソース

- ABC (英字)
- 日本語入力 (macOS 標準 IME)

#### 1.6.1 macOS 標準 IME の設定

自動変換・自動修正を無効化する。

1. 「システム設定」→「キーボード」→「入力ソース」の「編集...」を開く
2. 「日本語 - ローマ字入力」を選択する
3. 「ライブ変換」のチェックを外す
4. 「タイプミスを修正」のチェックを外す

これによりスペースキーで任意のタイミングで変換する従来の挙動になり、入力中に勝手に修正されることもなくなる。

### 1.7 Finder

```bash
# 隠しファイルの表示
defaults write com.apple.finder AppleShowAllFiles -bool true

# すべてのファイル拡張子を表示
defaults write NSGlobalDomain AppleShowAllExtensions -bool true

# 既定の表示形式をカラム表示にする (clmv = column view)
defaults write com.apple.finder FXPreferredViewStyle -string "clmv"

# Finder を再起動
killall Finder
```

デスクトップのアイコン表示を整えるには、以下の手順で設定する。

1. デスクトップ上のどこかをクリックしてフォーカスを合わせる（Finderがアクティブになる）
2. 上部メニューバーの「表示」→「表示オプションを表示」を開く
3. 以下の値に合わせる

| 項目                     | 値                               |
| ------------------------ | -------------------------------- |
| 並べ替え                 | なし                             |
| 表示順序                 | グリッドに沿う                   |
| アイコンサイズ           | 72×72                            |
| グリッド間隔             | 中央よりやや左（デフォルト付近） |
| テキストサイズ           | 14                               |
| ラベルの位置             | 下側                             |
| 項目の情報を表示         | オフ                             |
| アイコンプレビューを表示 | オフ                             |

### 1.8 Dock

GUIで合わせる場合の目安:

- `サイズ`: 小と中の間
- `拡大`: 有効
- `拡大の大きさ`: 小と大の間

```bash
# Dock自動非表示を有効化
defaults write com.apple.dock autohide -bool true

# Dock を再起動
killall Dock
```

この環境では `Dock` のサイズ系はGUIで合わせる前提にする。
新Macで大きさが合わない場合は「システム設定」→「デスクトップとDock」で `サイズ` と `拡大の大きさ` を調整する。

### 1.9 テキストエディット

新規書類のデフォルトをリッチテキストから標準テキスト（プレーンテキスト）に変更し、起動時にファイル選択画面ではなく空の新規書類を直接開くようにする。

```bash
# デフォルトを標準テキスト（プレーンテキスト）にする
defaults write com.apple.TextEdit RichText -int 0

# 起動時にファイル選択画面ではなく新規書類を開く
defaults write com.apple.TextEdit NSShowAppCentricOpenPanelInsteadOfUntitledFile -bool false

# プレーンテキストのデフォルトフォントサイズを16ptにする
defaults write com.apple.TextEdit NSFontSize -float 16
```

設定後、テキストエディットを再起動すると反映される。

### 1.10 ログイン項目（起動時に自動で開くアプリ）

Mac 起動時（ログイン時）に自動で立ち上げるアプリをログイン項目に登録する。

GUIから設定する場合:

1. 「システム設定」→「一般」→「ログイン項目」を開く
2. 「ログイン時に開く」セクションの「+」ボタンをクリック
3. 以下のアプリを順に追加する

| アプリ         | パス                             |
| -------------- | -------------------------------- |
| Google Chrome  | /Applications/Google Chrome.app  |
| Slack          | /Applications/Slack.app          |
| Docker         | /Applications/Docker.app         |
| GitHub Desktop | /Applications/GitHub Desktop.app |
| cmux           | /Applications/cmux.app           |

コマンドで一括追加する場合:

```bash
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/Applications/Google Chrome.app", hidden:false}'
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/Applications/Slack.app", hidden:false}'
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/Applications/Docker.app", hidden:false}'
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/Applications/GitHub Desktop.app", hidden:false}'
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/Applications/cmux.app", hidden:false}'
```

登録後は「システム設定」→「一般」→「ログイン項目」で一覧に表示されていることを確認する。不要になったアプリは同画面から「-」ボタンで除外できる。

## 2. Homebrew とパッケージ

`brew` を使う手順がこの後に続くため、最初にここを実施する。

### 2.1 Homebrewのインストール

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
grep -qxF 'eval "$(/opt/homebrew/bin/brew shellenv)"' ~/.zprofile || echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Homebrew導入直後は macOS デフォルトの `zsh` から `brew` を使うため、`~/.zprofile` に `brew shellenv` を追記しておく。以降の `zsh` 設定はセクション15で行う。反映されない場合は、ターミナルを再起動してから再度 `brew --version` を確認する。

以降のシェル設定は macOS デフォルトの `zsh` 前提で進める。

---

## 3. Karabiner-Elements（キーボードカスタマイズ）

キーボードの操作感に直結するため、早い段階でセットアップしておく。

`brew` コマンドを使う場合は、先にセクション2.1のHomebrewインストールとPATH反映を実施すること。

### 3.1 インストール

```bash
brew install --cask karabiner-elements
```

### 3.2 設定の復元

`~/.config/karabiner/karabiner.json` をバックアップから復元する。

### 3.3 設定の有効化

`karabiner.json` を復元したら Karabiner-Elements を起動し、`Complex Modifications` でルールが有効になっていることを確認する。

必要に応じて以下も確認する。

1. `Karabiner-Elements` を起動する
2. `Complex Modifications` を開く
3. 復元したルールが有効になっていることを確認する
4. 初回起動時は `Input Monitoring` などの権限付与を求められたら許可する

### 3.4 設定されているカスタムルール

- Command + Arrow キーのカスタム動作
- 左Option単押し → バッククォート (`) を送信、長押し → 通常のleft_option
- コマンドキー単押し → 英数/かな切り替え（左Cmd: 英数、右Cmd: かな）

---

## 4. フォント設定

ターミナルやエディタで使うフォントを先に入れておきます。英語側は Nerd Font のグリフが必要な `Hack Nerd Font Mono`、日本語側はセルベースで組んでも幅が崩れにくい `Noto Sans Mono CJK JP` を併用する構成にしています。

参考記事: https://zenn.dev/kimkiyong/articles/7fd6ebf831a67f

### 4.1 フォントのインストール

```bash
# 英語側: Hack Nerd Font Mono（Powerline / Nerd Font アイコン対応）
brew install --cask font-hack-nerd-font

# 日本語側: Noto Sans Mono CJK JP（等幅 CJK フォント）
brew install --cask font-noto-sans-mono-cjk-jp
```

インストール後、`Font Book` でそれぞれが `Mono` バリアントとして追加されているかを確認します。プロポーショナル版を誤って導入するとターミナルで文字幅がブレてカーソル位置や整列が崩れるため、必ず `Mono` 版を使用してください。

### 4.2 cmux のインストールと設定ファイル

cmux は、複数のAIコーディングエージェントを並行して実行するために設計されたネイティブ macOS ターミナルアプリケーションです。Ghostty (libghostty) をベースに GPU 加速レンダリングを備えており、Claude Code などのエージェントを複数同時に走らせるワークフローに適しているため、本ガイドではメインのターミナルとして使用します。

Homebrew 経由でインストールします。

```bash
brew tap manaflow-ai/cmux
brew install --cask cmux
```

アップデートは `brew upgrade --cask cmux` を使用します。公式サイト (https://cmux.com/) から DMG をダウンロードして手動インストールすることもできます。

インストール後の初回起動:

1. cmux を起動する
2. 初回起動時にアクセシビリティ権限を求められた場合は許可する
3. 自動更新 (Sparkle) が組み込まれているため、以降のバージョンアップは自動で通知される

フォント設定は Ghostty と同じ設定ファイルで指定します。

- `~/.config/ghostty/config`

なお、cmux をインストールしただけでは `~/.config/ghostty/` ディレクトリも設定ファイルも自動生成されません。cmux は起動時にユーザーが事前に配置した設定ファイルを読みに行く設計のため、以下の手順でディレクトリとファイルを自分で用意する必要があります。

```bash
# ディレクトリを作成（存在しない場合）
mkdir -p ~/.config/ghostty

# 設定ファイルを新規作成
touch ~/.config/ghostty/config
```

作成したら `~/.config/ghostty/config` に以下の内容を追記します。Ghostty は `font-family` を複数行書くと、先頭から順にグリフの検索が行われ、見つからない文字は次のフォントにフォールバックします。これにより ASCII は Hack Nerd Font Mono、日本語は Noto Sans Mono CJK JP で描画される構成になります。

```
# 英語用のメインフォント（Nerd Font アイコンもここに含まれる）
font-family = Hack Nerd Font Mono

# 日本語用フォールバック（等幅 CJK フォント）
font-family = Noto Sans Mono CJK JP

# フォントサイズ
font-size = 14

# 細い文字をわずかに太らせて視認性を上げる
font-thicken = true

# 日本語グリフの縦方向の詰まりを少し緩める（CJK との相性調整）
adjust-cell-height = 2
```

設定を保存したら cmux を再起動して反映させます。日本語が描画されない・幅が揃わない場合は、Noto Sans Mono CJK JP が `Mono` バリアントで入っているか、`font-family` の綴りが完全一致しているかをまず確認してください。

### 4.3 VSCode 側の整合

ターミナルと同じ組み合わせを VSCode でも使う場合は、ユーザー設定の `settings.json` にフォント指定を追加します。VSCode は単一の `fontFamily` 文字列にカンマ区切りで書くと、OS のフォント解決機構でフォールバックが効きます。

対象ファイルのパス:

```
~/Library/Application Support/Code/User/settings.json
```

GUI から開く場合は以下の手順です。

1. VSCode を起動する
2. コマンドパレット (`⌘ + Shift + P`) を開く
3. `Preferences: Open User Settings (JSON)` を実行する

`settings.json` を開いたら、以下のエントリを既存の JSON オブジェクト内に追記します（末尾にカンマが必要な点に注意）。

```json
{
  "editor.fontFamily": "'Hack Nerd Font Mono', 'Noto Sans Mono CJK JP', monospace",
  "terminal.integrated.fontFamily": "'Hack Nerd Font Mono', 'Noto Sans Mono CJK JP', monospace"
}
```

保存すると即時反映されます。反映されない場合は VSCode の再起動、またはコマンドパレットから `Developer: Reload Window` を実行してください。

---

## 5. GUIアプリケーション

ブラウザやコミュニケーションツールなど、日常的なPC利用に必要なものから先にインストールする。

### ブラウザ

| アプリ        | インストール方法                                  |
| ------------- | ------------------------------------------------- |
| Google Chrome | 公式サイト or `brew install --cask google-chrome` |

### コミュニケーション

| アプリ | インストール方法                         |
| ------ | ---------------------------------------- |
| Slack  | App Store or `brew install --cask slack` |
| Zoom   | 公式サイト or `brew install --cask zoom` |

#### Slack のアプリ内設定

インストール後、以下のテーマ設定を行う。

1. Slack を起動し、左上のワークスペース名をクリック → 「環境設定」を開く（または `⌘ + ,`）
2. 「テーマ」セクションを開く
3. 「モード」で「ライト」を選択する
4. 「カラー」セクションで好みの色を選択する

### ユーティリティ

| アプリ        | インストール方法                                                            | 備考                                             |
| ------------- | --------------------------------------------------------------------------- | ------------------------------------------------ |
| Raycast       | `brew install --cask raycast`                                               | 設定の移行は本節末尾を参照                       |
| Logi Options+ | 公式サイト (https://www.logitech.com/ja-jp/software/logi-options-plus.html) | ロジクールマウスの設定管理。設定は本節末尾を参照 |

### AI / クリエイティブ

| アプリ         | インストール方法 |
| -------------- | ---------------- |
| Claude Desktop | 公式サイト       |

### ノート・知識管理

| アプリ   | インストール方法                                                            |
| -------- | --------------------------------------------------------------------------- |
| Obsidian | 公式サイト (https://obsidian.md/download) or `brew install --cask obsidian` |

#### Obsidian のインストールと初期設定

Obsidian はローカルのMarkdownファイルを基盤とするノートアプリです。Vault（ノートを格納するフォルダ単位の保管庫。`.obsidian/` 配下に設定・プラグイン・テーマがまとまって保存される）単位で管理されるため、旧MacBookからのデータ引き継ぎは Vault フォルダをそのままコピーするだけで完結します。

1. Obsidian をインストールする

   ```bash
   brew install --cask obsidian
   ```

2. 初回起動時に Vault の選択画面が表示される
   - 新規作成する場合は「Create new vault」から任意の場所（例: `~/Documents/ObsidianVault`）にVaultを作成する

3. 外観をダークモードに設定する
   - `⌘ + ,` で設定を開く
   - 「外観」タブ → 「ベーステーマ」で「ダーク」を選択する

### メディア

| アプリ        | インストール方法                                                                                                  |
| ------------- | ----------------------------------------------------------------------------------------------------------------- |
| BlackHole 2ch | `brew install --cask blackhole-2ch`（詳細セットアップは本節末尾の「BlackHole による内部音声ルーティング」を参照） |

### 開発ツール（GUIアプリ）

| アプリ             | インストール方法                                       |
| ------------------ | ------------------------------------------------------ |
| Visual Studio Code | 公式サイト or `brew install --cask visual-studio-code` |
| Docker Desktop     | 公式サイト or `brew install --cask docker`             |
| GitHub Desktop     | 公式サイト or `brew install --cask github`             |
| Postman            | 公式サイト or `brew install --cask postman`            |

#### GitHub Desktop のアプリ内設定

インストール後、以下の設定を行う。設定画面は「GitHub Desktop」→「Settings...」（または `⌘ + ,`）から開く。

Appearance（外観）:

1. 「Appearance」タブを選択する
2. 「Dark」を選択する

### Raycast の設定移行

旧データのエクスポート手順:

1. Raycast を起動し、`Cmd + ,` で Settings を開く
2. `Advanced` タブに移動する
3. 画面下部の `Export` ボタンをクリックし、出力先とパスフレーズを指定する
4. `.rayconfig` ファイルが生成されるので、AirDrop やクラウドストレージ等で新MacBookに転送する

インポート手順:

1. `brew install --cask raycast` で Raycast をインストールする
2. Raycast を起動し、初回セットアップを進める
3. `Cmd + ,` で Settings → `Advanced` タブを開く
4. `Import` ボタンから `.rayconfig` を選択し、エクスポート時のパスフレーズを入力する
5. インポート完了後、ホットキー、Quicklinks、Snippets、拡張機能などが復元されていることを確認する

### Raycast Window Management の設定

Raycast に組み込みのウィンドウ管理機能があるため、Settings → Extensions → Window Management でホットキーを割り当てる。

よく使う操作のホットキー例（`⌃ + ⌥` ベース、ShiftIt / Rectangle と同系統）:

| 操作        | ホットキー      |
| ----------- | --------------- |
| Left Half   | `⌃ + ⌥ + J`     |
| Right Half  | `⌃ + ⌥ + K`     |
| Top Half    | `⌃ + ⌥ + I`     |
| Bottom Half | `⌃ + ⌥ + M`     |
| Maximize    | `⌃ + ⌥ + Enter` |
| Center      | `⌃ + ⌥ + C`     |

好みに合わせて変更可。設定はエクスポート（`.rayconfig`）に含まれるので、一度設定すれば新PCへの移行時にも引き継がれる。

### Raycast Clipboard History の有効化

Raycast にはクリップボード履歴機能が組み込まれているため、有効化する。

1. Raycast を起動（`⌥ + Space` 等）し、`Clipboard History` と入力して実行する
2. 初回は「アクセシビリティ」の権限を求められるので許可する
3. 以降、同じコマンドで直近のコピー履歴を一覧・検索・貼り付けできる

ホットキーを割り当てておくと便利。

1. `Cmd + ,` で Settings → `Extensions` タブを開く
2. `Clipboard History` を探す
3. `Hotkey` 欄にショートカット（例: `⌘ + Shift + V`）を割り当てる

保持期間の変更は同じ `Clipboard History` の設定から行う（デフォルトは直近30日分を保持）。

### Logi Options+ の設定

ロジクールマウスのボタン割り当てとスクロール方向を設定する。macOS はトラックパッドとマウスのスクロール方向がシステム設定で連動してしまうため、Logi Options+ 側で独立して変更する。

1. https://www.logitech.com/ja-jp/software/logi-options-plus.html からダウンロード・インストール
2. マウスを USB レシーバーまたは Bluetooth で接続し、Logi Options+ を起動
3. 接続されたマウスを選択し、以下を設定する

| 設定項目                                     | 値                           |
| -------------------------------------------- | ---------------------------- |
| スクロールホイールのクリック（ミドルボタン） | Mission Control              |
| スクロールの方向                             | 標準（ナチュラルではない方） |

macOS の「システム設定」→「トラックパッド」→「ナチュラルなスクロール」は有効のままにしておく。Logi Options+ はトラックパッド側には影響しないため、トラックパッド=ナチュラル / マウス=標準 の独立設定が実現する。

### BlackHole による内部音声ルーティング

BlackHole は macOS 用の仮想オーディオループバックドライバです。`brew install --cask blackhole-2ch` だけではドライバが入るだけで、画面収録時に内部音声を録るには「マルチ出力装置」の作成と入出力の切り替えが別途必要になります。

macOS の画面収録機能や QuickTime Player で内部音声を録りたい場合のセットアップ手順は以下の通りです。

1. BlackHole をインストールする

   ```bash
   brew install --cask blackhole-2ch
   ```

   インストール直後は Audio MIDI 設定やサウンド環境設定に `BlackHole 2ch` が表示されないことがあります。その場合は Mac を一度再起動してから次の手順に進んでください（ログアウト・ログインでは認識されないケースが多いため、再起動を推奨します）。

2. 「Audio MIDI設定」（Spotlight で `Audio MIDI Setup` と検索）を開き、マルチ出力装置を作成する
   - 左下の `+` ボタン → `Multi-Output Device を作成`
   - 作成されたマルチ出力装置で、`Mac のスピーカー（またはヘッドフォン）` と `BlackHole 2ch` の2つにチェックを入れる
   - ドリフト補正は `BlackHole 2ch` 側を有効にしておくと安定します

3. Mac の音声出力先をマルチ出力装置に切り替える
   - メニューバーのサウンドアイコン、または「システム設定」→「サウンド」→「出力」で、先ほど作成したマルチ出力装置を選択します
   - この状態で Mac のスピーカーからも音が出つつ、BlackHole にも同じ音声が流れるようになります

4. QuickTime Player を起動する
   - `ファイル` →`新規画面収録` を選択します
   - オプション（▼ボタン）の `マイク` 項目で `BlackHole 2ch` を選択します

5. 録画を開始する
   - 画面と内部音声が同時に収録されます

初回起動時に「プライバシーとセキュリティ」→「画面収録とシステムオーディオの収録」（Screen & System Audio Recording）の許可を求められるため、QuickTime Player（または使用するアプリ）にチェックを入れてください。

録画終了後は、通常のスピーカー出力に戻すためサウンド出力先を元に戻すことを忘れないようにしてください。ステレオチャンネルで録りたい場合は `blackhole-2ch`、サラウンド等より多チャンネルが必要な場合は `blackhole-16ch` を選択します。

---

## 6. プログラミング言語とランタイム

各言語のバージョン管理は以下のツールで統一する。

| 言語    | バージョン管理     | 理由                                                                    |
| ------- | ------------------ | ----------------------------------------------------------------------- |
| Python  | uv                 | バージョン管理 + venv + パッケージ管理を1つで完結。pip の 10〜100倍速い |
| Go      | 標準ツールチェイン | `go install golang.org/dl/goX.Y.Z@latest` で追加バージョンを管理できる  |
| Node.js | mise               | 汎用ランタイム管理。pnpm もランタイムとして管理可能                     |

### 6.1 Python（uv）

uv で Python のインストールからパッケージ管理まで一括で行う。pyenv は使わない。

```bash
# uv のインストール
brew install uv

# 最新安定版をインストール
uv python install 3.14

# インストール済みバージョンの確認
uv python list --only-installed
```

古いバージョンが必要になったらその都度 `uv python install 3.13` 等で追加する。プロジェクト単位でバージョンを固定する場合は、プロジェクトディレクトリで `uv python pin 3.14` を実行すると `.python-version` ファイルが生成される。

CLIツールのインストールは `uv tool install` で行う（pipx の代替）。

```bash
uv tool install ruff
```

### 6.2 Go（標準ツールチェイン）

Go 1.21 以降、`go` コマンド自体がツールチェインの自動切替機能を持っている。Mac には Go を1つだけ入れ、プロジェクトごとのバージョン切替は `go.mod` の `go` / `toolchain` 行と `GOTOOLCHAIN=auto` に任せる。`golang.org/dl/go1.XX` 方式や gvm 等の追加ツールは不要。

公式リファレンス: https://go.dev/doc/toolchain

#### 6.2.1 インストール

```bash
brew install go
```

`go version` で `go1.26.x` が返ればOK。

#### 6.2.2 環境変数

`~/.zshrc` に以下を追記する（セクション15.4 の `.zshrc` 例にも含まれている）。

```bash
export GOTOOLCHAIN=auto
export GOPATH="$HOME/go"
```

`GOTOOLCHAIN=auto` にすると、`go.mod` が現在の Go より新しいバージョンを要求した場合、自動的にそのツールチェインをダウンロードして切り替える。

#### 6.2.3 プロジェクトごとのバージョン管理

各プロジェクトの `go.mod` で管理する。

```go
module example.com/myapp

go 1.24.0          // このモジュールの最低要件
toolchain go1.26.1 // 推奨ツールチェイン（オプション）
```

`go.mod` の Go 要件を更新するには以下を使う。

```bash
go get go@1.26.1
go get toolchain@1.26.1
```

#### 6.2.4 Go ツール群

`gopls`（言語サーバー）や `dlv`（デバッガ）等の開発ツールは、VSCode の Go 拡張が初回起動時に自動インストールを促すため、手動での `go install` は不要。lint / セキュリティ / コード生成系のツールはプロジェクトごとにバージョンが異なることが多いため、リポジトリ側で `tools.go` + Makefile で固定する運用を推奨する。

### 6.3 mise のインストール

Node.js のバージョン管理に mise を使用する。先にインストールしておく。

```bash
brew install mise
mise --version
```

`~/.zshrc` に `eval "$(mise activate zsh)"` が記述されていること（セクション15.4 参照）。シェル統合後、以下でインストール済みランタイムを確認できる。

```bash
mise list
```

### 6.4 Node.js（mise）

```bash
# Node.js をインストールしてグローバルにセット
mise use -g node@25.9.0
```

古いバージョンが必要になったらその都度 `mise use -g node@22.22.2` (LTS) 等で追加する。

---

## 7. Docker

### 7.1 Docker Desktop のインストール

Docker Desktop for Mac (Apple Silicon) をインストールする。

- 確認されたバージョン: Docker 28.0.4, Docker Compose v2.34.0

公式サイトからダウンロードするか、Homebrew Cask経由でインストールする:

```bash
brew install --cask docker
```

### 7.2 Docker Desktop の設定（GUI）

Docker Desktop の設定は全て GUI から行います。内部の設定ファイル（`settings-store.json`）を直接編集すると、Docker Desktop 側の自動書き換えと競合してデータ破損や起動不能につながる可能性があるため触りません。

設定画面を開く手順:

1. Docker Desktop を起動する
2. 右上の歯車アイコン、またはメニューバーのクジラアイコン → `Settings...` を選択

以降、各タブごとに現PCの設定値を一覧します。項目名は英語表記（Docker Desktop の日本語ローカライズは未対応のため）です。

#### 7.2.1 General タブ

- Start Docker Desktop when you sign in to your computer: 有効（ログイン時に自動起動）
- Theme: `Use system settings`
- Enable Docker terminal: 無効（Desktop 内蔵ターミナルは使わない）
- Send usage statistics: 無効（テレメトリオフ）
- Use Resource Saver: 有効（アイドル時に自動一時停止、しきい値300秒）
- SBOM indexing: 有効

#### 7.2.2 Resources → Advanced タブ

PCのスペックに応じて調整してください。

| 項目               | 値       |
| ------------------ | -------- |
| CPU limit          | 12       |
| Memory limit       | 32.00 GB |
| Swap               | 4 GB     |
| Virtual disk limit | 256 GB   |

#### 7.2.3 Software updates タブ

- すべて無効にする。

#### 7.2.4 Notifications タブ

- すべて無効にする。

---

## 8. Git / GitHub

### 8.1 インストール

Git は Homebrew 経由で導入する。macOS の Xcode Command Line Tools に付属する Git より新しいバージョンを使いたいため、Homebrew 版を優先する。

```bash
brew install git
```

`which git` が `/opt/homebrew/bin/git` を返せば導入完了。

もし `/usr/bin/git` のままになっている場合は、シェルのコマンドハッシュキャッシュが古い可能性がある。以下のいずれかで解消する。

```bash
# zsh のコマンドハッシュを再構築
hash -r

# または現在のシェルで .zprofile を再読み込み
source ~/.zprofile

# または新しいターミナルを開き直す
```

それでも解消しない場合は、`.zprofile` に `eval "$(/opt/homebrew/bin/brew shellenv)"` が含まれているかを確認する。含まれていない場合はセクション2.1のHomebrewインストール時のPATH設定を実施すること。

#### 8.1.1 Git LFS

バイナリや大容量ファイルを管理しているリポジトリ（モデルファイル、画像・動画素材、データセット等）で必要となる。`~/.gitconfig` に `[filter "lfs"]` セクションが含まれているため、本体のインストールも併せて行う。

```bash
brew install git-lfs

# 現在のユーザに対して LFS フィルタを登録（.gitconfig に既存の設定があれば上書きはされない）
git lfs install
```

クローン済みリポジトリで LFS ファイルが正しく取得されていない場合は、リポジトリ内で `git lfs pull` を実行する。

### 8.2 GitHub CLI

```bash
brew install gh
gh auth login
```

- アカウント: `<your-github-account>`
- Git操作プロトコル: SSH
- スコープ: gist, read:org, repo

---

## 9. AWS CLI

AWS CLI のセットアップは以下の流れで行う。

1. AWS CLI 本体と Session Manager Plugin を Homebrew で導入
2. 旧MacBookから `~/.aws/config` と `~/.aws/credentials` を配置（詳細はセクション17）
3. SSO プロファイルのみログイン初期化
4. 動作確認

### 9.1 インストール

#### 9.1.1 AWS CLI 本体（AWS 公式インストーラー）

AWS 公式サイトの手順に従ってインストールする: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

インストール後に `aws --version` が通ることを確認する。

#### 9.1.2 Session Manager Plugin（Homebrew Cask）

EC2 への SSM 接続に必要。こちらは Homebrew で問題ない。

```bash
brew install --cask session-manager-plugin
```

インストール確認:

```bash
session-manager-plugin --version
```

### 9.2 認証情報の配置

旧MacBook から以下を引き継ぐ（実ファイル転送はセクション17）。

- `~/.aws/config` — プロファイル定義（リージョン、SSO設定、ロール等）
- `~/.aws/credentials` — アクセスキーID / シークレットキー（機密情報のため暗号化推奨）

### 9.3 プロファイル一覧（参考）

プロファイルの棚卸し、認証方式ごとに分かれている。

IAM ユーザーのアクセスキーを直接使うもの（`~/.aws/credentials` に `[プロファイル名]` で定義）

AWS IAM Identity Center (SSO) を使うもの（`~/.aws/config` に `[profile 名]` と `sso_session` を定義）

### 9.4 SSO ログイン

SSO ベースのプロファイルを使う場合は、初回利用時にブラウザ認証が必要。プロファイル単位でログインする。

```bash
# 例: <sso-profile> プロファイルでログイン
aws sso login --profile <sso-profile>
```

ブラウザが開くので SSO アカウントで承認すると、トークンが `~/.aws/sso/cache/` に保存される。有効期限が切れたら同じコマンドで再ログインする。credentials ベースのプロファイル（IAM ユーザーのアクセスキー）はこの手順は不要。

### 9.5 動作確認

プロファイルを指定して呼び出し元アカウントを確認する。エラーにならず `Account` と `Arn` が返ればセットアップ完了。

```bash
# credentials ベース
aws sts get-caller-identity --profile <your-profile>

# SSO ベース
aws sts get-caller-identity --profile <sso-profile>
```

---

## 10. Terraform関連

### 10.1 tfenv

```bash
brew install tfenv
```

必要なTerraformバージョンは各プロジェクトの `.terraform-version` に記載されているため、プロジェクトディレクトリで `tfenv install` を実行すること。

### 10.2 tflint

```bash
brew install tflint
```

---

## 11. Visual Studio Code

### 11.1 設定ファイルの復元

現PCで使用している設定を以下のパスに配置します。内容はそのままコピーで構いません。

- `~/Library/Application Support/Code/User/settings.json`
- `~/Library/Application Support/Code/User/keybindings.json`

#### 11.1.1 settings.json

VSCode のユーザー設定を開く方法は二つあります。

- Finder で直接 `~/Library/Application Support/Code/User/settings.json` を開いて編集する
- VSCode 起動後にコマンドパレット (`⌘ + Shift + P`) → `Preferences: Open User Settings (JSON)`

以下の内容をそのまま貼り付けてください。フォント設定（セクション4）、ターミナル既定プロファイル（fish）、cSpell のカスタム辞書、Claude Code のパネル表示などが含まれています。

```json
{
  "terminal.integrated.defaultProfile.osx": "fish",
  "terminal.integrated.profiles.osx": {
    "fish": {
      "path": "/opt/homebrew/bin/fish"
      // "args": ["--no-config"]
    }
  },
  "files.autoSave": "afterDelay",
  "editor.fontSize": 14,
  "editor.fontFamily": "'Hack Nerd Font Mono', 'Noto Sans Mono CJK JP', monospace",
  "explorer.confirmDelete": false,
  "explorer.confirmDragAndDrop": false,
  "security.workspace.trust.untrustedFiles": "open",
  "git.openRepositoryInParentFolders": "never",
  "editor.accessibilityPageSize": 14,
  "editor.wordWrap": "on",
  "ruff.configurationPreference": "filesystemFirst",
  "workbench.panel.defaultLocation": "right",
  "docker.extension.enableComposeLanguageServer": false,
  "editor.minimap.sectionHeaderLetterSpacing": 0,
  "editor.codeLensFontSize": 1,
  "scm.inputFontSize": 12,
  "terminal.integrated.fontSize": 14,
  "terminal.integrated.fontFamily": "'Hack Nerd Font Mono', 'Noto Sans Mono CJK JP', monospace",
  // ターミナル速度改善
  "terminal.integrated.shellIntegration.enabled": false,
  "terminal.integrated.inheritEnv": true,
  "accessibility.signals.terminalBell": {
    "sound": "on"
  },
  "zenMode.silentNotifications": false,
  "workbench.secondarySideBar.defaultVisibility": "hidden",
  "workbench.editorAssociations": {
    "*.csv": "default"
  },
  "[dockercompose]": {
    "editor.insertSpaces": true,
    "editor.tabSize": 2,
    "editor.autoIndent": "advanced",
    "editor.quickSuggestions": {
      "other": true,
      "comments": false,
      "strings": true
    },
    "editor.defaultFormatter": "redhat.vscode-yaml"
  },
  "[github-actions-workflow]": {
    "editor.defaultFormatter": "redhat.vscode-yaml"
  },
  "markdown-pdf.footerTemplate": " <div style=\"font-size: 9px; margin: 0 auto;\"> <span class='pageNumber'></span> / <span class='totalPages'></span></div>",
  "markdown-pdf.headerTemplate": "<div></div>",
  "markdown-pdf.margin.bottom": "2cm",
  "claudeCode.preferredLocation": "panel",
  "[dynamic csv]": {
    "editor.inlayHints.maximumLength": 0
  }
}
```

主な設定ポイント:

- デフォルトターミナル: fish (`/opt/homebrew/bin/fish`)
- フォント: `Hack Nerd Font Mono` + `Noto Sans Mono CJK JP`（エディタ・ターミナル共通、サイズ14）
- ファイル自動保存: afterDelay
- エディタのワードラップ: on
- パネル位置: 右
- ターミナル速度改善: `shellIntegration.enabled = false`
- Ruff: filesystemFirst（プロジェクト設定を優先）
- markdown-pdf のヘッダー/フッター設定
- `[dockercompose]` と `[github-actions-workflow]` のフォーマッタを `redhat.vscode-yaml` に固定

#### 11.1.2 keybindings.json

コマンドパレット → `Preferences: Open Keyboard Shortcuts (JSON)` で開き、以下をそのまま貼り付けます。

```json
[
  {
    "key": "shift+enter",
    "command": "workbench.action.terminal.sendSequence",
    "args": {
      "text": "\\\r\n"
    },
    "when": "terminalFocus"
  }
]
```

- `Shift+Enter` (ターミナルフォーカス時): バックスラッシュ+改行を送信（長いコマンドを複数行に分けて実行するため）

### 11.2 自動保存の有効化

VSCode の自動保存は `settings.json` の `files.autoSave` で制御する。11.1 の設定をそのまま貼り付ければ `afterDelay` で有効化される。GUI から切り替える場合は以下。

1. メニューバー「File」→「Auto Save」をクリック（チェックが付いた状態で有効）
2. または コマンドパレット (`⌘ + Shift + P`) → `File: Toggle Auto Save`

より細かい挙動を指定したい場合は `settings.json` で以下のキーを編集する。

```json
{
  "files.autoSave": "afterDelay",
  "files.autoSaveDelay": 1000
}
```

`files.autoSave` に設定できる値:

- `off`: 自動保存しない
- `afterDelay`: 入力停止から `files.autoSaveDelay` ミリ秒後に保存（現設定: 1000ms）
- `onFocusChange`: エディタのフォーカスが外れたら保存
- `onWindowChange`: VSCode ウィンドウからフォーカスが外れたら保存

### 11.3 拡張機能のインストール (参考)

```bash
code --install-extension anthropic.claude-code
code --install-extension charliermarsh.ruff
code --install-extension docker.docker
code --install-extension github.vscode-github-actions
code --install-extension golang.go
code --install-extension hashicorp.terraform
code --install-extension mechatroner.rainbow-csv
code --install-extension ms-python.debugpy
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-python.vscode-python-envs
code --install-extension ms-vscode-remote.remote-containers
code --install-extension ms-vscode-remote.remote-ssh
code --install-extension ms-vscode-remote.remote-ssh-edit
code --install-extension ms-vscode.remote-explorer
code --install-extension redhat.vscode-yaml
code --install-extension streetsidesoftware.code-spell-checker
code --install-extension tomoki1207.pdf
code --install-extension yzane.markdown-pdf
```

---

## 12. Claude Code

### 12.1 インストール

公式ではネイティブインストーラー経由の導入が推奨されている。Node.js や npm のグローバル環境に依存せず、自動アップデートにも対応するため、基本的にはこちらを使用する。

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

インストール後、バイナリは `~/.local/bin/claude` に配置される。

### 12.2 PATH を通す

`~/.local/bin` はデフォルトの `$PATH` に含まれていないため、手動で通す必要がある。`~/.zprofile` に以下を追記する。

```bash
# ~/.zprofile に追記
export PATH="$HOME/.local/bin:$PATH"
```

設定を現在のシェルに反映する。

```bash
source ~/.zprofile
```

もしくは新しいターミナルを開き直す。どちらでもOK。

### 12.3 動作確認

```bash
# インストール先と PATH 解決を確認
which claude                # → /Users/<user>/.local/bin/claude
claude --version            # → 何らかのバージョン番号が表示される
```

`claude: command not found` が出る場合は、12.2 の PATH 追記が `.zprofile` に反映されていないか、シェルのリロードが済んでいない。以下を順に試す。

```bash
# PATH に ~/.local/bin が含まれているか
echo $PATH | tr ':' '\n' | grep "$HOME/.local/bin"

# コマンドハッシュキャッシュをクリア
hash -r

# .zprofile を再読み込み
source ~/.zprofile
```

### 12.4 データの復元

会話履歴を含む全データを引き継ぐため、`~/.claude/` ディレクトリを丸ごとコピーする。g

```bash
# 旧MacBookからAirDrop/USBメモリ等で転送した後、新MacBookで配置
cp -a /path/to/backup/.claude ~/
```

主な内容物:

| ファイル/ディレクトリ | 内容                                              |
| --------------------- | ------------------------------------------------- |
| `CLAUDE.md`           | グローバルプロンプト                              |
| `settings.json`       | 権限設定、プラグイン、言語設定 (日本語)、音声有効 |
| `settings.local.json` | フック設定 (model: sonnet, env変数)               |
| `hooks/`              | フック設定ファイル                                |
| `skills/`             | カスタムスキル一式                                |
| `projects/`           | プロジェクト別設定・メモリ                        |
| `history.jsonl`       | 会話履歴                                          |
| `plugins/`            | インストール済みプラグイン                        |
| `plans/`              | 作成済みプラン                                    |
| `todos/`              | Todoデータ                                        |
| `tasks/`              | タスクデータ                                      |
| `file-history/`       | ファイル変更履歴                                  |
| `sessions/`           | セッションデータ                                  |

丸ごとコピーすることでプラグインの再インストールも不要になる。

### 12.6 hooks設定

`~/.claude/hooks/settings.json` の内容:

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "30000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "85",
    "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet"
  }
}
```

### 12.7 プラグイン

現PCで有効化されているプラグインは以下の4つ。公式マーケットプレイス (`claude-plugins-official`) はデフォルトで登録されているが、サードパーティのマーケットプレイスは新PC側で追加登録する必要がある。

| プラグイン         | マーケットプレイス             | リポジトリ                                         |
| ------------------ | ------------------------------ | -------------------------------------------------- |
| `coderabbit`       | `claude-plugins-official`      | `anthropics/claude-plugins-official`（デフォルト） |
| `visual-explainer` | `visual-explainer-marketplace` | `nicobailon/visual-explainer`                      |
| `codex`            | `openai-codex`                 | `openai/codex-plugin-cc`                           |

Claude Code を起動し、スラッシュコマンドで順に実行する。

```
# サードパーティマーケットプレイスを追加
/plugin marketplace add nicobailon/visual-explainer
/plugin marketplace add openai/codex-plugin-cc

# プラグインをインストール
/plugin install coderabbit@claude-plugins-official
/plugin install visual-explainer@visual-explainer-marketplace
/plugin install codex@openai-codex
```

セクション17で `~/.claude/settings.json` を引き継ぐと `enabledPlugins` / `extraKnownMarketplaces` の設定もコピーされるため、Claude Code 初回起動時にプラグイン本体が自動で取得される。明示的に `/plugin install` を打つ必要があるのは、設定ファイルを引き継がずに新PCで一から構築する場合のみ。

インストール状況は以下で確認できる。

```
/plugin list
```

---

## 13. Codex CLI (OpenAI)

OpenAI の Codex CLI をローカルに導入し、Claude Code と併用する構成です。

### 13.1 インストール

Homebrew Cask 経由でインストールします。

```bash
brew install --cask codex
```

インストール後、`codex --version` でバージョン（例: `codex-cli 0.114.0`）が表示されれば導入完了です。バイナリは `/opt/homebrew/bin/codex` として提供されます。アップデートは `brew upgrade --cask codex` で行います。

### 13.2 初回起動と認証

初回起動時に ChatGPT アカウントへのサインインを求められます。ブラウザが開くので、OpenAI アカウントで認証を行うと、認証トークンが `~/.codex/auth.json` に保存されます。

```bash
codex login
```

`auth.json` は機密情報を含むため、旧Macからコピーする場合は暗号化して転送するか、新Macで再ログインすることを推奨します。

### 13.3 主な設定値（参考）

`~/.codex/config.toml` の冒頭で以下を指定しています。

```toml
model = "gpt-5.5"
model_reasoning_effort = "high"
personality = "pragmatic"
```

プロジェクト単位の信頼設定は `[projects."..."]` セクションで個別に管理されます。新Macでは作業ディレクトリのパスが変わるため、必要に応じて `trust_level = "trusted"` を再指定してください。

### 13.4 Claude Code との連携

Claude Code 側に codex プラグイン (`codex@openai-codex`) を導入すると、Claude Code から `Task` ツール経由で Codex を呼び出せます。プラグインの有効化手順はセクション12.7 を参照してください。

---

## 14. その他のツール

### 14.1 ngrok

ローカルで動作しているサーバーを一時的に外部公開するためのトンネリングツール。Webhook の開発やモバイル端末からのテスト時に使用する。

#### 14.1.1 インストール

```bash
brew install --cask ngrok
```

#### 14.1.2 authtoken の登録

ngrok は起動時に authtoken で認証する。トークンは ngrok のダッシュボードから取得する。

1. https://dashboard.ngrok.com/get-started/your-authtoken にブラウザでアクセスする（アカウントが無い場合はサインアップ）
2. `Your Authtoken` の値をコピーする
3. ターミナルで以下を実行

```bash
ngrok config add-authtoken <コピーしたトークン>
```

実行すると `~/Library/Application Support/ngrok/ngrok.yml` に authtoken が保存される。

#### 14.1.3 ドメインの登録

Pay-as-you-go プランでは、事前にドメインを登録しないとトンネルを開始できない（`ERR_NGROK_15002`）。

1. https://dashboard.ngrok.com/domains にアクセスする
2. 無料のドメインを1つ登録する（`xxxx.ngrok-free.app` 形式で発行される）

#### 14.1.4 動作確認

起動時に `--url` で登録済みドメインを指定する。

```bash
ngrok http --url=your-domain.ngrok-free.app 8080
```

ターミナルに `Forwarding` の URL が表示されれば成功。`Ctrl + C` で停止。

### 14.2 go-task

`Taskfile.yml` を起点としたタスクランナー。`make` の代替として、ビルド・テスト・デプロイ等の定型コマンドをプロジェクトごとに `task <タスク名>` で呼び出すために使用する。

```bash
brew install go-task
```

動作確認は `task --version` で行う。プロジェクト直下に `Taskfile.yml` がある場合、`task --list` で利用可能なタスクを一覧できる。

---

## 15. シェル環境 (zsh)

本節は旧MacBookの `~/.zprofile` / `~/.zshrc` を引き継ぐ前提のため、セクション17のファイル引き継ぎと併せて実施してください。

### 15.1 zshを使用する

macOS のデフォルトシェルは `zsh` のため、通常はシェル変更は不要。

```bash
echo $SHELL
```

期待値は `/bin/zsh`。

### 15.2 `.zprofile` の設定

`~/.zprofile` を作成または復元し、ログインシェルで必要なPATH初期化を記述する。

主な設定内容は以下の通り。

- Homebrew パス (`/opt/homebrew/bin`)
- `~/.local/bin` パス（uv、Claude Code ネイティブインストール版などユーザーローカルバイナリ用）
- Go バイナリパス (`~/go/bin`)

例:

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"

# uv、Claude Code（ネイティブ版）、その他ユーザーローカルバイナリを通す
export PATH="$HOME/.local/bin:$HOME/go/bin:$PATH"
```

### 15.3 補完プラグインのインストール

素の zsh は Tab 補完が弱くインライン予測も効かないため、fish 相当の体験を得るために2つのプラグインを Homebrew で導入する。

```bash
brew install zsh-autosuggestions zsh-syntax-highlighting
```

| プラグイン                | 役割                                                           |
| ------------------------- | -------------------------------------------------------------- |
| `zsh-autosuggestions`     | 過去コマンドから灰色でインライン予測を表示。→キー / End で受諾 |
| `zsh-syntax-highlighting` | 入力中のコマンドをリアルタイムで色付け（有効=緑 / 無効=赤）    |

加えて、zsh 標準の `compinit` を有効化すると Tab 2連打でメニュー式候補が出るようになる。

- `autoload -Uz compinit`: `compinit` 関数を遅延ロード形式で宣言する（`-U` でエイリアス展開無効、`-z` で zsh スタイル強制）
- `compinit`: 宣言した関数を実際に呼び出し、補完定義を読み込む

初回起動時に `~/.zcompdump` というキャッシュファイルが生成される。補完定義を追加・変更した場合は `rm ~/.zcompdump` で削除してから zsh を起動し直すと再生成される。

次項の `.zshrc` 設定例には既にこの1行が含まれているため、そのまま貼り付ければ有効になる。

### 15.4 `.zshrc` の設定

`~/.zshrc` を作成または復元し、対話シェル用の初期化を記述する。

主な設定内容は以下の通り。

- プロンプトからホスト名（AirDrop表示名）を除外
- zsh 標準補完 (`autoload -Uz compinit && compinit`)
- `zsh-autosuggestions` / `zsh-syntax-highlighting` の読み込み
- Go の GOPATH 設定（GOROOT は `go` バイナリが自動認識するため明示不要）
- AWS_REGION デフォルト (`ap-northeast-1`)
- `mise activate zsh`

zsh デフォルトのプロンプト `%n@%m %1~ %#` は `@%m` 部分に `LocalHostName`（AirDrop表示名）が出てしまうため、`@%m` を除外した形式に上書きしておきます。

例:

```bash
# プロンプトからホスト名（AirDrop表示名）を除外
PROMPT='%n %1~ %# '

# Tab補完の基本（Tab 2連打でメニュー式候補表示）
autoload -Uz compinit && compinit

# fish風のインライン予測（過去コマンドから灰色でヒント表示、→キー/Endで受諾）
source /opt/homebrew/share/zsh-autosuggestions/zsh-autosuggestions.zsh

# コマンドをリアルタイムで色付け（有効=緑/無効=赤）※最後に読み込むこと
source /opt/homebrew/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

command -v mise >/dev/null 2>&1 && eval "$(mise activate zsh)"
export AWS_REGION=ap-northeast-1
export GOTOOLCHAIN=auto
export GOPATH="$HOME/go"
```

`zsh-syntax-highlighting` は他のプラグインの発火タイミングに影響を受けるため、必ず `.zshrc` の最後に読み込むこと。

### 15.5 設定の反映

```bash
source ~/.zprofile
source ~/.zshrc
```

---

## 16. SSH鍵と設定

本節は旧MacBookの `~/.ssh/` を引き継ぐ前提のため、セクション17のファイル引き継ぎと併せて実施してください。

### 16.1 鍵ファイルの復元

`~/.ssh/` ディレクトリをバックアップから復元する。

### 16.2 SSH config の復元

`~/.ssh/config` をバックアップから復元する。設定されているホスト:

### 16.3 パーミッションの設定

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/*
```

---

## 17. 旧MacBookからの設定ファイル引き継ぎ

ここまでの環境構築が一通り終わった段階で、旧MacBookから以下のファイル/ディレクトリを USBメモリ、AirDrop、クラウドストレージ等でコピーし、新MacBookの同じパスに配置する。機密情報を含むものは暗号化して転送すること。

| 対象                | パス                                                       | 備考                                                                                                 |
| ------------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| SSH鍵と設定         | `~/.ssh/`                                                  | 秘密鍵を含むため暗号化して転送することを推奨（セクション16）                                         |
| zsh設定             | `~/.zprofile`, `~/.zshrc`                                  | Homebrew PATH, mise, 補完設定など（セクション15）                                                    |
| Git設定             | `~/.gitconfig`                                             | user.name / user.email / LFS フィルタ等（セクション8）                                               |
| AWS設定             | `~/.aws/config`, `~/.aws/credentials`                      | credentials は機密情報のため暗号化推奨（セクション9）                                                |
| Ruff設定            | `~/.config/ruff.toml`                                      | 存在する場合                                                                                         |
| VSCode設定          | `~/Library/Application Support/Code/User/settings.json`    | ガイド内にインライン記載もあり（セクション11）                                                       |
| VSCodeキーバインド  | `~/Library/Application Support/Code/User/keybindings.json` | 同上                                                                                                 |
| Claude Code全データ | `~/.claude/`                                               | ディレクトリ丸ごとコピー。会話履歴、スキル、プラグイン、プロジェクト設定等を全て含む（セクション12） |
| Codex全データ       | `~/.codex/`                                                | ディレクトリ丸ごとコピー。auth.json は機密情報のため暗号化推奨（セクション13）                       |
| Karabiner設定       | `~/.config/karabiner/karabiner.json`                       | セクション3参照                                                                                      |
| Goバイナリ          | `~/go/bin/`                                                | コピー不要。新PCで `go install` で再取得する                                                         |

配置後、セクション15（zsh）とセクション16（SSH）の手順で反映とパーミッション設定を行う。

---

## チェックリスト

セットアップ完了後にセクション順で確認する。

### 0. 初期セットアップ

- [ ] Appleアカウントにサインインし「Macを探す」が有効になっている
- [ ] Touch ID で指紋を登録済み
- [ ] Mac 名（AirDrop表示名）を変更済み

### 1. macOS システム設定

- [ ] ディスプレイ/システム/ディスクのスリープが全て無効、スクリーンセーバーも無効 (`pmset -g` で確認)
- [ ] メニューバーに サウンド / Bluetooth / 集中モード / 入力ソース / バッテリー（数値付き）が表示されている
- [ ] フルスクリーンモードでもメニューバーが常時表示されている（「メニューバーを自動的に表示/非表示」が「しない」）
- [ ] トラックパッドの操作感が旧MacBookと同等
- [ ] キーのリピートが最短、音声入力ショートカットが `Control 2回押し` に設定されている
- [ ] HHKB が JIS/ANSI 判定済みで正常動作
- [ ] 入力ソース編集画面で「日本語 - ローマ字入力」の `ライブ変換` と `タイプミスを修正` がオフになっている
- [ ] Finder で隠しファイル/拡張子が表示、既定の表示形式がカラム表示、デスクトップアイコンがグリッドに整列
- [ ] ログイン項目に Chrome / Slack / Docker / GitHub Desktop / cmux が登録されている

### 2. Homebrew

- [ ] `brew --version` が通る
- [ ] `/opt/homebrew/bin` が `$PATH` の先頭に入っている

### 3. Karabiner-Elements

- [ ] カスタムルール（Cmd英数/かな切り替え、左Option単押しでバッククォート等）が動作している

### 4. フォント & cmux

- [ ] `Hack Nerd Font Mono` と `Noto Sans Mono CJK JP` がインストール済み
- [ ] cmux が起動し、垂直タブやペイン分割が動作している
- [ ] cmux で英数字と日本語の幅が揃っている（`~/.config/ghostty/config` のフォント設定反映）

### 5. GUIアプリケーション

- [ ] ブラウザ / Slack / Teams / Zoom など常用アプリが導入済み
- [ ] Raycast が起動し、設定がエクスポート/インポートで復元されている

### 6. プログラミング言語とランタイム

- [ ] Python (`uv python list --only-installed` でバージョンが揃っている)、`uv tool install ruff` 済み
- [ ] Go (`go version` が通る)、`GOTOOLCHAIN=auto` が設定済み、`$GOPATH/bin` の開発ツールが揃っている
- [ ] Node.js (mise 経由で `node@20.8.1` がグローバル)

### 7. Docker

- [ ] Docker Desktop が起動し、コンテナを実行できる
- [ ] リソース割り当て（CPU 12 / Memory 32GiB / Disk 256GiB）が GUI から設定済み
- [ ] Software updates / Notifications タブで通知類が全て無効

### 8. Git

- [ ] `which git` が `/opt/homebrew/bin/git` を返す（Homebrew版）
- [ ] `git lfs version` が応答する（`git-lfs` インストール済み）
- [ ] `gh auth login` 済み (`<your-github-account>` アカウント)
- [ ] `~/.gitconfig` が復元されている

### 9. AWS CLI

- [ ] `aws --version` が公式版（`/usr/local/bin/aws` 経由）で応答する
- [ ] `session-manager-plugin --version` が通る
- [ ] `aws sts get-caller-identity --profile <名前>` が credentials / SSO どちらでも成功

### 10. Terraform

- [ ] `tfenv install` が動作する
- [ ] `tflint` / `infracost` が導入済み

### 11. Visual Studio Code

- [ ] ユーザー設定 `settings.json` / `keybindings.json` が貼り付け済み
- [ ] 既定ターミナルが fish で開く
- [ ] 自動保存（`files.autoSave: afterDelay`）が動作
- [ ] パネル位置が右側
- [ ] 必要な拡張機能が全てインストールされている

### 12. Claude Code

- [ ] `claude --version` が通り `~/.local/bin/claude` に解決される
- [ ] `~/.claude/` の設定（CLAUDE.md, settings.json, hooks/, skills/）が復元済み
- [ ] プラグイン `coderabbit` / `visual-explainer` / `codex` が `/plugin list` で有効
- [ ] `CLAUDE_CODE_SUBAGENT_MODEL` が `sonnet` になっている

### 13. Codex CLI

- [ ] `codex --version` が通る
- [ ] `codex login` 済みで `~/.codex/auth.json` が配置されている

### 14. その他のツール

- [ ] `mise` が動作している
- [ ] `ngrok` が `authtoken` 設定済み
- [ ] `task --version` が応答する（`go-task` インストール済み）

### 15. シェル環境 (zsh)

- [ ] `echo $SHELL` が `/bin/zsh`
- [ ] Tab 2連打でメニュー式候補が出る（`compinit` 有効）
- [ ] 過去コマンドから灰色でインライン予測が表示される（`zsh-autosuggestions`）
- [ ] コマンド入力中に有効/無効が色で判別できる（`zsh-syntax-highlighting`）
- [ ] プロンプトにホスト名（AirDrop表示名）が出ない

### 16. SSH

- [ ] SSH鍵で GitHub に接続できる (`ssh -T git@github.com`)
- [ ] 社内サーバー (`<internal-host>`) にSSH接続できる
- [ ] `~/.ssh/` のパーミッションが 700、秘密鍵が 600

### 17. 旧MacBookからの引き継ぎ

- [ ] `~/.ssh/`, `~/.zprofile`, `~/.zshrc`, `~/.gitconfig`, `~/.aws/`, `~/.claude/`, `~/.codex/`, `~/.config/karabiner/` が配置済み
- [ ] VSCode 設定ファイル、Raycast エクスポートも反映済み

---

## 18. ユーザーデータの移行

環境構築とは別に、MacBookへ移す必要があるファイル・ディレクトリは、有線、外付けSSD 等で転送する。
