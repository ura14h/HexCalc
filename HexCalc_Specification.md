<!--
この Markdown ファイルは Visual Studio Code と以下のプラグインを使って書かれ HTML ファイルへエクスポートしました。
  - Markdown Preview Enhanced by Yiyi Wang
-->
<style>
html body[for="html-export"]:not([data-presentation-mode]):not([html-show-sidebar-toc]) .markdown-preview {
    left: 0 !important;
    transform: none !important;
}
html body[for="html-export"]:not([data-presentation-mode]) .markdown-preview {
    padding: 2em !important;
}
html body table {
    font-size: 88% !important;
}
html body h1 {
    border-bottom: 2pt solid #000 !important;
}
html body h2 {
    margin-top: 2em !important;
    border-bottom: 1pt solid #888 !important;
}
html body h3 {
    border-bottom: 1pt solid #ccc !important;
}
</style>


# 16進電卓

これは [16進電卓](https://itunes.apple.com/jp/app/16進電卓/id335773208?mt=8) のキー入力制御に関する仕様メモです。

## 1. 画面レイアウト

![16進電卓の画面レイアウト](Screen_Shot.png) 

### 1.1 キー

| **キーの種類**     | キーの画面での名前 |
|--------------------|--------------------|
| **数字キー**       | `0`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `A`, `B`, `C`, `D`, `E`, `F` |
| **単項演算子キー** | `NOT`, `NEG` |
| **二項演算子キー** | `+`, `-`, `×`, `÷`, `MOD`, `AND`, `OR`, `XOR` |
| **結果キー**       | `=` |
| **削除キー**       | `DEL` |
| **クリアキー**     | `C` (赤色) |
| **全クリアキー**   | `AC` |

## 2. 状態遷移図

### 2.1 キー入力全体

```puml
@startuml

state "初期状態" as initial
state "数値1入力中" as number1
state "数式入力中" as formula
state "数値2入力中" as number2
state "結果表示中" as result
state "エラー表示中" as error

[*] --> initial

initial -left-> number1: 数字,単項演算子
initial -> formula: 二項演算子

number1 -> initial: 全クリア
number1 -> number1: 数字,単項演算子,\n削除,クリア
number1 --> formula: 二項演算子
number1 --> result: 結果

formula -> initial: 全クリア
formula -> formula: 二項演算子
formula --> number2: 数字
formula -right-> result: 結果(正常)
formula --> error: 結果(異常)

number2 -> initial: 全クリア
number2 -> number2: 数字,単項演算子,\n削除,クリア
number2 --> result: 結果(正常)
number2 --> formula: 二項演算子(正常)
number2 --> error: 結果(エラー),二項演算子(エラー)

result -> initial: 全クリア
result -> result: 結果,単項演算子,\n削除,クリア
result --> number1: 数字
result --> formula: 二項演算子

error -right-> initial: クリア

@enduml
```

### 2.2 クリアキー

```puml
@startuml

state "ACモード" as ac_mode
state "Cモード" as c_mode

[*] --> ac_mode
ac_mode -> c_mode: 数字
c_mode -> ac_mode: 全クリア

@enduml
```

## 3. 状態遷移表

|                  | **数字キー** | **単項演算子キー** | **二項演算子キー** | **結果キー** | **削除キー** | **クリアキー** | **全クリアキー** |
|------------------|--------------|--------------------|--------------------|--------------|--------------|----------------|------------------|
| **初期状態**     | 数値1=入力数字<br/>→**数値1入力中へ** | 数値1=入力演算子(数値1)<br/>→**数値1入力中へ** | 数値1=0<br/>最終演算子=入力演算子<br/>→**数式入力中へ** | 無視 | 無視 | 無視 | 無視 |
| **数値1入力中**  | 数値1=数値1×基数+入力数字 | 数値1=入力演算子(数値1) | 最終演算子=入力演算子<br/>→**数式入力中へ** | 数値1=最終演算子(数値1,数値2)<br/>成功の時: →**結果表示中へ**<br/>失敗の時: →**エラー表示中へ** | 数値1=数値1÷基数 | 数値1=0 | 数値1=0<br/>数値2=0<br/>→**初期状態へ** |
| **数式入力中**   | 数値2=入力数字<br/>→**数値2入力中へ** | 数値1=入力演算子(数値1)<br/>→**数値1入力中へ** | 最終演算子=入力演算子 | 数値1=最終演算子(数値1,数値2)<br/>成功の時: →**結果表示中へ**<br/>失敗の時: →**エラー表示中へ** | 数値1=数値1÷基数 | 数値1=0 | 数値1=0<br/>数値2=0<br/>→**初期状態へ** |
| **数値2入力中**  | 数値2=数値2×基数+入力数字 | 数値2=入力演算子(数値2) | 数値1=最終演算子(数値1,数値2)<br/>最終演算子=入力演算子<br/>成功の時: →**数式入力中へ**<br/>失敗の時: →**エラー表示中へ** | 数値1=最終演算子(数値1,数値2)<br/>成功の時: →**結果表示中へ**<br/>失敗の時: →**エラー表示中へ** | 数値2=数値2÷基数 | 数値2=0 | 数値1=0<br/>数値2=0<br/>→**初期状態へ** |
| **結果表示中**   | 数値1=入力数字<br/>→**数値1入力中へ** | 数値1=入力演算子(数値1)<br/>→**数値1入力中へ** | 最終演算子=入力演算子<br/>→**数式入力中へ** | 数値1=最終演算子(数値1,数値2)<br/>成功の時: →**結果表示中へ**<br/>失敗の時: →**エラー表示中へ** | 数値1=数値1÷基数 | 数値1=0 | 数値1=0<br/>数値2=0<br/>→**初期状態へ** |
| **エラー表示中** | 無視 | 無視 | 無視 | 無視 | 無視 | 数値1=0<br/>数値2=0<br/>→**初期状態へ** | 数値1=0<br/>数値2=0<br/>→**初期状態へ** |

以上

---
2018-05-19 H.Ishiura