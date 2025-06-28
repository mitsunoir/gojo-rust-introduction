# ライフタイム - 呪力の持続時間

## ライフタイムとは - 術式の有効期間

ライフタイムは参照がいつまで有効なのかを示すものだ。俺の術式にも発動時間があるように、参照にも「生きている期間」がある。

大抵の場合、Rustのコンパイラが自動で推論してくれるが、時には明示的に指定する必要がある。これが**ライフタイム注釈**だ。

!!! note "五条先生の解説"
    ライフタイム注釈は `'a` のような形で書く。これは「この参照はライフタイム'aの間有効」という意味だ。
    コンパイラがダングリングポインタ（無効な参照）を防ぐために使う。

## ライフタイムの基本概念

### 基本的なライフタイム

```rust
fn main() {
    let technique;               // -------+-- 'a
                                //        |
    {                           //        |
        let temp = String::from("蒼");  // -+-- 'b
        technique = &temp;      //        |  |
    }                           // -------+  |
                                //           |
    // println!("{}", technique); // エラー！'b < 'a
}                               // ----------+
```

### 有効なライフタイム

```rust
fn main() {
    let temp = String::from("術式順転『蒼』");  // ----+-- 'a
    let technique = &temp;                     //     |
    println!("{}", technique);                 //     |
}                                              // ----+
```

## 関数のライフタイム注釈

### 基本的な注釈

```rust
// 入力と出力が同じライフタイム
fn get_longer<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let technique1 = "蒼";
    let technique2 = "術式順転『蒼』";

    let longer = get_longer(technique1, technique2);
    println!("長い方: {}", longer);
}
```

### 複数のライフタイム

```rust
// 異なるライフタイムを持つ参照
fn combine_techniques<'a, 'b>(
    primary: &'a str,
    secondary: &'b str
) -> String {
    format!("{} + {}", primary, secondary)
}

fn main() {
    let blue = "蒼";
    let red = "赤";
    let combo = combine_techniques(blue, red);
    println!("コンボ: {}", combo);
}
```

## 構造体のライフタイム

### ライフタイムを持つ構造体

```rust
// 参照を保持する構造体
struct TechniqueRef<'a> {
    name: &'a str,
    power: i32,
}

impl<'a> TechniqueRef<'a> {
    fn new(name: &'a str, power: i32) -> Self {
        TechniqueRef { name, power }
    }

    fn display(&self) {
        println!("{}: 威力 {}", self.name, self.power);
    }

    // メソッドのライフタイム
    fn get_name(&self) -> &'a str {
        self.name
    }
}

fn main() {
    let technique_name = "無下限呪術";
    let technique = TechniqueRef::new(technique_name, 9999);

    technique.display();
    println!("技名: {}", technique.get_name());
}
```

### 複雑な構造体

```rust
struct Sorcerer<'a> {
    name: &'a str,
    techniques: Vec<&'a str>,
    power: i32,
}

impl<'a> Sorcerer<'a> {
    fn new(name: &'a str) -> Self {
        Sorcerer {
            name,
            techniques: Vec::new(),
            power: 1000,
        }
    }

    fn add_technique(&mut self, technique: &'a str) {
        self.techniques.push(technique);
        self.power += 200;
    }

    // 最強の技を返す
    fn get_strongest_technique(&self) -> Option<&'a str> {
        // 簡易的な判定
        if self.techniques.iter().any(|&t| t.contains("紫")) {
            self.techniques.iter().find(|&&t| t.contains("紫")).copied()
        } else if self.techniques.iter().any(|&t| t.contains("茈")) {
            self.techniques.iter().find(|&&t| t.contains("茈")).copied()
        } else {
            self.techniques.first().copied()
        }
    }

    // 戦闘力評価
    fn evaluate_combat_power(&self) -> String {
        match self.techniques.len() {
            0 => "初心者".to_string(),
            1..=2 => "中級者".to_string(),
            3..=4 => "上級者".to_string(),
            _ => "最強候補".to_string(),
        }
    }
}

fn main() {
    // すべての文字列リテラルは'staticライフタイム
    let mut gojo = Sorcerer::new("五条悟");

    gojo.add_technique("術式順転『蒼』");
    gojo.add_technique("術式反転『赤』");
    gojo.add_technique("虚式『茈』");
    gojo.add_technique("無下限呪術『紫』");

    println!("呪術師: {}", gojo.name);
    println!("呪力: {}", gojo.power);
    println!("評価: {}", gojo.evaluate_combat_power());

    if let Some(strongest) = gojo.get_strongest_technique() {
        println!("最強技: {}", strongest);
    }
}
```

## ライフタイム省略規則

Rustには3つの省略規則がある：

### 規則1: 各パラメータに独自のライフタイム

```rust
// これは...
fn analyze(text: &str) -> usize {
    text.len()
}

// 実際はこう解釈される
fn analyze_explicit<'a>(text: &'a str) -> usize {
    text.len()
}
```

### 規則2: 単一の入力ライフタイムが出力に適用

```rust
// これは...
fn get_first_word(text: &str) -> &str {
    text.split_whitespace().next().unwrap_or("")
}

// 実際はこう解釈される
fn get_first_word_explicit<'a>(text: &'a str) -> &'a str {
    text.split_whitespace().next().unwrap_or("")
}
```

### 規則3: &selfがある場合、そのライフタイムが出力に適用

```rust
struct Technique {
    name: String,
}

impl Technique {
    // これは...
    fn get_name(&self) -> &str {
        &self.name
    }

    // 実際はこう解釈される
    fn get_name_explicit<'a>(&'a self) -> &'a str {
        &self.name
    }
}
```

## 静的ライフタイム

### 'static ライフタイム

```rust
// 文字列リテラルは'staticライフタイム
fn get_default_technique() -> &'static str {
    "基本術式"  // プログラム終了まで有効
}

// 静的変数
static ULTIMATE_TECHNIQUE: &str = "無下限呪術『紫』";

fn main() {
    let default = get_default_technique();
    println!("デフォルト: {}", default);
    println!("究極技: {}", ULTIMATE_TECHNIQUE);
}
```

### Box<T>とライフタイム

```rust
// ヒープ上のデータは所有権で管理
fn create_owned_technique() -> Box<str> {
    let technique = String::from("術式順転『蒼』");
    technique.into_boxed_str()
}

fn main() {
    let owned = create_owned_technique();
    println!("所有された技: {}", owned);
}
```

## 実践例 - 呪術戦闘記録システム

```rust
use std::collections::HashMap;

// 戦闘記録
struct BattleRecord<'a> {
    combatants: Vec<&'a str>,
    techniques_used: Vec<&'a str>,
    winner: Option<&'a str>,
    duration: u32,
}

impl<'a> BattleRecord<'a> {
    fn new() -> Self {
        BattleRecord {
            combatants: Vec::new(),
            techniques_used: Vec::new(),
            winner: None,
            duration: 0,
        }
    }

    fn add_combatant(&mut self, name: &'a str) {
        self.combatants.push(name);
    }

    fn record_technique(&mut self, technique: &'a str) {
        self.techniques_used.push(technique);
    }

    fn set_winner(&mut self, winner: &'a str) {
        if self.combatants.contains(&winner) {
            self.winner = Some(winner);
        }
    }

    fn set_duration(&mut self, duration: u32) {
        self.duration = duration;
    }

    // 戦闘統計を返す
    fn get_statistics(&self) -> BattleStats<'a> {
        let mut technique_count = HashMap::new();
        for &technique in &self.techniques_used {
            *technique_count.entry(technique).or_insert(0) += 1;
        }

        BattleStats {
            record: self,
            technique_frequency: technique_count,
        }
    }
}

// 戦闘統計（元の記録への参照を保持）
struct BattleStats<'a> {
    record: &'a BattleRecord<'a>,
    technique_frequency: HashMap<&'a str, i32>,
}

impl<'a> BattleStats<'a> {
    fn most_used_technique(&self) -> Option<&'a str> {
        self.technique_frequency
            .iter()
            .max_by_key(|(_, &count)| count)
            .map(|(&technique, _)| technique)
    }

    fn generate_report(&self) -> String {
        let mut report = String::new();

        report.push_str("=== 戦闘レポート ===\n");
        report.push_str(&format!("参加者: {:?}\n", self.record.combatants));
        report.push_str(&format!("継続時間: {}分\n", self.record.duration));

        if let Some(winner) = self.record.winner {
            report.push_str(&format!("勝者: {}\n", winner));
        } else {
            report.push_str("勝者: 未決定\n");
        }

        report.push_str(&format!("使用技数: {}\n", self.record.techniques_used.len()));

        if let Some(most_used) = self.most_used_technique() {
            let count = self.technique_frequency[most_used];
            report.push_str(&format!("最多使用技: {} ({}回)\n", most_used, count));
        }

        report.push_str("\n=== 技使用履歴 ===\n");
        for (i, technique) in self.record.techniques_used.iter().enumerate() {
            report.push_str(&format!("{}. {}\n", i + 1, technique));
        }

        report
    }
}

// 戦闘シミュレーター
struct BattleSimulator;

impl BattleSimulator {
    fn simulate_gojo_vs_sukuna<'a>() -> BattleRecord<'a> {
        let mut record = BattleRecord::new();

        record.add_combatant("五条悟");
        record.add_combatant("両面宿儺");

        // 戦闘シミュレーション
        record.record_technique("術式順転『蒼』");
        record.record_technique("解");
        record.record_technique("術式反転『赤』");
        record.record_technique("捌");
        record.record_technique("虚式『茈』");
        record.record_technique("伏魔御廚子");
        record.record_technique("無下限呪術『紫』");
        record.record_technique("領域展開・無量空処");

        record.set_winner("五条悟");
        record.set_duration(15);

        record
    }

    fn simulate_student_battle<'a>(student1: &'a str, student2: &'a str) -> BattleRecord<'a> {
        let mut record = BattleRecord::new();

        record.add_combatant(student1);
        record.add_combatant(student2);

        record.record_technique("基本術式");
        record.record_technique("呪具操作");
        record.record_technique("強化術式");
        record.record_technique("防御術式");

        // ランダムな勝者（簡易版）
        record.set_winner(student1);
        record.set_duration(8);

        record
    }
}

fn main() {
    println!("=== 呪術戦闘記録システム ===\n");

    // 最強対決
    let gojo_battle = BattleSimulator::simulate_gojo_vs_sukuna();
    let stats = gojo_battle.get_statistics();
    println!("{}", stats.generate_report());

    // 学生戦闘
    let student_battle = BattleSimulator::simulate_student_battle("虎杖悠仁", "伏黒恵");
    let student_stats = student_battle.get_statistics();
    println!("{}", student_stats.generate_report());
}
```

## 高度なライフタイム

### ライフタイム境界

```rust
use std::fmt::Display;

// ライフタイム境界付きのジェネリック関数
fn display_longest<'a, T>(x: &'a T, y: &'a T) -> &'a T
where
    T: Display + PartialOrd,
{
    println!("比較中: {} vs {}", x, y);
    if x > y { x } else { y }
}

fn main() {
    let power1 = 1000;
    let power2 = 1500;

    let stronger = display_longest(&power1, &power2);
    println!("より強い呪力: {}", stronger);
}
```

### 高階ライフタイム境界

```rust
// 関数ポインタのライフタイム
fn apply_to_technique<'a, F>(technique: &'a str, func: F) -> String
where
    F: Fn(&str) -> String,
{
    func(technique)
}

fn enhance_technique(technique: &str) -> String {
    format!("強化された{}", technique)
}

fn main() {
    let technique = "術式順転『蒼』";
    let result = apply_to_technique(technique, enhance_technique);
    println!("{}", result);
}
```

## まとめ

ライフタイムの習得は完了だ！重要なポイント：

1. **ライフタイム注釈** - `'a` で参照の有効期間を明示
1. **省略規則** - 多くの場合は自動推論
1. **構造体のライフタイム** - 参照を保持する構造体
1. **'static** - プログラム全体で有効な参照
1. **借用チェッカー** - ダングリングポインタを防止

これで所有権、借用、ライフタイムの三位一体をマスターした。俺の無下限術式のように、完璧な制御ができるようになったな。

______________________________________________________________________

*「ライフタイムを理解すれば、参照の生死を自在に操れる」*
