# 第1章：術式習得編 - 基本的な術式

## Rustの世界へようこそ

よし、ここからが本格的な修行の始まりだ。まずはRustという言語の基本的な術式から覚えていこう。

俺たち呪術師が最初に習得するのは基本的な呪力操作だろう？Rustも同じだ。基礎をしっかり固めないと、後で高等術式を習得するときに必ず躓く。

## Rust環境の構築 - 術式発動の準備

まずは環境構築から。これは呪術師が術式を発動する前の準備のようなものだ。

### Rustupのインストール

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

!!! note "五条先生のアドバイス"
    これでRustのツールチェーン一式がインストールされる。`rustc`（コンパイラ）、`cargo`（パッケージマネージャー）、`rustup`（ツール管理）が使えるようになる。

    俺の無下限術式みたいに、一度セットアップすれば後は楽勝だ。

### 最初のプログラム - 「Hello, 最強!」

新しいプロジェクトを作ってみよう：

```bash
cargo new hello_saikyou
cd hello_saikyou
```

`src/main.rs`を開いて、以下のコードを書く：

```rust
fn main() {
    println!("Hello, 最強!");
    println!("俺の名前は五条悟。よろしく。");
}
```

実行してみよう：

```bash
cargo run
```

!!! tip "術式のコツ"
    `println!`は標準出力に文字列を表示するマクロだ。マクロは名前の後に`!`がつく。
    これは俺の「術式順転『蒼』」みたいなもので、決まった形式で強力な効果を発揮する。

## コメント - 術式の解説

コードに注釈をつけるのは重要だ。俺だって教え子に術式を教えるときは丁寧に説明するからね。

```rust
// これは一行コメント - 術式の簡単な説明

/*
 * これは複数行コメント
 * 複雑な術式の詳細な解説に使う
 */

/// これはドキュメンテーションコメント
/// 関数や構造体の説明に使う（rustdocで自動生成される）
fn saikyou_technique() {
    // ここに最強の術式を書く
}
```

## 文と式 - 術式の構造

Rustには\*\*文（Statement）**と**式（Expression）\*\*がある。これを理解するのは重要だ。

```rust
fn main() {
    // 文（Statement）- 値を返さない
    let x = 5;  // 変数宣言文

    // 式（Expression）- 値を返す
    let y = {
        let inner = 3;
        inner + 1  // セミコロンがない！これが式の値になる
    }; // yは4になる

    println!("x: {}, y: {}", x, y);
}
```

!!! note "式と文の違い"
    - **文**：処理を実行するが値を返さない（セミコロンで終わる）
    - **式**：値を評価して返す（セミコロンがない）

    俺の術式で例えると、「術式順転『蒼』」の発動は文で、その威力の計算は式だ。

## 基本的なデータ型 - 呪力の種類

### 整数型

```rust
fn main() {
    let power_level: i32 = 999999;  // 32ビット符号付き整数
    let curse_count: u64 = 1000000000;  // 64ビット符号なし整数

    // 型推論も使える
    let small_power = 100;  // デフォルトはi32

    println!("俺の呪力レベル: {}", power_level);
}
```

### 浮動小数点型

```rust
fn main() {
    let precision: f64 = 99.999;  // 64ビット浮動小数点
    let speed: f32 = 299792458.0;  // 32ビット浮動小数点

    println!("術式の精度: {}%", precision);
}
```

### 論理型

```rust
fn main() {
    let is_saikyou = true;
    let has_limits = false;

    if is_saikyou && !has_limits {
        println!("俺は最強だ");
    }
}
```

### 文字型

```rust
fn main() {
    let technique = '蒼';  // Unicodeスカラー値
    let symbol = '∞';      // 無下限のシンボル

    println!("術式: {}", technique);
}
```

### 文字列型

```rust
fn main() {
    // 文字列スライス（&str）- 不変
    let greeting = "やあやあ";

    // String型 - 可変長、ヒープに保存
    let mut technique_name = String::from("術式順転");
    technique_name.push_str("『蒼』");

    println!("{}: {}", greeting, technique_name);
}
```

## 基本的な演算 - 呪力の計算

```rust
fn main() {
    // 算術演算
    let base_power = 1000;
    let multiplier = 10;

    let total_power = base_power + multiplier * 100;  // 2000
    let remaining = total_power % 300;                // 200

    // 論理演算
    let is_strongest = true;
    let has_limitless = true;
    let is_gojo = is_strongest && has_limitless;

    // 比較演算
    let enemy_power = 500;
    let can_defeat = total_power > enemy_power;

    println!("呪力: {}, 敵を倒せる: {}", total_power, can_defeat);
}
```

## 型変換 - 呪力の変換術

```rust
fn main() {
    let power_i32: i32 = 1000;
    let power_f64 = power_i32 as f64;  // 明示的な型変換

    // 文字列から数値への変換
    let power_str = "9999";
    let power_from_str: i32 = power_str.parse()
        .expect("数値変換に失敗しました");

    println!("変換された呪力: {}", power_from_str);
}
```

!!! tip "五条流プログラミング"
    型変換は慎重に行え。間違った変換は術式の暴走につながる。
    `as`キーワードを使った変換は情報が失われる可能性があるから注意しろ。

## 定数 - 不変の真理

```rust
// グローバル定数（コンパイル時に決定）
const INFINITY_POWER: u32 = 999_999_999;
const TECHNIQUE_NAME: &str = "無下限呪術";

fn main() {
    println!("最大呪力: {}", INFINITY_POWER);
    println!("奥義: {}", TECHNIQUE_NAME);
}
```

## まとめ - 基礎術式の完成

ここまでで基本的な術式の習得は完了だ。どうだった？思ったより簡単だっただろう？

覚えておくべき重要なポイント：

1. **型システム** - Rustは型に厳格。でもそれが安全性を保証してくれる
1. **式と文** - セミコロンの有無で意味が変わる
1. **型推論** - 明示しなくてもコンパイラが推論してくれる
1. **不変性** - デフォルトで値は不変（`mut`で可変にできる）

次は変数と可変性について学んでいこう。俺の術式もそうだけど、時には変化が必要なんだ。

______________________________________________________________________

*「基礎をしっかり固めることが、最強への第一歩だ」*
