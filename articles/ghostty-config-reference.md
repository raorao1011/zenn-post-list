---
title: "Ghostty設定リファレンス完全ガイド"
emoji: "👻"
type: "tech"
topics: ["ghostty", "terminal", "設定", "ターミナルエミュレータ"]
published: false
---

## はじめに

この記事は、GPU加速ターミナルエミュレータ[Ghostty](https://ghostty.org/)の公式設定リファレンスを日本語でまとめたものです。全ての設定オプションを網羅的に解説しています。

設定ファイルは通常 `~/.config/ghostty/config` に配置します。

## フォント関連設定

### font-family / font-family-bold / font-family-italic / font-family-bold-italic

ターミナルで使用するフォントファミリーを指定します。複数指定で代替フォントをサポートします。

- **デフォルト**: システムデフォルト
- **形式**: フォント名（複数指定時はカンマ区切り）

```
font-family = "JetBrains Mono"
font-family-bold = "JetBrains Mono Bold"
```

利用可能なフォント一覧は `ghostty +list-fonts` コマンドで確認できます。

### font-size

フォントサイズをポイント単位で指定します。小数点での指定も可能です。

- **デフォルト**: 12pt程度
- **形式**: 数値（例：13.5）
- **注意**: 変更は新規ターミナルにのみ反映されます

```
font-size = 14
```

### font-style / font-style-bold / font-style-italic / font-style-bold-italic

フォントスタイルを指定します。スタイル無効化も可能です。

- **可能な値**: フォント固有のスタイル名、または `false`
- **デフォルト**: 自動検出

```
font-style = false
```

### font-synthetic-style

提供されないスタイルの自動合成を制御します。

- **可能な値**: `true`、`false`、`no-bold`、`no-italic`、`no-bold-italic`
- **デフォルト**: `true`

### font-feature

OpenType機能を有効化します。複数指定可能です。

- **形式**: `feat`、`+feat`、`-feat`、`feat=value`など

```
font-feature = -calt  # プログラミングリガチャを無効化
```

### font-variation

可変フォント（Variable Font）の軸を調整します。

- **形式**: `id=value`（例：wght=600）
- **可能な軸**: wght、slnt、ital、opsz、wdth、GRAD等

```
font-variation = wght=500
```

### font-codepoint-map

特定のUnicodeコードポイントを別フォントにマッピングします。

- **形式**: `U+ABCD=fontname` または範囲指定

```
font-codepoint-map = U+2713=Noto Color Emoji
```

### font-thicken / font-thicken-strength

フォントの太さを調整します（macOS限定）。

- **thicken**: `true`/`false`
- **strength**: 0～255の整数

```
font-thicken = true
font-thicken-strength = 50
```

### font-shaping-break

フォント成形の分割を制御します。

- **可能な値**: `cursor`
- **利用可能**: 1.2.0以降

## フォント調整設定

### adjust-cell-width / adjust-cell-height

セルサイズをピクセルまたはパーセンテージで調整します。

- **形式**: 整数、パーセンテージ（例：20%）
- **注意**: 値は追加的なもの（絶対値ではない）

```
adjust-cell-width = 10%
adjust-cell-height = 5
```

### adjust-font-baseline

ベースラインの位置を調整します（下からの距離）。

```
adjust-font-baseline = 2
```

### adjust-underline-position / adjust-underline-thickness

アンダーラインの位置と太さを調整します。

```
adjust-underline-position = 1
adjust-underline-thickness = 2
```

### adjust-strikethrough-position / adjust-strikethrough-thickness

打消し線の位置と太さを調整します。

### adjust-overline-position / adjust-overline-thickness

オーバーラインの位置と太さを調整します。

### adjust-cursor-thickness / adjust-cursor-height

カーソルの太さと高さを調整します。

```
adjust-cursor-thickness = 3
adjust-cursor-height = 100%
```

### adjust-box-thickness

枠線描画文字の太さを調整します。

### adjust-icon-height

Nerd Fontアイコンの最大高さを調整します。

- **利用可能**: 1.2.0以降

## 色・表示設定

### background / foreground

ウィンドウの背景色と前景色を指定します。

- **形式**: 16進数（#RRGGBB）または名前付きX11色

```
background = #1e1e1e
foreground = #d4d4d4
```

### theme

組み込みまたはカスタムテーマを指定します。

- **形式**: テーマ名またはファイルパス
- **ライト/ダーク対応**: `light:theme-name,dark:theme-name`

```
theme = dark:nord,light:github
```

テーマファイルは `~/.config/ghostty/themes/` ディレクトリに配置します。

### palette

256色パレットを設定します。

- **形式**: `N=COLOR`（N：0～255）
- **インデックス形式**: 10進数、2進数（0b）、8進数（0o）、16進数（0x）

```
palette = 0=#000000
palette = 1=#ff0000
```

### cursor-color / cursor-opacity / cursor-text

カーソルの色、透明度、テキスト色を指定します。

- **cursor-color**: 特殊値 `cell-foreground`/`cell-background` 対応（1.2.0～）
- **cursor-opacity**: 0～1の値

```
cursor-color = #ff0000
cursor-opacity = 0.8
cursor-text = #ffffff
```

### cursor-style

カーソルのスタイルを指定します。

- **可能な値**: `block`、`bar`、`underline`、`block_hollow`
- **デフォルト**: システム依存

```
cursor-style = bar
```

### cursor-style-blink

カーソルの点滅を制御します。

- **可能な値**: `true`、`false`、空（AT&Tモード対応）
- **デフォルト**: `true`

```
cursor-style-blink = false
```

### selection-foreground / selection-background

選択領域の前景色と背景色を指定します。

- **特殊値**: `cell-foreground`、`cell-background`（1.2.0～）

```
selection-background = #264f78
selection-foreground = cell-foreground
```

### selection-clear-on-typing / selection-clear-on-copy

タイピング時やコピー時に選択範囲をクリアするかを指定します。

- **デフォルト**: `clear-on-typing=true`、`clear-on-copy=false`

### minimum-contrast

最小コントラスト比を指定します（WCAG 2.0基準）。

- **範囲**: 1～21
- **推奨**: 1.1（視認性）、3以上（読みやすさ）

```
minimum-contrast = 1.1
```

### bold-color

太字テキストの色を制御します。

- **可能な値**: 色コード、または `bright`
- **利用可能**: 1.2.0以降

```
bold-color = bright
```

### faint-opacity

薄い色（faint）テキストの透明度を指定します。

- **利用可能**: 1.2.0以降

## 背景・透明度設定

### background-opacity

ウィンドウ背景の透明度を指定します。

- **範囲**: 0～1
- **注意**: macOS全画面モード時は無効

```
background-opacity = 0.95
```

### background-opacity-cells

明示的背景色セルにも透明度を適用するかを指定します。

- **デフォルト**: `false`
- **利用可能**: 1.2.0以降

### background-image / background-image-opacity / background-image-position / background-image-fit / background-image-repeat

背景画像を設定します。

- **形式**: PNGまたはJPEGファイルパス
- **fit値**: `contain`、`cover`、`stretch`、`none`
- **position値**: `top-left`～`bottom-right`
- **repeat**: `true`/`false`
- **利用可能**: 1.2.0以降
- **注意**: VRAMメモリが増加する可能性があります

```
background-image = /path/to/image.png
background-image-opacity = 0.3
background-image-fit = cover
background-image-position = center
background-image-repeat = false
```

### background-blur

背景のぼかしを設定します（opacity < 1時）。

- **可能な値**: `false`、`true`、0～255のぼかし強度
- **対応**: macOS、KDE Plasma（強度は無視されます）

```
background-blur = 20
```

### unfocused-split-opacity / unfocused-split-fill

フォーカスされていないスプリットの暗さと色を指定します。

- **opacity範囲**: 0.15～1
- **fill**: 色コード指定

```
unfocused-split-opacity = 0.7
unfocused-split-fill = #000000
```

### split-divider-color

スプリット分割線の色を指定します。

- **利用可能**: 1.1.0以降

```
split-divider-color = #444444
```

## ウィンドウ設定

### window-height / window-width

初期ウィンドウサイズをグリッドセル数で指定します。

- **制限**: 10×4より大きい必要があります
- **注意**: ウィンドウマネージャが制限することもあります

```
window-height = 40
window-width = 120
```

### window-position-x / window-position-y

ウィンドウの開始位置をピクセルで指定します。

- **注意**: macOS限定。Waylandでは未サポート

```
window-position-x = 100
window-position-y = 100
```

### window-padding-x / window-padding-y

ウィンドウのパディングをポイント単位で指定します。

- **形式**: 単一値または左,右／上,下をコンマ区切り

```
window-padding-x = 10
window-padding-y = 10
# または
window-padding-x = 5,15
```

### window-padding-balance

余剰パディングを4方向で自動分散するかを指定します。

- **デフォルト**: `true`

### window-padding-color

パディング領域の色を指定します。

- **可能な値**: `background`、`extend`、`extend-always`

```
window-padding-color = extend
```

### window-decoration

ウィンドウ装飾を制御します。

- **可能な値**: `none`、`auto`、`client`、`server`
- **デフォルト**: `auto`
- **互換**: `true`=`auto`、`false`=`none`

```
window-decoration = none
```

### window-title-font-family

ウィンドウ・タブタイトル用のフォントを指定します。

- **利用可能**: 1.1.0以降（GTK）

### window-subtitle

ウィンドウのサブタイトルを制御します。

- **可能な値**: `false`、`working-directory`
- **利用可能**: 1.1.0以降（GTK）

```
window-subtitle = working-directory
```

### window-theme

ウィンドウのテーマを指定します。

- **可能な値**: `auto`、`system`、`light`、`dark`、`ghostty`
- **注意**: macOS `titlebar=tabs`で自動変更されます

```
window-theme = dark
```

### window-colorspace

色空間を指定します。

- **可能な値**: `srgb`、`display-p3`
- **デフォルト**: `srgb`
- **限定**: macOSのみ

```
window-colorspace = display-p3
```

### window-inherit-working-directory / window-inherit-font-size

新規ウィンドウが前のウィンドウ設定を継承するかを指定します。

- **デフォルト**: 両方 `true`

```
window-inherit-working-directory = true
window-inherit-font-size = true
```

### window-vsync

リフレッシュレートと同期するかを指定します。

- **デフォルト**: `true`（macOSカーネルパニック防止のため）
- **限定**: macOSのみ

### window-save-state

ウィンドウの状態を保存するかを指定します。

- **可能な値**: `default`、`never`、`always`
- **デフォルト**: `default`
- **限定**: macOSのみ

### window-step-resize

ウィンドウをセルサイズ単位でリサイズするかを指定します。

- **限定**: macOSのみ

### window-new-tab-position

新しいタブの位置を指定します。

- **可能な値**: `current`、`end`
- **デフォルト**: `current`

```
window-new-tab-position = end
```

### window-show-tab-bar

タブバーの表示を制御します。

- **可能な値**: `always`、`auto`、`never`
- **デフォルト**: `auto`
- **限定**: GTK（Linux）のみ

### window-titlebar-background / window-titlebar-foreground

GTKタイトルバーの色を指定します（ghosttyテーマ時）。

- **限定**: GTK Linuxのみ

### maximize / fullscreen

初期ウィンドウ状態を指定します。

- **デフォルト**: `false`
- **maximize利用可**: 1.1.0以降

```
maximize = true
```

### title

ウィンドウの固定タイトルを指定します。

- **注意**: 空白にするには引用符内にスペースを入れます

```
title = "My Terminal"
```

## マウス・入力設定

### mouse-hide-while-typing

タイピング時にマウスを非表示にするかを指定します。

- **デフォルト**: `false`

```
mouse-hide-while-typing = true
```

### mouse-shift-capture

Shift+マウスクリック検出のプロトコル送信を制御します。

- **可能な値**: `true`、`false`、`always`、`never`

### mouse-scroll-multiplier

マウススクロールの速度倍率を指定します。

- **範囲**: 0.01～10,000
- **デフォルト**: 3
- **利用可能**: 1.2.0以降

```
mouse-scroll-multiplier = 5
```

### scroll-to-bottom

スクロールを下部に戻すタイミングを指定します。

- **可能な値**: `keystroke`、`output`（コンマ区切り）
- **デフォルト**: `keystroke`、no-output
- **output**: 未実装

```
scroll-to-bottom = keystroke
```

### cursor-click-to-move

プロンプト位置でのカーソル移動（Alt+クリック）を有効化します。

- **要件**: シェル統合、プライマリスクリーン

```
cursor-click-to-move = true
```

### click-repeat-interval

クリック連続判定間隔をミリ秒で指定します。

- **デフォルト**: 0（OS設定）、一般的に500ms

```
click-repeat-interval = 500
```

### right-click-action

右クリック時のアクションを指定します。

- **可能な値**: `context-menu`、`paste`、`copy`、`copy-or-paste`、`ignore`
- **デフォルト**: `context-menu`

```
right-click-action = paste
```

## クリップボード設定

### clipboard-read / clipboard-write

OSC 52経由のクリップボード操作を許可するかを指定します。

- **可能な値**: `ask`、`allow`、`deny`
- **デフォルト**: `ask`（読）、`allow`（書）

```
clipboard-read = allow
clipboard-write = allow
```

### clipboard-trim-trailing-spaces

コピー時の末尾スペースを削除するかを指定します。

- **デフォルト**: `false`

```
clipboard-trim-trailing-spaces = true
```

### clipboard-paste-protection

危険な貼り付け内容で確認を要求するかを指定します。

```
clipboard-paste-protection = true
```

### clipboard-paste-bracketed-safe

括弧付き貼り付けを安全と見なすかを指定します。

- **デフォルト**: `true`

### copy-on-select

選択時に自動コピーするかを指定します。

- **可能な値**: `true`、`false`、`clipboard`
- **true**: 選択クリップボード優先
- **clipboard**: 両方にコピー

```
copy-on-select = clipboard
```

## コマンド・ターミナル設定

### command / initial-command

実行するコマンドを指定します。

- **command**: 毎回実行
- **initial-command**: 初回のみ実行
- **形式**: `direct:～`で直接実行、`shell:～`でシェル経由
- **デフォルト**: SHELL環境変数またはpasswdエントリ

```
command = zsh
initial-command = tmux attach || tmux new
```

### env

追加の環境変数を指定します。

- **形式**: `KEY=VALUE`
- **リセット**: 空の値で全削除

```
env = EDITOR=vim
env = LANG=ja_JP.UTF-8
```

### input

起動時の標準入力データを指定します。

- **形式**: `raw:<text>` または `path:<filepath>`
- **利用可能**: 1.2.0以降
- **制限**: 10MBまで

```
input = raw:echo "Hello, Ghostty!"
```

### wait-after-command

コマンド終了後もウィンドウを維持するかを指定します。

- **デフォルト**: `false`

```
wait-after-command = true
```

### abnormal-command-exit-runtime

異常終了と判定するミリ秒を指定します。

- **注意**: Linux=非ゼロ終了必須、macOS=任意

```
abnormal-command-exit-runtime = 1000
```

### scrollback-limit

スクロールバック容量をバイト単位で指定します。

- **注意**: 完全にメモリ内に蓄積されます

```
scrollback-limit = 10485760  # 10MB
```

## リンク・通知設定

### link / link-url / link-previews

URL/ファイルパスのクリック動作を制御します。

- **link-url**: `true`/`false`（デフォルト `true`）
- **link-previews**: `true`/`false`/`osc8`（1.2.0～）

```
link-url = true
link-previews = osc8
```

### title-report

タイトルクエリを許可するかを指定します（OSC 21）。

- **デフォルト**: `false`
- **警告**: セキュリティ・プライバシーリスクがあります

```
title-report = false
```

### desktop-notifications

アプリケーション通知を許可するかを指定します（OSC 9/777）。

- **デフォルト**: `true`

```
desktop-notifications = true
```

### image-storage-limit

画像データの上限をバイト単位で指定します。

- **デフォルト**: 320MB（最大4GB）
- **0値**: 画像プロトコルを無効化

```
image-storage-limit = 536870912  # 512MB
```

## キーバインド設定

### keybind

キーバインドを設定します。

- **形式**: `trigger=action`
- **修飾子**: shift、ctrl（control）、alt（opt/option）、super（cmd/command）
- **物理キー**: W3C仕様（KeyA、Key_a、f1等）
- **Unicodeコード**: 文字（aなど）
- **シーケンス**: `ctrl+a>n=action`で複数キー組み合わせ

#### プレフィックス

- `global:` - システムグローバルショートカット（macOS/特定Linux）
- `all:` - 全ターミナル対象
- `unconsumed:` - 入力をターミナルにも送信
- `performable:` - 実行可能時のみ

#### アクション

- `ignore` - 何もしない
- `unbind` - バインド削除
- `csi:text` - CSIシーケンス
- `esc:text` - Escapeシーケンス
- `text:text` - テキスト（Zig文字列構文）

```
keybind = ctrl+t=new_tab
keybind = ctrl+w=close_tab
keybind = ctrl+shift+c=copy_to_clipboard
keybind = ctrl+shift+v=paste_from_clipboard
keybind = ctrl+a>n=new_window
keybind = global:ctrl+grave_accent=toggle_quick_terminal
```

## シェル統合設定

### shell-integration

シェル統合機能を制御します。

- **可能な値**: `none`、`detect`、`bash`、`elvish`、`fish`、`zsh`
- **デフォルト**: `detect`
- **機能**: ディレクトリ報告、プロンプト表示、確認スキップ等

```
shell-integration = zsh
```

### shell-integration-features

シェル統合の個別機能を制御します。

- **可能な値**: `cursor`、`sudo`、`title`、`ssh-env`、`ssh-terminfo`
- **デフォルト**: 全有効
- **ssh系**: 1.2.0以降、terminfo自動インストール機能

```
shell-integration-features = cursor,sudo,title
```

## カスタムシェーダ・ベル設定

### custom-shader / custom-shader-animation

GLSLカスタムシェーダを指定します。

- **形式**: ファイルパス（複数指定可）
- **animation**: `true`（デフォルト）/`false`/`always`
- **注意**: シェーダコンパイルエラーは起動時ログに表示されます

```
custom-shader = /path/to/shader.glsl
custom-shader-animation = true
```

### bell-features

ベル機能を制御します。

- **可能な値**: `system`、`audio`、`attention`、`title`、`border`
- **デフォルト**: `attention`=有効、`title`=有効
- **形式**: コンマ区切り、`no-○○`で無効化

```
bell-features = audio,attention,no-border
```

### bell-audio-path / bell-audio-volume

ベル音声ファイルとボリュームを指定します。

- **volume範囲**: 0.0～1.0（デフォルト0.5）
- **限定**: GTK Linuxのみ

```
bell-audio-path = /path/to/bell.wav
bell-audio-volume = 0.3
```

### app-notifications

アプリケーション通知を制御します。

- **可能な値**: `clipboard-copy`、`config-reload`
- **デフォルト**: 両方 `true`
- **限定**: GTKのみ

```
app-notifications = clipboard-copy,config-reload
```

## macOS限定設定

### macos-non-native-fullscreen

ネイティブ以外の全画面モードを制御します。

- **可能な値**: `true`、`false`、`visible-menu`、`padded-notch`

```
macos-non-native-fullscreen = visible-menu
```

### macos-window-buttons

ウィンドウボタンの表示を制御します。

- **可能な値**: `visible`、`hidden`
- **デフォルト**: `visible`
- **利用可能**: 1.2.0以降

```
macos-window-buttons = hidden
```

### macos-titlebar-style

タイトルバーのスタイルを指定します。

- **可能な値**: `native`、`transparent`、`tabs`、`hidden`
- **デフォルト**: `transparent`

```
macos-titlebar-style = tabs
```

### macos-titlebar-proxy-icon

タイトルバーのプロキシアイコンを表示するかを指定します。

```
macos-titlebar-proxy-icon = true
```

### macos-option-as-alt

Optionキーを Altキーとして扱うかを指定します。

```
macos-option-as-alt = true
```

### macos-dock-drop-behavior

Dockへのドロップ動作を指定します。

- **可能な値**: `new-tab`、`new-window`
- **デフォルト**: `new-tab`

```
macos-dock-drop-behavior = new-window
```

### macos-window-shadow

ウィンドウの影を表示するかを指定します。

- **デフォルト**: `true`

```
macos-window-shadow = false
```

### macos-hidden

ドック・アプリ切替画面から非表示にするかを指定します。

- **可能な値**: `never`、`always`
- **利用可能**: 1.2.0以降

```
macos-hidden = always
```

### macos-auto-secure-input / macos-secure-input-indication

セキュア入力の自動有効化と表示を制御します。

```
macos-auto-secure-input = true
macos-secure-input-indication = true
```

### macos-icon

アプリケーションアイコンを指定します。

- **可能な値**: `official`、`blueprint`、`chalkboard`等、`custom`、`custom-style`
- **custom要件**: `macos-custom-icon`でパス指定
- **custom-style要件**: フレーム・色設定必須

```
macos-icon = blueprint
```

### macos-shortcuts

Shortcuts.appの制御を許可するかを指定します。

- **可能な値**: `ask`、`allow`、`deny`
- **利用可能**: 1.2.0以降

```
macos-shortcuts = ask
```

## Linux/GTK限定設定

### linux-cgroup

リソース隔離制御を指定します。

- **可能な値**: `never`、`always`、`single-instance`
- **要件**: systemd

```
linux-cgroup = single-instance
```

### linux-cgroup-memory-limit / linux-cgroup-processes-limit

ターミナルプロセスのメモリ・プロセス数上限を指定します。

```
linux-cgroup-memory-limit = 1G
linux-cgroup-processes-limit = 256
```

### gtk-single-instance

単一インスタンスモードを制御します。

- **可能な値**: `true`、`false`、`detect`
- **デフォルト**: `detect`
- **注意**: 1.2.0で`desktop`廃止、`detect`に統一

```
gtk-single-instance = true
```

### gtk-titlebar / gtk-tabs-location

GTKタイトルバーとタブ位置を制御します。

- **tabs-location**: `top`、`bottom`、`hidden`

```
gtk-titlebar = true
gtk-tabs-location = top
```

### gtk-titlebar-hide-when-maximized

最大化時にタイトルバーを非表示にするかを指定します。

```
gtk-titlebar-hide-when-maximized = true
```

### gtk-toolbar-style / gtk-titlebar-style

ツールバーとタイトルバーのスタイルを指定します。

- **toolbar**: `flat`、`raised`、`raised-border`

```
gtk-toolbar-style = flat
```

### gtk-wide-tabs

タブ幅を広げるかを指定します。

```
gtk-wide-tabs = true
```

### gtk-custom-css

カスタムCSSファイルを指定します。

```
gtk-custom-css = /path/to/custom.css
```

### gtk-opengl-debug

OpenGLデバッグログを有効化します。

- **デフォルト**: debug=`true`、release=`false`

```
gtk-opengl-debug = true
```

## パフォーマンス・その他設定

### term

TERM環境変数の値を指定します。

- **デフォルト**: xterm-ghosttyプリフィックス

```
term = xterm-256color
```

### osc-color-report-format

OSC色レポートの形式を指定します。

- **可能な値**: `none`、`8-bit`、`16-bit`
- **デフォルト**: `16-bit`

```
osc-color-report-format = 16-bit
```

### vt-kam-allowed

KAM（キーボード無効）モードを許可するかを指定します。

- **デフォルト**: `false`

```
vt-kam-allowed = false
```

### async-backend

非同期バックエンドを指定します。

- **可能な値**: `auto`、`epoll`、`io_uring`
- **デフォルト**: `auto`
- **限定**: Linuxのみ

```
async-backend = io_uring
```

### auto-update / auto-update-channel

自動更新とチャネルを指定します。

- **可能値**: `off`、`check`、`download`
- **チャネル**: `stable`、`tip`
- **限定**: macOSのみ

```
auto-update = download
auto-update-channel = stable
```

### config-file

追加の設定ファイルを指定します。

- **注意**: 複数指定可、サイクル禁止

```
config-file = /path/to/additional-config
```

### config-default-files

デフォルト設定ファイルを読み込むかを指定します。

```
config-default-files = true
```

### confirm-close-surface

ウィンドウ終了時に確認を要求するかを指定します。

```
confirm-close-surface = true
```

### quit-after-last-window-closed / quit-after-last-window-closed-delay

最後のウィンドウ終了でアプリを終了するかを指定します。

- **デフォルト**: macOS=`false`、Linux=`true`
- **delay最小**: 1s（Linuxのみ）

```
quit-after-last-window-closed = true
quit-after-last-window-closed-delay = 2s
```

### initial-window

起動時の初期ウィンドウを作成するかを指定します。

- **限定**: Linux・macOS

```
initial-window = true
```

### undo-timeout

Undo操作の有効期限を指定します。

- **デフォルト**: 5秒
- **限定**: macOSのみ

```
undo-timeout = 10s
```

## Quick Terminal設定

### quick-terminal-position / quick-terminal-size

Quick Terminalの位置とサイズを指定します。

- **position**: `top`、`bottom`、`left`、`right`、`center`
- **size**: パーセンテージ（%）またはピクセル（px）
- **利用可能**: 1.2.0以降

```
quick-terminal-position = top
quick-terminal-size = 50%
```

### quick-terminal-screen

表示スクリーンを指定します。

- **可能な値**: `main`、`mouse`、`macos-menu-bar`
- **限定**: macOS

```
quick-terminal-screen = mouse
```

### quick-terminal-animation-duration

アニメーション時間を指定します。

```
quick-terminal-animation-duration = 200ms
```

### quick-terminal-autohide

自動非表示を有効化します。

```
quick-terminal-autohide = true
```

### quick-terminal-space-behavior

Space移動の動作を指定します。

### gtk-quick-terminal-layer / gtk-quick-terminal-namespace

GTKレイヤとネームスペースを指定します。

- **限定**: GTK Wayland

### quick-terminal-keyboard-interactivity

キーボード操作を指定します。

- **限定**: GTK Wayland

## 設定ファイルの管理

### デフォルトパス

- `$XDG_CONFIG_HOME/ghostty/config`
- `~/.config/ghostty/config`

### テーマディレクトリ

- `$XDG_CONFIG_HOME/ghostty/themes`
- Ghosttyリソース内themes/

### 設定の反映

全設定は**ランタイム変更対応**（新規ターミナルにのみ反映）または**即座反映**の2種類があります。詳細は各オプション説明を参照してください。

## まとめ

Ghosttyは非常に多機能なターミナルエミュレータで、細かいカスタマイズが可能です。この記事では全ての設定オプションを網羅的に解説しました。

設定を変更する際は、公式ドキュメントと併せてこの記事を参考にしてください。

## 参考

- [Ghostty公式サイト](https://ghostty.org/)
- [Ghostty設定リファレンス](https://ghostty.org/docs/config/reference)
