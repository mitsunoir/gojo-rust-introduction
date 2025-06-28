# 借用と参照 - 呪力の共有術

## 借用システム - 力を貸し与える技術

所有権の移動は強力だが、いちいち所有権を手放すのは面倒だろう？そこで登場するのが\*\*借用（Borrowing）\*\*だ。

これは俺が教え子に術式を教える時みたいなものだ。俺の知識は手放さずに、必要な分だけ共有する。それが借用の本質だ。

!!! note "五条先生の解説"
    借用は参照（Reference）を使って実現される。データの所有権を移動させずに、そのデータにアクセスする方法だ。
    `&`記号を使って参照を作り、`*`記号で参照先の値にアクセスする。

## 不変な借用 - 読み取り専用の共有

### 基本的な参照

```rust
fn main() {
    let technique = String::from("術式順転『蒼』");
    let reference = &technique;  // 参照を作成

    println!("元のデータ: {}", technique);    // 所有者も使える
    println!("参照経由: {}", reference);      // 参照も使える
    println!("参照先: {}", *reference);       // デリファレンス
}
```

### 関数での借用

```rust
fn analyze_technique(technique: &String) -> usize {
    technique.len()  // 参照先のメソッドを呼び出し
}

fn display_technique(technique: &String) {
    println!("技名: {}", technique);
}

fn main() {
    let my_technique = String::from("術式順転『蒼』");

    let length = analyze_technique(&my_technique);  // 借用
    display_technique(&my_technique);               // 借用

    println!("文字数: {}", length);
    println!("元の技: {}", my_technique);  // まだ使える！
}
```

### 複数の不変参照

```rust
fn main() {
    let technique = String::from("無下限呪術");

    let ref1 = &technique;
    let ref2 = &technique;
    let ref3 = &technique;

    // 複数の不変参照は同時に存在できる
    println!("参照1: {}", ref1);
    println!("参照2: {}", ref2);
    println!("参照3: {}", ref3);
    println!("元データ: {}", technique);
}
```

## 可変な借用 - 修正可能な共有

### 基本的な可変参照

```rust
fn main() {
    let mut technique = String::from("術式順転");
    let mutable_ref = &mut technique;  // 可変参照

    mutable_ref.push_str("『蒼』");  // 参照経由で変更

    println!("変更後: {}", mutable_ref);
}
```

### 関数での可変借用

```rust
fn power_up_technique(technique: &mut String) {
    technique.push_str(" - 最大出力");
}

fn clear_technique(technique: &mut String) {
    technique.clear();
    technique.push_str("新しい術式");
}

fn main() {
    let mut my_technique = String::from("術式順転『蒼』");
    println!("変更前: {}", my_technique);

    power_up_technique(&mut my_technique);  // 可変借用
    println!("強化後: {}", my_technique);

    clear_technique(&mut my_technique);     // 可変借用
    println!("変更後: {}", my_technique);
}
```

## 借用の規則 - 呪力管理の法則

### 規則1: 同時に存在できる参照の制限

```rust
fn main() {
    let mut technique = String::from("術式順転『蒼』");

    // パターン1: 複数の不変参照 - OK
    let ref1 = &technique;
    let ref2 = &technique;
    println!("{}, {}", ref1, ref2);

    // パターン2: 1つの可変参照 - OK
    let mut_ref = &mut technique;
    mut_ref.push_str(" - 強化");
    println!("{}", mut_ref);

    // パターン3: 不変と可変の混在 - エラー！
    /*
    let ref3 = &technique;
    let mut_ref2 = &mut technique;  // エラー！
    println!("{}, {}", ref3, mut_ref2);
    */
}
```

### 規則2: 参照のライフタイム

```rust
fn main() {
    let technique;

    {
        let temp = String::from("一時的な術式");
        // technique = &temp;  // エラー！tempのライフタイムが短い
    }

    // println!("{}", technique);  // エラー！

    // 正しい例
    let permanent = String::from("永続的な術式");
    let valid_ref = &permanent;
    println!("{}", valid_ref);  // OK
}
```

## スライス - 部分的な借用

### 文字列スライス

```rust
fn main() {
    let technique = String::from("術式順転『蒼』");

    // 部分的な借用（スライス）
    let technique_type = &technique[0..6];   // "術式順転"
    let technique_name = &technique[6..];    // "『蒼』"
    let full_slice = &technique[..];         // 全体

    println!("種類: {}", technique_type);
    println!("名前: {}", technique_name);
    println!("全体: {}", full_slice);

    // 文字列リテラルは&str型
    let literal: &str = "術式反転『赫』";
    println!("リテラル: {}", literal);
}
```

### 関数での文字列スライス

```rust
// String と &str の両方を受け取れる
fn analyze_technique_name(name: &str) -> usize {
    name.len()
}

fn get_technique_grade(name: &str) -> &str {
    if name.contains("茈") {
        "特級"
    } else if name.contains("蒼") || name.contains("赫") {
        "上級"
    } else {
        "基本"
    }
}

fn main() {
    let owned_string = String::from("虚式『茈』");
    let string_literal = "術式順転『蒼』";

    // どちらも同じ関数で処理できる
    println!("文字数1: {}", analyze_technique_name(&owned_string));
    println!("文字数2: {}", analyze_technique_name(string_literal));

    println!("等級1: {}", get_technique_grade(&owned_string));
    println!("等級2: {}", get_technique_grade(string_literal));
}
```

### 配列スライス

```rust
fn main() {
    let powers = [1000, 1500, 2000, 3000, 9999];

    let weak_powers = &powers[0..2];    // [1000, 1500]
    let strong_powers = &powers[3..];   // [3000, 9999]
    let all_powers = &powers[..];       // 全体

    println!("弱い技: {:?}", weak_powers);
    println!("強い技: {:?}", strong_powers);

    // スライスを操作する関数
    let max_power = find_max_power(strong_powers);
    println!("最大威力: {}", max_power);
}

fn find_max_power(powers: &[i32]) -> i32 {
    let mut max = 0;
    for &power in powers {
        if power > max {
            max = power;
        }
    }
    max
}
```

## 実践例 - 呪術師戦闘ログシステム

```rust
struct BattleLog {
    entries: Vec<String>,
}

impl BattleLog {
    fn new() -> Self {
        BattleLog {
            entries: Vec::new(),
        }
    }

    // 不変借用でログを追加
    fn add_entry(&mut self, message: &str) {
        self.entries.push(String::from(message));
    }

    // 不変借用でログを取得
    fn get_entry(&self, index: usize) -> Option<&String> {
        self.entries.get(index)
    }

    // スライスでログの一部を取得
    fn get_recent_entries(&self, count: usize) -> &[String] {
        let start = self.entries.len().saturating_sub(count);
        &self.entries[start..]
    }

    // 不変借用でログを検索
    fn find_entries_containing(&self, keyword: &str) -> Vec<&String> {
        self.entries.iter()
            .filter(|entry| entry.contains(keyword))
            .collect()
    }

    // 可変借用でログを編集
    fn edit_entry(&mut self, index: usize, new_message: &str) -> bool {
        if let Some(entry) = self.entries.get_mut(index) {
            *entry = String::from(new_message);
            true
        } else {
            false
        }
    }

    // 不変借用でレポート生成
    fn generate_report(&self) -> String {
        format!("戦闘ログ: {} 件のエントリ", self.entries.len())
    }
}

// 呪術師構造体
struct Sorcerer {
    name: String,
    power: i32,
}

impl Sorcerer {
    fn new(name: &str, power: i32) -> Self {
        Sorcerer {
            name: String::from(name),
            power,
        }
    }

    // 不変借用で攻撃
    fn attack(&self, target: &Sorcerer, log: &mut BattleLog) -> i32 {
        let damage = self.power / 10;
        let message = format!("{} が {} を攻撃！ {} ダメージ",
                             self.name, target.name, damage);
        log.add_entry(&message);
        damage
    }

    // 可変借用でダメージを受ける
    fn take_damage(&mut self, damage: i32, log: &mut BattleLog) {
        self.power = (self.power - damage).max(0);
        let message = format!("{} が {} ダメージを受けた（残り呪力: {}）",
                             self.name, damage, self.power);
        log.add_entry(&message);
    }
}

fn simulate_battle(fighter1: &mut Sorcerer, fighter2: &mut Sorcerer) -> BattleLog {
    let mut log = BattleLog::new();

    log.add_entry(&format!("戦闘開始: {} vs {}", fighter1.name, fighter2.name));

    let mut round = 1;
    while fighter1.power > 0 && fighter2.power > 0 && round <= 5 {
        log.add_entry(&format!("--- ラウンド {} ---", round));

        // 1番手の攻撃
        let damage1 = fighter1.attack(fighter2, &mut log);
        fighter2.take_damage(damage1, &mut log);

        if fighter2.power <= 0 {
            log.add_entry(&format!("{} の勝利！", fighter1.name));
            break;
        }

        // 2番手の攻撃
        let damage2 = fighter2.attack(fighter1, &mut log);
        fighter1.take_damage(damage2, &mut log);

        if fighter1.power <= 0 {
            log.add_entry(&format!("{} の勝利！", fighter2.name));
            break;
        }

        round += 1;
    }

    if round > 5 {
        log.add_entry("時間切れで引き分け");
    }

    log
}

fn main() {
    let mut gojo = Sorcerer::new("五条悟", 2000);
    let mut sukuna = Sorcerer::new("両面宿儺", 1800);

    println!("=== 最強対決シミュレーション ===");

    let battle_log = simulate_battle(&mut gojo, &mut sukuna);

    // ログの表示
    println!("\\n{}", battle_log.generate_report());
    println!("\\n=== 戦闘ログ ===");

    // 最近のエントリを表示
    let recent = battle_log.get_recent_entries(10);
    for (i, entry) in recent.iter().enumerate() {
        println!("{}: {}", i + 1, entry);
    }

    // 攻撃ログだけを抽出
    println!("\\n=== 攻撃ログ ===");
    let attack_logs = battle_log.find_entries_containing("攻撃");
    for log in attack_logs {
        println!("{}", log);
    }
}
```

## 借用チェッカー - コンパイラの守護者

Rustのコンパイラには借用チェッカー（Borrow Checker）という強力な守護者がいる：

```rust
fn main() {
    let mut data = vec![1, 2, 3, 4, 5];

    // 不変参照を作成
    let first = &data[0];

    // 可変参照を作ろうとするとエラー
    // data.push(6);  // エラー！不変参照が生きている間は変更不可

    println!("最初の要素: {}", first);
    // ここでfirstのライフタイム終了

    // 今度は変更できる
    data.push(6);
    println!("データ: {:?}", data);
}
```

## 練習問題

### 問題1: 文字列分析関数

文字列スライスを受け取って、以下の情報を返す関数を作成せよ：

- 文字数
- 「術式」が含まれているか
- 最初の単語

<details>
<summary>解答を見る</summary>

```rust
fn analyze_text(text: &str) -> (usize, bool, &str) {
    let char_count = text.chars().count();
    let contains_jutsu = text.contains("術式");

    let first_word = text.split_whitespace()
        .next()
        .unwrap_or("");

    (char_count, contains_jutsu, first_word)
}

fn main() {
    let technique1 = "術式順転『蒼』";
    let technique2 = String::from("無下限 呪術 領域展開");

    let (count1, has_jutsu1, first1) = analyze_text(technique1);
    println!("{}: 文字数{}, 術式含む{}, 最初の単語'{}'",
             technique1, count1, has_jutsu1, first1);

    let (count2, has_jutsu2, first2) = analyze_text(&technique2);
    println!("{}: 文字数{}, 術式含む{}, 最初の単語'{}'",
             technique2, count2, has_jutsu2, first2);
}
```

</details>

### 問題2: ベクター操作システム

整数のベクターを借用して、以下の操作を行う関数群を作成せよ：

- 合計値を計算（不変借用）
- 最大値を見つける（不変借用）
- 全要素を2倍にする（可変借用）

<details>
<summary>解答を見る</summary>

```rust
// 不変借用で合計計算
fn calculate_sum(numbers: &[i32]) -> i32 {
    numbers.iter().sum()
}

// 不変借用で最大値検索
fn find_maximum(numbers: &[i32]) -> Option<i32> {
    numbers.iter().max().copied()
}

// 可変借用で全要素を2倍
fn double_all(numbers: &mut [i32]) {
    for num in numbers.iter_mut() {
        *num *= 2;
    }
}

// 不変借用でスライス表示
fn display_slice(numbers: &[i32], label: &str) {
    println!("{}: {:?}", label, numbers);
}

fn main() {
    let mut power_levels = vec![1000, 1500, 800, 2000, 1200];

    display_slice(&power_levels, "初期状態");

    let total = calculate_sum(&power_levels);
    println!("合計呪力: {}", total);

    if let Some(max) = find_maximum(&power_levels) {
        println!("最大呪力: {}", max);
    }

    // 部分的な操作
    let middle_slice = &mut power_levels[1..4];
    double_all(middle_slice);

    display_slice(&power_levels, "一部強化後");

    // 全体を強化
    double_all(&mut power_levels);
    display_slice(&power_levels, "全体強化後");

    let final_total = calculate_sum(&power_levels);
    println!("最終合計呪力: {}", final_total);
}
```

</details>

## まとめ

借用システムの習得は完了だ！重要なポイント：

1. **不変借用（&T）** - 読み取り専用、複数同時可能
1. **可変借用（&mut T）** - 読み書き可能、同時に1つだけ
1. **スライス** - 部分的なデータへの参照
1. **ライフタイム** - 参照の有効期間
1. **借用チェッカー** - コンパイル時の安全性保証

これで所有権を手放すことなく、データを安全に共有できるようになった。まさに俺の無下限術式のように、完璧な制御だ。

次はライフタイムについて詳しく学ぼう。参照がいつまで有効なのかを明示的に指定する技術だ。

______________________________________________________________________

*「借用を極めれば、所有権と共有の完璧なバランスが取れる」*
