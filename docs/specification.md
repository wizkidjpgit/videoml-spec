# 🎥 VideoML 仕様書(ドラフト版) v1.0.2

VideoMLは、動画をマークアップ風に構造的に記述し、AIやプログラマーで自動生成されることを前提とした、新しい動画構成記述言語です。

---

## 📐 基本構文例

```xml
<videoml>
  <head>
    <meta name="duration" content="30s" />
    <meta name="resolution" content="1920x1080" />
    <meta name="fps" content="30" />
    <meta name="outputFormat" content="mp4" />
    <meta name="encoding" content="h264" />
    <voiceProfile id="narrator1" engine="gTTS" voice="ja-JP-Wavenet-B" rate="1.0" pitch="1.1" emotion="calm" />
  </head>
  <video>
    <!-- 映像要素 -->
  </video>
</videoml>
```

---

## 🧱 要素定義

### `<head>`

動画全体のメタデータを記述。

* `duration`: 動画の長さ
* `resolution`: 解像度（例: 1920x1080）
* `fps`: フレームレート（例: 30）
* `outputFormat`: 出力形式（例: mp4）
* `encoding`: エンコーディング形式（例: h264）
* `<voiceProfile>`: 音声プロファイルの定義（TTSや歌唱に使用）

---

### `<video>`

動画構成の本体要素。以下の子要素を含めることができます：

* `<sequence>`：動画シーケンスを論理的にグルーピング
* `<timeline>`：時間軸の制御（複数トラック対応）
* `<track>`：音声・映像・字幕などのトラック指定
* `<clip>`：動画ファイルのカット挿入
* `<text>`：テロップ、字幕、発話テキスト
* `<image>`：静止画像の表示
* `<bgm>`：BGM挿入
* `<layer>`：レイヤーの構造定義
* `<mask>`：マスク処理
* `<svg>`：装飾やアニメーション用SVG
* `<speech>`：TTS（テキスト音声合成）
* `<score>`：楽譜（歌唱表現）
* `<script>`：JSによる動的制御

---

### `<sequence>`

複数のシーン・セクションを順番にまとめるコンテナ。

```xml
<sequence>
  <clip src="intro.mp4" />
  <clip src="main.mp4" />
</sequence>
```

### `<timeline>`

タイムライン制御。複数のトラックを同時再生。

```xml
<timeline>
  <track type="video">
    <clip src="bg.mp4" />
  </track>
  <track type="text">
    <text content="こんにちは" />
  </track>
</timeline>
```

### `<track>`

タイムライン上の1つのレイヤー（映像・音声・字幕など）

| 属性     | 説明                                     |
| ------ | -------------------------------------- |
| `type` | トラックの種類（video / audio / text / svg など） |
| `id`   | 識別用ID（任意）                              |
| `position` | 表示位置（例: `top-left`, `center`, `bottom-right`） |
| `x`, `y` | カスタム位置指定（ピクセル単位、`vw`, `vh`対応、例: `x="10vw" y="20vh"`） |

---

### `<voiceProfile>`（音声プロファイル定義）

TTSや歌唱合成の声のパラメータセットを定義。
これにより`speech`や`score`で使い回しが可能。

```xml
<voiceProfile id="narrator1" engine="gTTS" voice="ja-JP-Wavenet-B" rate="1.0" pitch="1.1" emotion="calm" />
```

| 属性        | 説明                                      |
| --------- | --------------------------------------- |
| `id`      | 識別用ID（`voice="narrator1"` のように参照）       |
| `engine`  | 使用する音声エンジン（例: gTTS, Amazon Polly, etc.） |
| `voice`   | 音声IDやラベル（例: ja-JP-Wavenet-B）            |
| `rate`    | 話速（1.0が通常）                              |
| `pitch`   | 音程（1.0が通常）                              |
| `emotion` | 話し方の感情（calm, cheerfulなど）                |

`speech`や`score`では `voice="narrator1"` のように参照できます。

---

#### `<clip>`

| 属性             | 説明        |
| -------------- | --------- |
| `src`          | 動画ファイルパス  |
| `start`, `end` | 再生範囲      |
| `in`           | 開始時間      |
| `transition`   | トランジション名  |
| `mask`         | 適用するマスクID |

##### `start` / `end` の時間指定方法

VideoMLでは時間指定を以下のように多様な単位で行うことができます：

| 記述形式            | 意味                     | 例                        |
| --------------- | ---------------------- | ------------------------ |
| `3.0s`          | 秒（デフォルト）               | `start="3.0s"`           |
| `300ms`         | ミリ秒                        | `start="300ms"`          |
| `f90`           | フレーム番号                 | `start="f90"`（30fpsなら3秒） |
| `tc00:00:03:00` | タイムコード（HH\:MM\:SS\:FF） | `start="tc00:00:03:00"`  |

> パーサーは `f` で始まる場合はフレーム、`tc` で始まる場合はタイムコード、それ以外は秒として解釈します。

#### `<text>`

| 属性        | 説明                  |
| --------- | ------------------- |
| `content` | 表示する文字列             |
| `speak`   | TTSを使うか（true/false） |
| `voice`   | 使用音声の種類             |

#### `<speech>`

TTS専用。声の種類や感情・速度・同期ターゲットなど細かく指定可能。

| 属性           | 説明                                 |
| ------------ | ---------------------------------- |
| `content`    | 読み上げる文章（テキスト）                      |
| `src`        | 音声ファイルを直接指定する場合（省略可）               |
| `voice`      | 話者の種類（例: "ja\_female\_01"）         |
| `rate`       | 音声の速度（例: "1.0" = 通常速度）             |
| `pitch`      | ピッチ（高さ）調整（例: "0.8"）                |
| `volume`     | 音量（例: "1.0"）                       |
| `emotion`    | 感情表現（例: "joy", "anger"）            |
| `engine`     | 使用するTTSエンジン名（例: "Google", "Azure"） |
| `syncTarget` | リップシンク対象となるオブジェクトID                |
| `lang`       | 言語コード（例: "ja-JP"）                  |

```xml
<speech
  content="ようこそVideoMLの世界へ"
  voice="ja_female_01"
  rate="1.0"
  pitch="1.2"
  volume="0.9"
  emotion="joy"
  engine="AzureTTS"
  syncTarget="#face01"
  lang="ja-JP" />
```

> TTSエンジンによっては emotion, pitch などが未対応の場合もあります。

#### `<score>`

| 属性         | 説明                                                           |
| ---------- | ------------------------------------------------------------ |
| `voice`    | 歌声合成に使う声の種類                                                  |
| `lang`     | 言語コード（例: "ja-JP"）                                            |
| 子要素 `note` | 音階を定義。各noteは `pitch`, `duration`, `lyrics`, `velocity` などを持つ |

```xml
<score voice="ja_singer_01" lang="ja-JP">
  <note pitch="C4" duration="0.5s" lyrics="あ" />
  <note pitch="D4" duration="0.5s" lyrics="い" />
  <note pitch="E4" duration="0.5s" lyrics="う" />
</score>
```

#### `<transition>`

VideoMLでは `<clip transition="fade" transitionDuration="1.0s" />` のように、
2つの映像間の切り替えにアニメーション効果を指定できます。

アニメーションの種類：

* `fade`（フェードイン・アウト）
* `slide-left` / `slide-right`
* `wipe`
* `zoom`
* `crossfade`

CSSやJSアニメーションによっても拡張可能です。

---

## 🎨 スタイル定義 (VideoCSS)

VideoCSSは、VideoMLに対してスタイルやアニメーションを定義するためのCSSライクな構文です。

### 基本記法

```css
.text-large {
  font-size: 48px;
  color: white;
  animation: fadeIn 1s ease;
}

#logo {
  width: 200px;
  height: 100px;
  position: absolute;
  top: 10px;
  right: 10px;
}
```

### 主なプロパティ一覧

| プロパティ             | 説明             |
| ----------------- | -------------- |
| `font-size`       | 文字サイズ          |
| `color`           | 文字色            |
| `background`      | 背景色、グラデーション    |
| `position`        | 絶対 or 相対位置     |
| `top`, `left` 等   | 位置指定           |
| `width`, `height` | サイズ指定          |
| `z-index`         | レイヤー順序制御       |
| `opacity`         | 透明度            |
| `transform`       | 拡大縮小・回転・平行移動など |
| `animation`       | CSSアニメーション指定   |

### アニメーション指定

```css
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

---

## 🟩 クロマキー（Chroma Key）

VideoMLでは、映像クリップに対してクロマキー（グリーンバック等の背景透過）処理を指定できます。

### `<chromaKey>` 要素

`<clip>` の子要素または属性として指定します。

#### 属性例

| 属性         | 説明                         | 例                |
| ---------- | -------------------------- | ----------------- |
| `color`    | 透過する色（16進 or 色名）         | `#00FF00`（緑）    |
| `tolerance`| 許容範囲（0.0〜1.0）             | `0.1`             |
| `edge`     | エッジ補正（ぼかし半径pxなど）      | `2`               |

#### 記述例

```xml
<clip src="person.mp4">
  <chromaKey color="#00FF00" tolerance="0.08" edge="2" />
</clip>
```

---

## ✨ アニメーション・モーション表現

### SVGアニメーション

`<svg>`タグ内にCDATA形式でSMILやCSSアニメーションを埋め込み可能。

```xml
<svg>
  <![CDATA[
    <circle cx="50" cy="50" r="40" fill="red">
      <animate attributeName="r" from="40" to="10" dur="1s" repeatCount="indefinite" />
    </circle>
  ]]>
</svg>
```

### Motion Tween

`transform`, `translate`, `scale`, `rotate`などのプロパティを`VideoCSS`または`<script>`で制御可能。

### JavaScriptによる動的モーション

高度なアニメーション・同期処理は `<script type="application/javascript">` タグを使って記述します。

```xml
<script type="application/javascript">
  animateClip("#logo", {
    keyframes: [
      { time: 0, transform: "scale(0)" },
      { time: 0.5, transform: "scale(1)" }
    ]
  });
</script>
```

JSでは `animateClip`, `syncToSpeech`, `controlBGM` などの専用APIも定義可能とします。

---

## 🧠 `<script>`による動的制御

VideoMLは拡張性の高いJSベースの制御を可能にします。

```xml
<script type="application/javascript">
  document.getElementById("subtitle1").style.opacity = 0;
  setTimeout(() => {
    document.getElementById("subtitle1").style.opacity = 1;
  }, 2000);
</script>
```

### 利用目的

* 動的アニメーション制御
* ユーザーインタラクション（将来的な拡張）
* タイムライン制御や条件分岐

---

## 🎬 カットの繋ぎ方・編集構造

複数の `<clip>` を並べることで、編集可能です。

```xml
<clip src="a.mp4" end="5s" transition="fade" />
<clip src="b.mp4" start="5s" />
```

---

## 🖼 ワイプ・重ね合わせ・レイヤー順制御

`<layer zIndex="2">` で順序制御。下が背景・上が前面。

```xml
<layer zIndex="1"><clip src="bg.mp4"/></layer>
<layer zIndex="2"><clip src="face.mp4" mask="circle"/></layer>
```

---

## 📌 拡張性と応用例

* VideoML + VideoCSS + JS の3層構造により、構造・装飾・動作を分離
* FFmpegやRemotionなどの出力形式にも変換可能
* OSS拡張で音声合成エンジンやエフェクトも差し替え可能

---

## 🧪 バリデーション（XSD）

VideoMLはXML構文であり、厳密なスキーマチェックが可能です。XSDは仕様に準拠した完全版が別ファイルにて提供されます。

---

## 🛠 GitHub OSS展開予定構成

* `/spec/` : XSDファイル群
* `/examples/` : 実装サンプル
* `/converter/` : FFmpeg変換用コード
* `/editor/` : GUIエディタ用Vue/React構成

---

（最終更新: 2025年7月18日）
