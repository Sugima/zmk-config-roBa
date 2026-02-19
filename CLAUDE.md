# roBa ZMK キーボード設定

## プロジェクト概要

roBa は Seeeduino XIAO BLE ベースの左右分割キーボード。右手にPMW3610トラックボール、左手にEC11ロータリーエンコーダを搭載。

## ハードウェア構成

- MCU: Seeeduino XIAO BLE (nRF52840)
- トラックボール: PMW3610 (右手側、SPI接続、kumamuk-git ドライバ)
- エンコーダ: EC11 (左手側)
- キー数: **43キー** (4行×11列マトリクス、RC(1,5) に追加キーあり)
- 接続: BLE分割

### キーマトリクス配置

- Row 0: 10キー (左5 + 右5)
- Row 1: **12キー** (左6 + 右6) — RC(1,5) が追加キー (通常 `&none`)
- Row 2: 12キー (左6 + 右6)
- Row 3: 9キー (サムクラスタ)

### PMW3610 ドライバ (kumamuk-git)

- kumamuk-git 版を使用。badjeff 版とは **Y軸の方向が逆**
- `scroll-layers` はドライバレベルで設定 (scroller child node 方式は使わない)
- `zip_keybind_gesture` 等の上下バインディングは、この Y 軸方向を前提に設定されている

## レイヤー構成

| # | Name     | 用途                                    |
|---|----------|-----------------------------------------|
| 0 | default  | QWERTY                                  |
| 1 | NAVI     | 矢印・Home/End・Delete                  |
| 2 | NUM      | 数字・記号                               |
| 3 | FUNCTION | F1-F12                                  |
| 4 | MISC     | BT接続・メディア・スクリーンショット       |
| 5 | MOUSE    | マウスボタン (トラックボール移動で自動活性化) |
| 6 | SCROLL   | ブラウザ/ウィンドウ操作                    |
| 7 | GESTURE  | トラックボールジェスチャ (F長押しで進入)     |

## ファイル構成

| ファイル | 役割 |
|---------|------|
| `config/roBa.keymap` | キーマップ本体 (レイヤー、コンボ、マクロ、ビヘイビア、ジェスチャ) |
| `config/west.yml` | ビルド依存関係 (ZMK, PMW3610ドライバ, keybindプロセッサ) |
| `boards/shields/roBa/roBa.dtsi` | 物理レイアウト・マトリクス定義 (**変更することは稀**) |
| `boards/shields/roBa/roBa_R.overlay` | 右手側: トラックボール設定、自動マウスレイヤー、ジェスチャリスナー |
| `boards/shields/roBa/roBa_R.conf` | PMW3610 ドライバ設定 (CPI, 向き, スクロール等) |
| `boards/shields/roBa/roBa_L.overlay` | 左手側: エンコーダ有効化 |
| `build.yaml` | GitHub Actions ビルドマトリクス |

## 主要な設計パターン

### マウスクリックマクロ
`mkp_mb1_exit` 等: マウスボタン押下 → リリース時に `&to 0` でデフォルトレイヤーに戻る。

### ジェスチャシステム
1. MOUSE レイヤーで F キー長押し → `&mo GESTURE` で GESTURE レイヤー活性化
2. `roBa_R.overlay` の `trackball_listener` → gesture child node が GESTURE レイヤーで `zip_keybind_gesture` を適用
3. トラックボールの4方向移動がキー入力に変換される (右: Cmd+], 左: Cmd+[, 下: Cmd+T, 上: Cmd+W)

### 自動マウスレイヤー
`roBa_R.overlay` の `zip_temp_layer 5 500`: トラックボール移動で MOUSE レイヤー (5) に自動遷移、500ms で戻る。

## ビルド・デプロイ

1. `git push` → GitHub Actions が自動ビルド
2. Actions の Artifacts からファームウェア (.uf2) をダウンロード
3. 左右それぞれのキーボードにフラッシュ

## 変更時の注意

- `roBa.dtsi` (物理レイアウト) と `roBa_R.conf` (ドライバ設定) は基本的に変更しない
- PMW3610 ドライバは kumamuk-git 版を維持する (badjeff 版への変更はリスクが高い)
- レイヤー番号を変更した場合、`roBa_R.overlay` の `zip_temp_layer` と gesture `layers` も更新すること
- `roBa_R.overlay` では keymap の `#define` が使えないため、レイヤー番号はリテラル値で指定する
