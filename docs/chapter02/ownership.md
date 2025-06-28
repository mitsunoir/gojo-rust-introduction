# 第2章：呪力操作編 - 所有権システム

## 所有権システムとは - 呪力の絶対支配

さあ、ここからが本当のRustの力だ。所有権システム（Ownership System）- これは俺の無下限術式と同じく、**絶対的な支配力**を持つシステムだ。

他の言語では、メモリ管理はプログラマーに丸投げか、ガベージコレクションに頼っている。でもRustは違う。コンパイラが呪力の流れを完璧に管理してくれる。

!!! note "五条先生の解説"
    所有権システムは、プログラムが安全にメモリを管理するためのRust独自のシステムだ。
    これにより、メモリリークやダングリングポインタなどの呪いを完全に排除できる。

## 所有権の3つの法則

俺の無下限術式にも厳格なルールがあるように、所有権にも3つの絶対法則がある：

### 法則1：各値には所有者が存在する

```rust
fn main() {
    let technique = String::from(\"術式順転『蒼』\");  // techniqueが所有者
    // この時点で、techniqueが文字列データの唯一の所有者
}
```

### 法則2：同時に複数の所有者は存在しない

```rust
fn main() {
    let technique = String::from(\"術式順転『蒼』\");
    let other_technique = technique;  // 所有権がother_techniqueに移動

    // println!(\"{}\", technique);  // エラー！もう使えない
    println!(\"{}\", other_technique);  // これはOK
}
```

### 法則3：所有者がスコープを抜けると値は破棄される

```rust
fn main() {
    {
        let technique = String::from(\"術式順転『蒼』\");
        println!(\"{}\", technique);
    }  // ここでtechniqueは自動的に破棄される

    // println!(\"{}\", technique);  // エラー！スコープ外
}
```

## 所有権の移動（Move）- 呪力の移譲

### 基本的な移動

```rust
fn main() {
    let original = String::from(\"無下限呪術\");
    let transferred = original;  // 所有権が移動

    println!(\"移譲後: {}\", transferred);
    // println!(\"{}\", original);  // エラー！使用不可
}
```

### 関数への移動

```rust
fn cast_technique(spell: String) {
    println!(\"{}を発動！\", spell);
}  // ここでspellが破棄される

fn main() {
    let my_technique = String::from(\"術式順転『蒼』\");
    cast_technique(my_technique);  // 所有権が関数に移動

    // println!(\"{}\", my_technique);  // エラー！もう使えない
}
```

### 関数からの戻り値

```rust
fn create_technique() -> String {
    let technique = String::from(\"虚式『茈』\");
    technique  // 所有権を呼び出し元に移動
}

fn main() {
    let my_technique = create_technique();  // 所有権を受け取る
    println!(\"習得した技: {}\", my_technique);
}
```

## コピー可能な型 - 基本術式の複製

一部の型は`Copy`トレイトを実装しており、所有権の移動ではなく値のコピーが発生する：

```rust
fn main() {
    // 基本型はCopyトレイトを実装
    let power = 1000;
    let copied_power = power;  // コピーが発生

    println!(\"元の呪力: {}\", power);         // 使える！
    println!(\"コピーした呪力: {}\", copied_power);  // これも使える！

    // タプルも要素がすべてCopyならCopy
    let coordinates = (10, 20);
    let copied_coords = coordinates;

    println!(\"元の座標: {:?}\", coordinates);
    println!(\"コピーした座標: {:?}\", copied_coords);
}
```

### Copy vs Clone

```rust
fn main() {
    // Copy - 暗黙的な複製（スタック上の値）
    let x = 5;
    let y = x;  // 自動的にコピー

    // Clone - 明示的な複製（ヒープ上の値も可能）
    let technique = String::from(\"術式順転『蒼』\");
    let cloned_technique = technique.clone();  // 明示的にクローン

    println!(\"元の技: {}\", technique);
    println!(\"クローンした技: {}\", cloned_technique);
}
```

## 実践例 - 呪術師の技管理システム

```rust
struct Sorcerer {
    name: String,
    techniques: Vec<String>,
    power: i32,
}

impl Sorcerer {
    // 新しい呪術師を作成（所有権を受け取る）
    fn new(name: String) -> Self {
        Sorcerer {
            name,
            techniques: Vec::new(),
            power: 1000,
        }
    }

    // 技を習得（所有権を受け取る）
    fn learn_technique(&mut self, technique: String) {
        println!(\"{} が {} を習得！\", self.name, technique);
        self.techniques.push(technique);
        self.power += 200;
    }

    // 技を使用（所有権は移動しない）
    fn use_technique(&self, index: usize) -> Option<&String> {
        self.techniques.get(index)
    }

    // 技を忘れる（所有権を返す）
    fn forget_technique(&mut self, index: usize) -> Option<String> {
        if index < self.techniques.len() {
            Some(self.techniques.remove(index))
        } else {
            None
        }
    }

    // 技を他の呪術師に移譲
    fn transfer_technique(&mut self, other: &mut Sorcerer, index: usize) {
        if let Some(technique) = self.forget_technique(index) {
            println!(\"{} が {} に {} を移譲\",
                     self.name, other.name, technique);
            other.learn_technique(technique);
        }
    }
}

fn main() {
    // 呪術師作成
    let mut gojo = Sorcerer::new(String::from(\"五条悟\"));
    let mut megumi = Sorcerer::new(String::from(\"伏黒恵\"));

    // 技の習得
    gojo.learn_technique(String::from(\"術式順転『蒼』\"));
    gojo.learn_technique(String::from(\"術式反転『赫』\"));
    gojo.learn_technique(String::from(\"虚式『茈』\"));

    megumi.learn_technique(String::from(\"玉犬\"));
    megumi.learn_technique(String::from(\"大蛇\"));

    // 技の使用
    if let Some(technique) = gojo.use_technique(0) {
        println!(\"{} が {} を使用！\", gojo.name, technique);
    }

    // 技の移譲（基本術式を教える）
    gojo.transfer_technique(&mut megumi, 0);  // 蒼を移譲

    println!(\"\\n=== 最終状態 ===\");
    println!(\"{}: 呪力 {}, 技数 {}\", gojo.name, gojo.power, gojo.techniques.len());
    println!(\"{}: 呪力 {}, 技数 {}\", megumi.name, megumi.power, megumi.techniques.len());
}
```

## 所有権の設計パターン

### パターン1: 所有権を移動させる関数

```rust
// 文字列を受け取って加工し、新しい文字列を返す
fn enhance_technique(mut technique: String) -> String {
    technique.push_str(\" - 強化版\");
    technique
}

fn main() {
    let basic = String::from(\"術式順転『蒼』\");
    let enhanced = enhance_technique(basic);  // 所有権移動

    println!(\"{}\", enhanced);
    // basicはもう使えない
}
```

### パターン2: 複数の戻り値で所有権を返す

```rust
fn analyze_and_return(technique: String) -> (String, usize) {
    let length = technique.len();
    (technique, length)  // 所有権を戻す
}

fn main() {
    let technique = String::from(\"無下限呪術\");
    let (returned_technique, length) = analyze_and_return(technique);

    println!(\"{} の文字数: {}\", returned_technique, length);
}
```

### パターン3: 所有権を一時的に借用

```rust
// 次のセクションで詳しく説明するが、参照を使用
fn get_technique_info(technique: &String) -> usize {
    technique.len()  // 所有権を取らずに情報を取得
}

fn main() {
    let technique = String::from(\"術式順転『蒼』\");
    let info = get_technique_info(&technique);  // 借用

    println!(\"{} の情報: {}\", technique, info);  // まだ使える！
}
```

## 所有権とデータ競合

Rustの所有権システムは、データ競合を防ぐ：

```rust
use std::thread;

fn main() {
    let technique = String::from(\"術式順転『蒼』\");

    // 複数のスレッドで同じデータを使うとエラー
    /*
    let handle1 = thread::spawn(|| {
        println!(\"スレッド1: {}\", technique);  // エラー！
    });

    let handle2 = thread::spawn(|| {
        println!(\"スレッド2: {}\", technique);  // エラー！
    });
    */

    // 正しい方法は次章で学ぶ（Clone、Arc、Mutexなど）
    println!(\"メインスレッド: {}\", technique);
}
```

## 練習問題

<div class=\"exercise\">

### 問題1: 呪力の移譲

文字列を受け取って先頭に「最強の」を追加する関数を作成し、所有権の移動を確認せよ。

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
fn make_strongest(mut technique: String) -> String {
    technique.insert_str(0, \"最強の\");
    technique
}

fn main() {
    let basic_technique = String::from(\"呪術師\");
    println!(\"元: {}\", basic_technique);

    let strongest = make_strongest(basic_technique);
    println!(\"変換後: {}\", strongest);

    // println!(\"{}\", basic_technique);  // エラー！所有権が移動した
}
```

</div>
</details>

<div class=\"exercise\">

### 問題2: 技のコレクション管理

ベクターの所有権を操作して、技のリストを管理するプログラムを作成せよ。

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
fn add_technique(mut techniques: Vec<String>, new_technique: String) -> Vec<String> {
    techniques.push(new_technique);
    techniques
}

fn combine_techniques(mut tech1: Vec<String>, mut tech2: Vec<String>) -> Vec<String> {
    tech1.append(&mut tech2);  // tech2の内容をtech1に移動
    tech1
}

fn main() {
    let gojo_techniques = vec![
        String::from(\"術式順転『蒼』\"),
        String::from(\"術式反転『赫』\")
    ];

    let megumi_techniques = vec![
        String::from(\"玉犬\"),
        String::from(\"大蛇\")
    ];

    let enhanced_gojo = add_technique(gojo_techniques, String::from(\"虚式『茈』\"));
    let all_techniques = combine_techniques(enhanced_gojo, megumi_techniques);

    println!(\"全技術: {:?}\", all_techniques);
    println!(\"技数: {}\", all_techniques.len());
}
```

</div>
</details>

## まとめ

所有権システムの基本は理解できたか？重要なポイント：

1. **唯一の所有者** - 各値には必ず1人の所有者
1. **移動（Move）** - 所有権は移動する
1. **自動解放** - スコープ終了で自動的にメモリ解放
1. **Copy vs Clone** - スタック vs ヒープのデータ
1. **安全性** - コンパイル時にメモリ安全性を保証

これは俺の無下限術式と同じで、一度理解すれば絶対的な力になる。でも最初は戸惑うかもしれない。それが普通だ。

次は借用（Borrowing）について学ぼう。所有権を移動させずにデータを使う技術だ。

______________________________________________________________________

*「所有権を理解すれば、メモリの呪いから解放される」*
