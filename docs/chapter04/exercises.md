# 第4章 練習問題 - 領域展開の試練

## ジェネリクス・トレイトの総合演習

第4章で学んだ領域展開編の内容を実践で確認しよう。ジェネリクス、トレイト、高度なトレイト技術 - これらは俺の領域展開「無量空処」のように、**無限の可能性を包含する**究極の抽象化技術だ。

!!! tip "五条先生からのアドバイス"
    この章の問題は抽象化の極致だ。型システムを自在に操り、再利用可能で型安全なコードを書く力を養え。
    最初は複雑に感じるかもしれないが、マスターすれば無限の力を手に入れる。

## 初級編 - ジェネリクスの基本応用

### 問題1: 汎用的なコンテナシステム

様々な型の値を安全に管理できる汎用コンテナシステムを作成せよ。

<div class=\"exercise\">

```rust
// 以下の要件を満たすコンテナを実装せよ：
// 1. 任意の型Tの値を保持
// 2. 値の取得、設定、変換機能
// 3. Option型による安全なアクセス
// 4. イテレータ機能

struct Container<T> {
    // ここに実装
}

impl<T> Container<T> {
    // 必要なメソッドを実装
}

fn main() {
    // テスト用コード
    let mut power_container = Container::new();
    // 呪力値の管理をテスト

    let mut name_container = Container::new();
    // 呪術師名の管理をテスト
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
use std::collections::VecDeque;

#[derive(Debug)]
struct Container<T> {
    items: VecDeque<T>,
    capacity: Option<usize>,
}

impl<T> Container<T> {
    fn new() -> Self {
        Container {
            items: VecDeque::new(),
            capacity: None,
        }
    }

    fn with_capacity(capacity: usize) -> Self {
        Container {
            items: VecDeque::with_capacity(capacity),
            capacity: Some(capacity),
        }
    }

    fn push(&mut self, item: T) -> Result<(), T> {
        if let Some(cap) = self.capacity {
            if self.items.len() >= cap {
                return Err(item);
            }
        }
        self.items.push_back(item);
        Ok(())
    }

    fn pop(&mut self) -> Option<T> {
        self.items.pop_back()
    }

    fn peek(&self) -> Option<&T> {
        self.items.back()
    }

    fn peek_mut(&mut self) -> Option<&mut T> {
        self.items.back_mut()
    }

    fn len(&self) -> usize {
        self.items.len()
    }

    fn is_empty(&self) -> bool {
        self.items.is_empty()
    }

    fn clear(&mut self) {
        self.items.clear();
    }

    // 値の変換
    fn map<U, F>(self, f: F) -> Container<U>
    where
        F: Fn(T) -> U,
    {
        let mapped_items: VecDeque<U> = self.items.into_iter().map(f).collect();
        Container {
            items: mapped_items,
            capacity: self.capacity,
        }
    }

    // フィルタリング
    fn filter<F>(mut self, predicate: F) -> Container<T>
    where
        F: Fn(&T) -> bool,
    {
        self.items.retain(|item| predicate(item));
        self
    }

    // 安全なインデックスアクセス
    fn get(&self, index: usize) -> Option<&T> {
        self.items.get(index)
    }

    fn get_mut(&mut self, index: usize) -> Option<&mut T> {
        self.items.get_mut(index)
    }

    // イテレータ
    fn iter(&self) -> std::collections::vec_deque::Iter<T> {
        self.items.iter()
    }

    fn iter_mut(&mut self) -> std::collections::vec_deque::IterMut<T> {
        self.items.iter_mut()
    }
}

impl<T> IntoIterator for Container<T> {
    type Item = T;
    type IntoIter = std::collections::vec_deque::IntoIter<T>;

    fn into_iter(self) -> Self::IntoIter {
        self.items.into_iter()
    }
}

// より高度な操作
impl<T> Container<T>
where
    T: Clone,
{
    fn duplicate(&self) -> Container<T> {
        Container {
            items: self.items.clone(),
            capacity: self.capacity,
        }
    }
}

impl<T> Container<T>
where
    T: PartialOrd,
{
    fn max(&self) -> Option<&T> {
        self.items.iter().max()
    }

    fn min(&self) -> Option<&T> {
        self.items.iter().min()
    }

    fn sort(&mut self) {
        let mut vec: Vec<T> = self.items.drain(..).collect();
        vec.sort_by(|a, b| a.partial_cmp(b).unwrap_or(std::cmp::Ordering::Equal));
        self.items = VecDeque::from(vec);
    }
}

impl<T> Container<T>
where
    T: std::fmt::Display,
{
    fn display_all(&self) {
        for (i, item) in self.items.iter().enumerate() {
            println!(\"{}: {}\", i, item);
        }
    }
}

fn main() {
    println!(\"=== 汎用コンテナシステム ===\");

    // 呪力値の管理
    let mut power_container = Container::with_capacity(5);

    println!(\"呪力値を追加:\");
    for power in [1000, 1500, 800, 2000, 3000] {
        match power_container.push(power) {
            Ok(_) => println!(\"  {}を追加\", power),
            Err(rejected) => println!(\"  {}は容量オーバーで拒否\", rejected),
        }
    }

    println!(\"\\n呪力値一覧:\");
    power_container.display_all();

    println!(\"\\n統計情報:\");
    if let Some(max_power) = power_container.max() {
        println!(\"最大呪力: {}\", max_power);
    }
    if let Some(min_power) = power_container.min() {
        println!(\"最小呪力: {}\", min_power);
    }

    // 呪術師名の管理
    let mut name_container = Container::new();
    let names = [\"五条悟\", \"虎杖悠仁\", \"伏黒恵\", \"釘崎野薔薇\"];

    for name in names {
        name_container.push(name.to_string()).unwrap();
    }

    println!(\"\\n呪術師名一覧:\");
    name_container.display_all();

    // 変換操作
    let formatted_names = name_container.map(|name| format!(\"{}先輩\", name));
    println!(\"\\n敬称付き名前:\");
    formatted_names.display_all();

    // フィルタリング
    let mut long_names = Container::new();
    for name in names {
        long_names.push(name.to_string()).unwrap();
    }

    let filtered = long_names.filter(|name| name.len() >= 4);
    println!(\"\\n4文字以上の名前:\");
    filtered.display_all();

    // イテレータ使用
    println!(\"\\n呪力値の2倍計算:\");
    for power in power_container.iter() {
        println!(\"{} -> {}\", power, power * 2);
    }
}
```

</div>
</details>

### 問題2: ジェネリック計算システム

数値計算を汎用的に行うシステムを作成せよ。

<div class=\"exercise\">

```rust
// 以下の機能を持つ計算システムを実装せよ：
// 1. 任意の数値型での四則演算
// 2. 統計計算（平均、最大、最小）
// 3. 型変換機能
// 4. カスタム計算関数の適用

struct Calculator<T> {
    // ここに実装
}

// 必要なトレイト境界を考慮して実装
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
use std::ops::{Add, Sub, Mul, Div};
use std::fmt::Display;

#[derive(Debug)]
struct Calculator<T> {
    values: Vec<T>,
}

impl<T> Calculator<T> {
    fn new() -> Self {
        Calculator {
            values: Vec::new(),
        }
    }

    fn with_values(values: Vec<T>) -> Self {
        Calculator { values }
    }

    fn add_value(&mut self, value: T) {
        self.values.push(value);
    }

    fn len(&self) -> usize {
        self.values.len()
    }

    fn is_empty(&self) -> bool {
        self.values.is_empty()
    }

    fn clear(&mut self) {
        self.values.clear();
    }
}

// 基本計算操作
impl<T> Calculator<T>
where
    T: Add<Output = T> + Copy,
{
    fn sum(&self) -> Option<T> {
        if self.values.is_empty() {
            return None;
        }

        Some(self.values[1..].iter().fold(self.values[0], |acc, &x| acc + x))
    }
}

impl<T> Calculator<T>
where
    T: Add<Output = T> + Div<Output = T> + Copy + From<usize>,
{
    fn average(&self) -> Option<T> {
        if self.values.is_empty() {
            return None;
        }

        let sum = self.sum()?;
        let count = T::from(self.values.len());
        Some(sum / count)
    }
}

impl<T> Calculator<T>
where
    T: PartialOrd + Copy,
{
    fn max(&self) -> Option<T> {
        self.values.iter().max().copied()
    }

    fn min(&self) -> Option<T> {
        self.values.iter().min().copied()
    }

    fn range(&self) -> Option<(T, T)> {
        if self.values.is_empty() {
            return None;
        }

        Some((self.min()?, self.max()?))
    }
}

// 四則演算
impl<T> Calculator<T>
where
    T: Add<Output = T> + Sub<Output = T> + Mul<Output = T> + Div<Output = T> + Copy,
{
    fn element_wise_add(&self, other: &Calculator<T>) -> Calculator<T> {
        let results = self.values.iter()
            .zip(other.values.iter())
            .map(|(&a, &b)| a + b)
            .collect();

        Calculator { values: results }
    }

    fn element_wise_sub(&self, other: &Calculator<T>) -> Calculator<T> {
        let results = self.values.iter()
            .zip(other.values.iter())
            .map(|(&a, &b)| a - b)
            .collect();

        Calculator { values: results }
    }

    fn scalar_multiply(&self, scalar: T) -> Calculator<T> {
        let results = self.values.iter()
            .map(|&x| x * scalar)
            .collect();

        Calculator { values: results }
    }

    fn scalar_divide(&self, scalar: T) -> Calculator<T> {
        let results = self.values.iter()
            .map(|&x| x / scalar)
            .collect();

        Calculator { values: results }
    }
}

// 高度な操作
impl<T> Calculator<T>
where
    T: Copy,
{
    fn map<U, F>(&self, f: F) -> Calculator<U>
    where
        F: Fn(T) -> U,
    {
        let mapped_values = self.values.iter().map(|&x| f(x)).collect();
        Calculator { values: mapped_values }
    }

    fn filter<F>(&self, predicate: F) -> Calculator<T>
    where
        F: Fn(&T) -> bool,
    {
        let filtered_values = self.values.iter()
            .filter(|&x| predicate(x))
            .copied()
            .collect();

        Calculator { values: filtered_values }
    }

    fn reduce<F>(&self, initial: T, f: F) -> T
    where
        F: Fn(T, T) -> T,
    {
        self.values.iter().fold(initial, |acc, &x| f(acc, x))
    }
}

// 型変換
impl<T> Calculator<T> {
    fn convert<U, F>(self, converter: F) -> Calculator<U>
    where
        F: Fn(T) -> U,
    {
        let converted_values = self.values.into_iter().map(converter).collect();
        Calculator { values: converted_values }
    }
}

// 表示機能
impl<T> Calculator<T>
where
    T: Display,
{
    fn display(&self, label: &str) {
        println!(\"{}: [{}]\", label,
                 self.values.iter()
                     .map(|x| x.to_string())
                     .collect::<Vec<_>>()
                     .join(\", \"));
    }

    fn display_stats(&self)
    where
        T: Add<Output = T> + Div<Output = T> + PartialOrd + Copy + From<usize>,
    {
        println!(\"統計情報:\");
        println!(\"  要素数: {}\", self.len());

        if let Some(sum) = self.sum() {
            println!(\"  合計: {}\", sum);
        }

        if let Some(avg) = self.average() {
            println!(\"  平均: {}\", avg);
        }

        if let Some(max) = self.max() {
            println!(\"  最大: {}\", max);
        }

        if let Some(min) = self.min() {
            println!(\"  最小: {}\", min);
        }
    }
}

// 特化した機能
impl Calculator<f64> {
    fn standard_deviation(&self) -> Option<f64> {
        if self.values.len() < 2 {
            return None;
        }

        let avg = self.average()?;
        let variance = self.values.iter()
            .map(|&x| (x - avg).powi(2))
            .sum::<f64>() / (self.values.len() - 1) as f64;

        Some(variance.sqrt())
    }

    fn median(&self) -> Option<f64> {
        if self.values.is_empty() {
            return None;
        }

        let mut sorted = self.values.clone();
        sorted.sort_by(|a, b| a.partial_cmp(b).unwrap());

        let mid = sorted.len() / 2;

        if sorted.len() % 2 == 0 {
            Some((sorted[mid - 1] + sorted[mid]) / 2.0)
        } else {
            Some(sorted[mid])
        }
    }
}

fn main() {
    println!(\"=== ジェネリック計算システム ===\");

    // 整数での計算
    let mut int_calc = Calculator::new();
    let powers = [1000, 1500, 800, 2000, 1200];

    for power in powers {
        int_calc.add_value(power);
    }

    int_calc.display(\"呪力値\");
    int_calc.display_stats();

    println!();

    // 浮動小数点での計算
    let float_calc = Calculator::with_values(vec![1000.0, 1500.5, 800.3, 2000.7, 1200.1]);
    float_calc.display(\"精密呪力値\");
    float_calc.display_stats();

    if let Some(std_dev) = float_calc.standard_deviation() {
        println!(\"  標準偏差: {:.2}\", std_dev);
    }

    if let Some(median) = float_calc.median() {
        println!(\"  中央値: {:.2}\", median);
    }

    println!();

    // 計算操作
    let calc1 = Calculator::with_values(vec![100, 200, 300]);
    let calc2 = Calculator::with_values(vec![50, 100, 150]);

    calc1.display(\"計算1\");
    calc2.display(\"計算2\");

    let sum_result = calc1.element_wise_add(&calc2);
    sum_result.display(\"要素ごとの和\");

    let diff_result = calc1.element_wise_sub(&calc2);
    diff_result.display(\"要素ごとの差\");

    let scaled = calc1.scalar_multiply(2);
    scaled.display(\"2倍\");

    println!();

    // 変換操作
    let power_calc = Calculator::with_values(vec![1000, 1500, 2000]);
    power_calc.display(\"元の呪力\");

    // 整数から文字列への変換
    let string_calc = power_calc.convert(|x| {
        match x {
            x if x >= 2000 => format!(\"{}（特級）\", x),
            x if x >= 1500 => format!(\"{}（1級）\", x),
            x if x >= 1000 => format!(\"{}（2級）\", x),
            x => format!(\"{}（下級）\", x),
        }
    });

    string_calc.display(\"等級付き呪力\");

    // フィルタリング
    let high_powers = Calculator::with_values(vec![500, 1000, 1500, 2000, 3000]);
    high_powers.display(\"全呪力値\");

    let filtered = high_powers.filter(|&x| x >= 1500);
    filtered.display(\"1500以上の呪力\");

    // カスタム計算
    let doubled = high_powers.map(|x| x * 2);
    doubled.display(\"2倍した呪力\");

    let power_sum = high_powers.reduce(0, |acc, x| acc + x);
    println!(\"\\nカスタム合計計算: {}\", power_sum);
}
```

</div>
</details>

## 中級編 - トレイトの実践応用

### 問題3: 呪術戦闘システム

複数のトレイトを組み合わせた戦闘システムを実装せよ。

<div class=\"exercise\">

```rust
// 以下のトレイトを定義し、戦闘システムを実装せよ：
// 1. Entity - 基本的な存在情報
// 2. Combatant - 戦闘能力
// 3. TechniqueUser - 術式使用能力
// 4. Healable - 回復能力
// 5. Displayable - 表示機能

// さらに、これらのトレイトを実装する具体的な型を作成し、
// 戦闘シミュレーションを行うシステムを構築せよ。

fn main() {
    // 戦闘システムのテスト
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
use std::collections::HashMap;
use std::fmt;

// 基本的な存在情報
trait Entity {
    fn name(&self) -> &str;
    fn entity_type(&self) -> &str;
    fn level(&self) -> u32;
    fn is_alive(&self) -> bool;
}

// 戦闘能力
trait Combatant: Entity {
    fn max_health(&self) -> i32;
    fn current_health(&self) -> i32;
    fn attack_power(&self) -> i32;
    fn defense_power(&self) -> i32;

    fn take_damage(&mut self, damage: i32) -> i32;
    fn calculate_damage(&self, target: &dyn Combatant) -> i32 {
        let base_damage = self.attack_power();
        let defense = target.defense_power();
        (base_damage - defense / 2).max(1)
    }
}

// 術式使用能力
trait TechniqueUser: Entity {
    fn available_techniques(&self) -> &[String];
    fn mana(&self) -> i32;
    fn max_mana(&self) -> i32;

    fn use_technique(&mut self, technique_name: &str, target: &mut dyn Combatant) -> Result<String, String>;
    fn recover_mana(&mut self, amount: i32);
}

// 回復能力
trait Healable: Combatant {
    fn heal(&mut self, amount: i32) -> i32;
    fn can_heal(&self) -> bool {
        self.current_health() < self.max_health()
    }
}

// 表示機能
trait Displayable {
    fn display_status(&self) -> String;
    fn display_summary(&self) -> String;
}

// 呪術師実装
#[derive(Debug, Clone)]
struct Sorcerer {
    name: String,
    level: u32,
    max_health: i32,
    current_health: i32,
    attack_power: i32,
    defense_power: i32,
    max_mana: i32,
    current_mana: i32,
    techniques: Vec<String>,
    technique_costs: HashMap<String, i32>,
}

impl Sorcerer {
    fn new(name: &str, level: u32, health: i32, attack: i32, defense: i32, mana: i32) -> Self {
        let mut sorcerer = Sorcerer {
            name: name.to_string(),
            level,
            max_health: health,
            current_health: health,
            attack_power: attack,
            defense_power: defense,
            max_mana: mana,
            current_mana: mana,
            techniques: Vec::new(),
            technique_costs: HashMap::new(),
        };

        // デフォルト術式
        sorcerer.learn_technique(\"基本攻撃\", 10);
        sorcerer.learn_technique(\"強化攻撃\", 25);

        sorcerer
    }

    fn learn_technique(&mut self, name: &str, cost: i32) {
        self.techniques.push(name.to_string());
        self.technique_costs.insert(name.to_string(), cost);
    }
}

impl Entity for Sorcerer {
    fn name(&self) -> &str {
        &self.name
    }

    fn entity_type(&self) -> &str {
        \"呪術師\"
    }

    fn level(&self) -> u32 {
        self.level
    }

    fn is_alive(&self) -> bool {
        self.current_health > 0
    }
}

impl Combatant for Sorcerer {
    fn max_health(&self) -> i32 {
        self.max_health
    }

    fn current_health(&self) -> i32 {
        self.current_health
    }

    fn attack_power(&self) -> i32 {
        self.attack_power
    }

    fn defense_power(&self) -> i32 {
        self.defense_power
    }

    fn take_damage(&mut self, damage: i32) -> i32 {
        let actual_damage = damage.min(self.current_health);
        self.current_health -= actual_damage;
        actual_damage
    }
}

impl TechniqueUser for Sorcerer {
    fn available_techniques(&self) -> &[String] {
        &self.techniques
    }

    fn mana(&self) -> i32 {
        self.current_mana
    }

    fn max_mana(&self) -> i32 {
        self.max_mana
    }

    fn use_technique(&mut self, technique_name: &str, target: &mut dyn Combatant) -> Result<String, String> {
        let cost = self.technique_costs.get(technique_name)
            .ok_or_else(|| format!(\"{}は{}を知りません\", self.name, technique_name))?;

        if self.current_mana < *cost {
            return Err(format!(\"{}のマナが不足しています（必要: {}, 現在: {}）\",
                             self.name, cost, self.current_mana));
        }

        self.current_mana -= cost;

        let damage = match technique_name {
            \"基本攻撃\" => self.calculate_damage(target),
            \"強化攻撃\" => self.calculate_damage(target) * 2,
            \"回復術\" => {
                if let Some(healable_target) = (target as &mut dyn std::any::Any).downcast_mut::<Sorcerer>() {
                    let heal_amount = self.attack_power() / 2;
                    healable_target.heal(heal_amount);
                    return Ok(format!(\"{}が{}を{}回復させた\", self.name, target.name(), heal_amount));
                } else {
                    return Err(\"回復対象が無効です\".to_string());
                }
            },
            _ => self.calculate_damage(target),
        };

        let actual_damage = target.take_damage(damage);
        Ok(format!(\"{}が{}で{}に{}ダメージ\",
                  self.name, technique_name, target.name(), actual_damage))
    }

    fn recover_mana(&mut self, amount: i32) {
        self.current_mana = (self.current_mana + amount).min(self.max_mana);
    }
}

impl Healable for Sorcerer {
    fn heal(&mut self, amount: i32) -> i32 {
        let actual_heal = amount.min(self.max_health - self.current_health);
        self.current_health += actual_heal;
        actual_heal
    }
}

impl Displayable for Sorcerer {
    fn display_status(&self) -> String {
        format!(
            \"=== {} ===\\nレベル: {}\\nHP: {}/{}\\nマナ: {}/{}\\n攻撃力: {}\\n防御力: {}\\n習得術式: {:?}\\n状態: {}\",
            self.name,
            self.level,
            self.current_health, self.max_health,
            self.current_mana, self.max_mana,
            self.attack_power,
            self.defense_power,
            self.techniques,
            if self.is_alive() { \"生存\" } else { \"戦闘不能\" }
        )
    }

    fn display_summary(&self) -> String {
        format!(\"{} (Lv.{}, HP: {}/{}, マナ: {}/{})\",
                self.name, self.level,
                self.current_health, self.max_health,
                self.current_mana, self.max_mana)
    }
}

impl fmt::Display for Sorcerer {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, \"{}\", self.display_summary())
    }
}

// 呪霊実装
#[derive(Debug, Clone)]
struct Curse {
    name: String,
    grade: String,
    level: u32,
    max_health: i32,
    current_health: i32,
    attack_power: i32,
    defense_power: i32,
}

impl Curse {
    fn new(name: &str, grade: &str, level: u32, health: i32, attack: i32, defense: i32) -> Self {
        Curse {
            name: name.to_string(),
            grade: grade.to_string(),
            level,
            max_health: health,
            current_health: health,
            attack_power: attack,
            defense_power: defense,
        }
    }
}

impl Entity for Curse {
    fn name(&self) -> &str {
        &self.name
    }

    fn entity_type(&self) -> &str {
        \"呪霊\"
    }

    fn level(&self) -> u32 {
        self.level
    }

    fn is_alive(&self) -> bool {
        self.current_health > 0
    }
}

impl Combatant for Curse {
    fn max_health(&self) -> i32 {
        self.max_health
    }

    fn current_health(&self) -> i32 {
        self.current_health
    }

    fn attack_power(&self) -> i32 {
        self.attack_power
    }

    fn defense_power(&self) -> i32 {
        self.defense_power
    }

    fn take_damage(&mut self, damage: i32) -> i32 {
        let actual_damage = damage.min(self.current_health);
        self.current_health -= actual_damage;
        actual_damage
    }
}

impl Displayable for Curse {
    fn display_status(&self) -> String {
        format!(
            \"=== {} ===\\n等級: {}\\nレベル: {}\\nHP: {}/{}\\n攻撃力: {}\\n防御力: {}\\n状態: {}\",
            self.name,
            self.grade,
            self.level,
            self.current_health, self.max_health,
            self.attack_power,
            self.defense_power,
            if self.is_alive() { \"生存\" } else { \"祓われた\" }
        )
    }

    fn display_summary(&self) -> String {
        format!(\"{} ({}級, Lv.{}, HP: {}/{})\",
                self.name, self.grade, self.level,
                self.current_health, self.max_health)
    }
}

impl fmt::Display for Curse {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, \"{}\", self.display_summary())
    }
}

// 戦闘システム
struct BattleSystem;

impl BattleSystem {
    fn one_vs_one(
        fighter1: &mut dyn Combatant,
        fighter2: &mut dyn Combatant
    ) -> String {
        let mut log = String::new();
        let mut round = 1;

        log.push_str(&format!(\"=== 1対1戦闘: {} vs {} ===\\n\",
                             fighter1.name(), fighter2.name()));

        while fighter1.is_alive() && fighter2.is_alive() && round <= 10 {
            log.push_str(&format!(\"\\n--- ラウンド {} ---\\n\", round));

            // ファイター1の攻撃
            if fighter1.is_alive() {
                let damage = fighter1.calculate_damage(fighter2);
                let actual_damage = fighter2.take_damage(damage);
                log.push_str(&format!(\"{}が{}を攻撃！{}ダメージ\\n\",
                                     fighter1.name(), fighter2.name(), actual_damage));

                if !fighter2.is_alive() {
                    log.push_str(&format!(\"{}が倒れた！\\n\", fighter2.name()));
                    break;
                }
            }

            // ファイター2の攻撃
            if fighter2.is_alive() {
                let damage = fighter2.calculate_damage(fighter1);
                let actual_damage = fighter1.take_damage(damage);
                log.push_str(&format!(\"{}が{}を攻撃！{}ダメージ\\n\",
                                     fighter2.name(), fighter1.name(), actual_damage));

                if !fighter1.is_alive() {
                    log.push_str(&format!(\"{}が倒れた！\\n\", fighter1.name()));
                    break;
                }
            }

            round += 1;
        }

        // 勝者の発表
        if fighter1.is_alive() && !fighter2.is_alive() {
            log.push_str(&format!(\"\\n勝者: {}！\\n\", fighter1.name()));
        } else if fighter2.is_alive() && !fighter1.is_alive() {
            log.push_str(&format!(\"\\n勝者: {}！\\n\", fighter2.name()));
        } else if round > 10 {
            log.push_str(\"\\n時間切れで引き分け！\\n\");
        } else {
            log.push_str(\"\\n両者戦闘不能...\\n\");
        }

        log
    }

    fn technique_battle<T1, T2>(
        user: &mut T1,
        target: &mut T2,
        technique: &str
    ) -> String
    where
        T1: TechniqueUser + Combatant,
        T2: Combatant,
    {
        match user.use_technique(technique, target) {
            Ok(result) => result,
            Err(error) => error,
        }
    }
}

fn main() {
    println!(\"=== 呪術戦闘システム ===\");

    // キャラクター作成
    let mut gojo = Sorcerer::new(\"五条悟\", 10, 2000, 300, 200, 500);
    gojo.learn_technique(\"無下限術式\", 50);
    gojo.learn_technique(\"領域展開\", 200);

    let mut yuji = Sorcerer::new(\"虎杖悠仁\", 5, 1500, 250, 150, 300);
    yuji.learn_technique(\"黒閃\", 40);

    let mut special_curse = Curse::new(\"特級呪霊\", \"特級\", 8, 1800, 280, 180);

    // ステータス表示
    println!(\"{}\", gojo.display_status());
    println!();
    println!(\"{}\", yuji.display_status());
    println!();
    println!(\"{}\", special_curse.display_status());
    println!();

    // 術式戦闘
    println!(\"=== 術式使用テスト ===\");
    let result = BattleSystem::technique_battle(&mut gojo, &mut special_curse, \"無下限術式\");
    println!(\"{}\", result);

    let result = BattleSystem::technique_battle(&mut yuji, &mut gojo, \"黒閃\");
    println!(\"{}\", result);

    // マナ不足のテスト
    let result = BattleSystem::technique_battle(&mut gojo, &mut special_curse, \"領域展開\");
    println!(\"{}\", result);

    println!();

    // 1対1戦闘
    let mut gojo_clone = gojo.clone();
    let mut curse_clone = special_curse.clone();

    let battle_log = BattleSystem::one_vs_one(&mut gojo_clone, &mut curse_clone);
    println!(\"{}\", battle_log);

    // 戦闘後ステータス
    println!(\"=== 戦闘後ステータス ===\");
    println!(\"{}\", gojo_clone.display_summary());
    println!(\"{}\", curse_clone.display_summary());

    // 回復テスト
    if gojo_clone.can_heal() {
        let healed = gojo_clone.heal(200);
        println!(\"\\n{}が{}回復した\", gojo_clone.name(), healed);
        println!(\"{}\", gojo_clone.display_summary());
    }

    // マナ回復
    gojo_clone.recover_mana(100);
    println!(\"マナ回復後: {}\", gojo_clone.display_summary());
}
```

</div>
</details>

### 問題4: 型安全な設定管理システム

異なる型の設定値を安全に管理するシステムを作成せよ。

<div class=\"exercise\">

```rust
// 以下の要件を満たす設定管理システムを実装せよ：
// 1. 任意の型の設定値を保存・取得
// 2. 型安全性の保証
// 3. デフォルト値の提供
// 4. 設定の検証機能
// 5. 設定のシリアライゼーション

use std::collections::HashMap;
use std::any::{Any, TypeId};

struct ConfigManager {
    // ここに実装
}

// 設定値のトレイト定義も含めて実装
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
use std::collections::HashMap;
use std::any::{Any, TypeId};
use std::fmt;

// 設定値のトレイト
trait ConfigValue: Any + fmt::Debug + Clone {
    fn validate(&self) -> Result<(), String> {
        Ok(())  // デフォルトでは常に有効
    }

    fn as_any(&self) -> &dyn Any;
    fn clone_box(&self) -> Box<dyn ConfigValue>;
}

impl<T: Any + fmt::Debug + Clone> ConfigValue for T {
    fn as_any(&self) -> &dyn Any {
        self
    }

    fn clone_box(&self) -> Box<dyn ConfigValue> {
        Box::new(self.clone())
    }
}

// 設定エントリ
#[derive(Debug, Clone)]
struct ConfigEntry {
    value: Box<dyn ConfigValue>,
    description: String,
    required: bool,
}

impl ConfigEntry {
    fn new<T: ConfigValue>(value: T, description: &str, required: bool) -> Self {
        ConfigEntry {
            value: Box::new(value),
            description: description.to_string(),
            required,
        }
    }
}

// 設定管理システム
struct ConfigManager {
    configs: HashMap<String, ConfigEntry>,
    defaults: HashMap<String, Box<dyn ConfigValue>>,
}

impl ConfigManager {
    fn new() -> Self {
        ConfigManager {
            configs: HashMap::new(),
            defaults: HashMap::new(),
        }
    }

    // 設定値の登録
    fn set<T: ConfigValue>(&mut self, key: &str, value: T, description: &str, required: bool) -> Result<(), String> {
        // バリデーション
        value.validate()?;

        let entry = ConfigEntry::new(value, description, required);
        self.configs.insert(key.to_string(), entry);
        Ok(())
    }

    // 設定値の取得
    fn get<T: ConfigValue>(&self, key: &str) -> Result<T, String> {
        let entry = self.configs.get(key)
            .or_else(|| {
                // デフォルト値を確認
                if let Some(default) = self.defaults.get(key) {
                    Some(&ConfigEntry {
                        value: default.clone_box(),
                        description: \"Default value\".to_string(),
                        required: false,
                    })
                } else {
                    None
                }
            })
            .ok_or_else(|| format!(\"設定 '{}' が見つかりません\", key))?;

        entry.value.as_any()
            .downcast_ref::<T>()
            .ok_or_else(|| format!(\"設定 '{}' の型が一致しません\", key))?
            .clone()
            .validate()
            .map_err(|e| format!(\"設定 '{}' の検証エラー: {}\", key, e))?;

        Ok(entry.value.as_any()
            .downcast_ref::<T>()
            .unwrap()
            .clone())
    }

    // デフォルト値の設定
    fn set_default<T: ConfigValue>(&mut self, key: &str, value: T) {
        self.defaults.insert(key.to_string(), Box::new(value));
    }

    // 設定の存在確認
    fn exists(&self, key: &str) -> bool {
        self.configs.contains_key(key) || self.defaults.contains_key(key)
    }

    // 設定の削除
    fn remove(&mut self, key: &str) -> Result<(), String> {
        let entry = self.configs.get(key)
            .ok_or_else(|| format!(\"設定 '{}' が見つかりません\", key))?;

        if entry.required {
            return Err(format!(\"必須設定 '{}' は削除できません\", key));
        }

        self.configs.remove(key);
        Ok(())
    }

    // 全設定の検証
    fn validate_all(&self) -> Result<(), Vec<String>> {
        let mut errors = Vec::new();

        for (key, entry) in &self.configs {
            if let Err(e) = entry.value.validate() {
                errors.push(format!(\"{}: {}\", key, e));
            }
        }

        if errors.is_empty() {
            Ok(())
        } else {
            Err(errors)
        }
    }

    // 設定一覧の取得
    fn list_configs(&self) -> Vec<(String, String, bool)> {
        self.configs.iter()
            .map(|(key, entry)| {
                (key.clone(), entry.description.clone(), entry.required)
            })
            .collect()
    }

    // 型別設定取得
    fn get_configs_of_type<T: ConfigValue>(&self) -> HashMap<String, T> {
        let mut result = HashMap::new();
        let type_id = TypeId::of::<T>();

        for (key, entry) in &self.configs {
            if entry.value.as_any().type_id() == type_id {
                if let Some(value) = entry.value.as_any().downcast_ref::<T>() {
                    result.insert(key.clone(), value.clone());
                }
            }
        }

        result
    }

    // 設定のエクスポート（簡易版）
    fn export_config(&self) -> String {
        let mut output = String::from(\"# 設定ファイル\\n\\n\");

        for (key, entry) in &self.configs {
            output.push_str(&format!(\"# {}\\n\", entry.description));
            output.push_str(&format!(\"# 必須: {}\\n\", entry.required));
            output.push_str(&format!(\"{} = {:?}\\n\\n\", key, entry.value));
        }

        output
    }
}

impl fmt::Debug for ConfigManager {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, \"ConfigManager {{ configs: {} entries }}\", self.configs.len())
    }
}

// カスタム設定型の例
#[derive(Debug, Clone, PartialEq)]
struct PowerSettings {
    max_power: i32,
    regeneration_rate: f64,
    power_type: String,
}

impl PowerSettings {
    fn new(max_power: i32, regeneration_rate: f64, power_type: &str) -> Self {
        PowerSettings {
            max_power,
            regeneration_rate,
            power_type: power_type.to_string(),
        }
    }
}

impl ConfigValue for PowerSettings {
    fn validate(&self) -> Result<(), String> {
        if self.max_power <= 0 {
            return Err(\"最大呪力は正の値である必要があります\".to_string());
        }

        if self.regeneration_rate < 0.0 || self.regeneration_rate > 1.0 {
            return Err(\"回復率は0.0から1.0の間である必要があります\".to_string());
        }

        let valid_types = [\"物理\", \"呪術\", \"特殊\"];
        if !valid_types.contains(&self.power_type.as_str()) {
            return Err(format!(\"無効な呪力タイプ: {}。有効: {:?}\",
                             self.power_type, valid_types));
        }

        Ok(())
    }

    fn as_any(&self) -> &dyn Any {
        self
    }

    fn clone_box(&self) -> Box<dyn ConfigValue> {
        Box::new(self.clone())
    }
}

// 検証付きの数値型
#[derive(Debug, Clone, PartialEq)]
struct ValidatedInt {
    value: i32,
    min: i32,
    max: i32,
}

impl ValidatedInt {
    fn new(value: i32, min: i32, max: i32) -> Self {
        ValidatedInt { value, min, max }
    }
}

impl ConfigValue for ValidatedInt {
    fn validate(&self) -> Result<(), String> {
        if self.value < self.min || self.value > self.max {
            Err(format!(\"値{}は範囲{}..{}を超えています\",
                       self.value, self.min, self.max))
        } else {
            Ok(())
        }
    }

    fn as_any(&self) -> &dyn Any {
        self
    }

    fn clone_box(&self) -> Box<dyn ConfigValue> {
        Box::new(self.clone())
    }
}

fn main() {
    println!(\"=== 型安全設定管理システム ===\");

    let mut config = ConfigManager::new();

    // デフォルト値の設定
    config.set_default(\"debug_mode\", false);
    config.set_default(\"max_connections\", 100i32);
    config.set_default(\"timeout\", 30.0f64);

    // 基本設定の登録
    config.set(\"debug_mode\", true, \"デバッグモードの有効/無効\", false).unwrap();
    config.set(\"max_connections\", 200i32, \"最大接続数\", true).unwrap();
    config.set(\"server_name\", \"呪術高専サーバー\".to_string(), \"サーバー名\", true).unwrap();
    config.set(\"timeout\", 45.5f64, \"タイムアウト（秒）\", false).unwrap();

    // カスタム設定型の登録
    let power_config = PowerSettings::new(3000, 0.1, \"呪術\");
    config.set(\"gojo_power\", power_config, \"五条悟の呪力設定\", true).unwrap();

    let validated_level = ValidatedInt::new(10, 1, 100);
    config.set(\"player_level\", validated_level, \"プレイヤーレベル（1-100）\", false).unwrap();

    // 設定値の取得
    println!(\"=== 設定値の取得 ===\");

    let debug: bool = config.get(\"debug_mode\").unwrap();
    println!(\"デバッグモード: {}\", debug);

    let max_conn: i32 = config.get(\"max_connections\").unwrap();
    println!(\"最大接続数: {}\", max_conn);

    let server_name: String = config.get(\"server_name\").unwrap();
    println!(\"サーバー名: {}\", server_name);

    let gojo_power: PowerSettings = config.get(\"gojo_power\").unwrap();
    println!(\"五条悟の呪力設定: {:?}\", gojo_power);

    // デフォルト値の取得テスト
    let default_timeout: f64 = config.get(\"default_timeout\").unwrap_or(60.0);
    println!(\"デフォルトタイムアウト: {}\", default_timeout);

    println!();

    // 設定一覧
    println!(\"=== 設定一覧 ===\");
    for (key, description, required) in config.list_configs() {
        let req_mark = if required { \"[必須]\" } else { \"[任意]\" };
        println!(\"{} {}: {}\", req_mark, key, description);
    }

    println!();

    // 型別設定取得
    println!(\"=== 整数型設定 ===\");
    let int_configs: HashMap<String, i32> = config.get_configs_of_type();
    for (key, value) in int_configs {
        println!(\"{}: {}\", key, value);
    }

    println!();

    // バリデーションテスト
    println!(\"=== バリデーションテスト ===\");

    // 無効な設定の登録を試行
    let invalid_power = PowerSettings::new(-100, 1.5, \"無効タイプ\");
    match config.set(\"invalid_power\", invalid_power, \"無効な呪力設定\", false) {
        Ok(_) => println!(\"無効設定が登録されました（予期しない）\"),
        Err(e) => println!(\"予期されたエラー: {}\", e),
    }

    let invalid_level = ValidatedInt::new(150, 1, 100);
    match config.set(\"invalid_level\", invalid_level, \"無効レベル\", false) {
        Ok(_) => println!(\"無効レベルが登録されました（予期しない）\"),
        Err(e) => println!(\"予期されたエラー: {}\", e),
    }

    // 全設定の検証
    match config.validate_all() {
        Ok(_) => println!(\"✓ 全設定が有効です\"),
        Err(errors) => {
            println!(\"✗ 設定エラー:\");
            for error in errors {
                println!(\"  {}\", error);
            }
        }
    }

    println!();

    // 設定のエクスポート
    println!(\"=== 設定エクスポート ===\");
    println!(\"{}\", config.export_config());

    // 型安全性のテスト
    println!(\"=== 型安全性テスト ===\");

    // 正しい型での取得
    match config.get::<bool>(\"debug_mode\") {
        Ok(value) => println!(\"✓ bool取得成功: {}\", value),
        Err(e) => println!(\"✗ bool取得失敗: {}\", e),
    }

    // 間違った型での取得
    match config.get::<String>(\"debug_mode\") {
        Ok(value) => println!(\"✓ String取得成功: {}\", value),
        Err(e) => println!(\"✗ 予期されたエラー: {}\", e),
    }

    // 存在しない設定の取得
    match config.get::<i32>(\"nonexistent\") {
        Ok(value) => println!(\"✓ 存在しない設定取得: {}\", value),
        Err(e) => println!(\"✗ 予期されたエラー: {}\", e),
    }

    println!(\"\\n=== 設定管理システムテスト完了 ===\");
}
```

</div>
</details>

## 上級編 - 高度なトレイト応用

### 問題5: 関数型プログラミングライブラリ

関数型プログラミングの概念を取り入れた汎用ライブラリを実装せよ。

<div class=\"exercise\">

```rust
// 以下の関数型プログラミング概念を実装せよ：
// 1. Functor（マップ可能）
// 2. Applicative（適用可能）
// 3. Monad（モナド）
// 4. Foldable（畳み込み可能）
// 5. Traversable（走査可能）

// これらのトレイトを定義し、適切な型に実装せよ
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
// Functor trait - map操作が可能
trait Functor<A> {
    type Output<B>;

    fn map<B, F>(self, f: F) -> Self::Output<B>
    where
        F: FnOnce(A) -> B;
}

// Applicative trait - 関数の適用が可能
trait Applicative<A>: Functor<A> {
    fn pure(value: A) -> Self;

    fn apply<B, F>(self, f: Self::Output<F>) -> Self::Output<B>
    where
        F: FnOnce(A) -> B;
}

// Monad trait - flat_map操作が可能
trait Monad<A>: Applicative<A> {
    fn flat_map<B, F>(self, f: F) -> Self::Output<B>
    where
        F: FnOnce(A) -> Self::Output<B>;

    fn and_then<B, F>(self, f: F) -> Self::Output<B>
    where
        F: FnOnce(A) -> Self::Output<B>,
    {
        self.flat_map(f)
    }
}

// Foldable trait - 畳み込み操作が可能
trait Foldable<A> {
    fn fold<B, F>(self, init: B, f: F) -> B
    where
        F: Fn(B, A) -> B;

    fn fold_right<B, F>(self, init: B, f: F) -> B
    where
        F: Fn(A, B) -> B;

    fn reduce<F>(self, f: F) -> Option<A>
    where
        F: Fn(A, A) -> A;
}

// Traversable trait - 構造を保持した走査が可能
trait Traversable<A>: Functor<A> + Foldable<A> {
    fn traverse<B, F, M>(self, f: F) -> M
    where
        F: Fn(A) -> M,
        M: Monad<B>;
}

// Maybe型の実装（Option風）
#[derive(Debug, Clone, PartialEq)]
enum Maybe<T> {
    Just(T),
    Nothing,
}

impl<T> Maybe<T> {
    fn is_just(&self) -> bool {
        matches!(self, Maybe::Just(_))
    }

    fn is_nothing(&self) -> bool {
        matches!(self, Maybe::Nothing)
    }

    fn unwrap_or(self, default: T) -> T {
        match self {
            Maybe::Just(value) => value,
            Maybe::Nothing => default,
        }
    }

    fn filter<F>(self, predicate: F) -> Maybe<T>
    where
        F: FnOnce(&T) -> bool,
    {
        match self {
            Maybe::Just(ref value) if predicate(value) => self,
            _ => Maybe::Nothing,
        }
    }
}

impl<A> Functor<A> for Maybe<A> {
    type Output<B> = Maybe<B>;

    fn map<B, F>(self, f: F) -> Self::Output<B>
    where
        F: FnOnce(A) -> B,
    {
        match self {
            Maybe::Just(value) => Maybe::Just(f(value)),
            Maybe::Nothing => Maybe::Nothing,
        }
    }
}

impl<A> Applicative<A> for Maybe<A> {
    fn pure(value: A) -> Self {
        Maybe::Just(value)
    }

    fn apply<B, F>(self, f: Self::Output<F>) -> Self::Output<B>
    where
        F: FnOnce(A) -> B,
    {
        match (self, f) {
            (Maybe::Just(value), Maybe::Just(func)) => Maybe::Just(func(value)),
            _ => Maybe::Nothing,
        }
    }
}

impl<A> Monad<A> for Maybe<A> {
    fn flat_map<B, F>(self, f: F) -> Self::Output<B>
    where
        F: FnOnce(A) -> Self::Output<B>,
    {
        match self {
            Maybe::Just(value) => f(value),
            Maybe::Nothing => Maybe::Nothing,
        }
    }
}

impl<A> Foldable<A> for Maybe<A> {
    fn fold<B, F>(self, init: B, f: F) -> B
    where
        F: Fn(B, A) -> B,
    {
        match self {
            Maybe::Just(value) => f(init, value),
            Maybe::Nothing => init,
        }
    }

    fn fold_right<B, F>(self, init: B, f: F) -> B
    where
        F: Fn(A, B) -> B,
    {
        match self {
            Maybe::Just(value) => f(value, init),
            Maybe::Nothing => init,
        }
    }

    fn reduce<F>(self, _f: F) -> Option<A>
    where
        F: Fn(A, A) -> A,
    {
        match self {
            Maybe::Just(value) => Some(value),
            Maybe::Nothing => None,
        }
    }
}

// List型の実装
#[derive(Debug, Clone, PartialEq)]
struct List<T> {
    items: Vec<T>,
}

impl<T> List<T> {
    fn new() -> Self {
        List { items: Vec::new() }
    }

    fn from_vec(items: Vec<T>) -> Self {
        List { items }
    }

    fn push(&mut self, item: T) {
        self.items.push(item);
    }

    fn len(&self) -> usize {
        self.items.len()
    }

    fn is_empty(&self) -> bool {
        self.items.is_empty()
    }

    fn head(&self) -> Maybe<&T> {
        match self.items.first() {
            Some(item) => Maybe::Just(item),
            None => Maybe::Nothing,
        }
    }

    fn tail(&self) -> List<T>
    where
        T: Clone,
    {
        if self.items.len() <= 1 {
            List::new()
        } else {
            List::from_vec(self.items[1..].to_vec())
        }
    }
}

impl<A> Functor<A> for List<A> {
    type Output<B> = List<B>;

    fn map<B, F>(self, f: F) -> Self::Output<B>
    where
        F: FnOnce(A) -> B,
    {
        // FnOnceは一度しか呼べないので、ベクターの各要素に適用するためにクロージャを作る
        let mut result = Vec::new();
        let mut f = Some(f);

        for (i, item) in self.items.into_iter().enumerate() {
            if i == 0 {
                if let Some(func) = f.take() {
                    result.push(func(item));
                }
            } else {
                // 最初の要素以降は同じ変換を適用（実際にはFnではなくFnOnceなので制限あり）
                break;
            }
        }

        List::from_vec(result)
    }
}

// より実用的なマップ実装
impl<T> List<T> {
    fn map_fn<U, F>(&self, f: F) -> List<U>
    where
        F: Fn(&T) -> U,
    {
        let mapped_items: Vec<U> = self.items.iter().map(f).collect();
        List::from_vec(mapped_items)
    }

    fn flat_map_fn<U, F>(&self, f: F) -> List<U>
    where
        F: Fn(&T) -> List<U>,
    {
        let mut result = Vec::new();
        for item in &self.items {
            result.extend(f(item).items);
        }
        List::from_vec(result)
    }

    fn filter<F>(&self, predicate: F) -> List<T>
    where
        F: Fn(&T) -> bool,
        T: Clone,
    {
        let filtered_items: Vec<T> = self.items.iter()
            .filter(|item| predicate(item))
            .cloned()
            .collect();
        List::from_vec(filtered_items)
    }
}

impl<A> Foldable<A> for List<A> {
    fn fold<B, F>(self, init: B, f: F) -> B
    where
        F: Fn(B, A) -> B,
    {
        self.items.into_iter().fold(init, f)
    }

    fn fold_right<B, F>(self, init: B, f: F) -> B
    where
        F: Fn(A, B) -> B,
    {
        self.items.into_iter().rev().fold(init, |acc, x| f(x, acc))
    }

    fn reduce<F>(self, f: F) -> Option<A>
    where
        F: Fn(A, A) -> A,
    {
        self.items.into_iter().reduce(f)
    }
}

// Either型の実装（Result風）
#[derive(Debug, Clone, PartialEq)]
enum Either<L, R> {
    Left(L),
    Right(R),
}

impl<L, R> Either<L, R> {
    fn is_left(&self) -> bool {
        matches!(self, Either::Left(_))
    }

    fn is_right(&self) -> bool {
        matches!(self, Either::Right(_))
    }

    fn left(self) -> Option<L> {
        match self {
            Either::Left(value) => Some(value),
            Either::Right(_) => None,
        }
    }

    fn right(self) -> Option<R> {
        match self {
            Either::Left(_) => None,
            Either::Right(value) => Some(value),
        }
    }
}

impl<L, A> Functor<A> for Either<L, A> {
    type Output<B> = Either<L, B>;

    fn map<B, F>(self, f: F) -> Self::Output<B>
    where
        F: FnOnce(A) -> B,
    {
        match self {
            Either::Left(left) => Either::Left(left),
            Either::Right(right) => Either::Right(f(right)),
        }
    }
}

impl<L, A> Applicative<A> for Either<L, A> {
    fn pure(value: A) -> Self {
        Either::Right(value)
    }

    fn apply<B, F>(self, f: Self::Output<F>) -> Self::Output<B>
    where
        F: FnOnce(A) -> B,
    {
        match (self, f) {
            (Either::Right(value), Either::Right(func)) => Either::Right(func(value)),
            (Either::Left(left), _) => Either::Left(left),
            (_, Either::Left(left)) => Either::Left(left),
        }
    }
}

impl<L, A> Monad<A> for Either<L, A> {
    fn flat_map<B, F>(self, f: F) -> Self::Output<B>
    where
        F: FnOnce(A) -> Self::Output<B>,
    {
        match self {
            Either::Left(left) => Either::Left(left),
            Either::Right(right) => f(right),
        }
    }
}

// 実用的な関数型操作
fn compose<A, B, C, F, G>(f: F, g: G) -> impl Fn(A) -> C
where
    F: Fn(A) -> B,
    G: Fn(B) -> C,
{
    move |x| g(f(x))
}

fn curry<A, B, C, F>(f: F) -> impl Fn(A) -> Box<dyn Fn(B) -> C>
where
    F: Fn(A, B) -> C + 'static,
{
    move |a| Box::new(move |b| f(a, b))
}

// 呪術システムでの応用例
#[derive(Debug, Clone)]
struct Sorcerer {
    name: String,
    power: i32,
    techniques: Vec<String>,
}

impl Sorcerer {
    fn new(name: &str, power: i32) -> Self {
        Sorcerer {
            name: name.to_string(),
            power,
            techniques: Vec::new(),
        }
    }

    fn add_technique(&mut self, technique: String) {
        self.techniques.push(technique);
    }

    fn power_up(&mut self, amount: i32) {
        self.power += amount;
    }
}

fn find_sorcerer(sorcerers: &[Sorcerer], name: &str) -> Maybe<&Sorcerer> {
    for sorcerer in sorcerers {
        if sorcerer.name == name {
            return Maybe::Just(sorcerer);
        }
    }
    Maybe::Nothing
}

fn validate_power(power: i32) -> Either<String, i32> {
    if power >= 0 {
        Either::Right(power)
    } else {
        Either::Left(\"呪力は負の値にできません\".to_string())
    }
}

fn main() {
    println!(\"=== 関数型プログラミングライブラリ ===\");

    // Maybe型の使用例
    println!(\"=== Maybe型の例 ===\");

    let just_value = Maybe::Just(1000);
    let nothing_value: Maybe<i32> = Maybe::Nothing;

    let doubled = just_value.map(|x| x * 2);
    let doubled_nothing = nothing_value.map(|x| x * 2);

    println!(\"Just(1000) * 2 = {:?}\", doubled);
    println!(\"Nothing * 2 = {:?}\", doubled_nothing);

    // モナドチェーンの例
    let result = Maybe::Just(15)
        .flat_map(|x| if x > 10 { Maybe::Just(x * 2) } else { Maybe::Nothing })
        .map(|x| x + 5);

    println!(\"モナドチェーン結果: {:?}\", result);

    // List型の使用例
    println!(\"\\n=== List型の例 ===\");

    let powers = List::from_vec(vec![1000, 1500, 800, 2000]);
    let doubled_powers = powers.map_fn(|&x| x * 2);

    println!(\"元の呪力: {:?}\", powers);
    println!(\"2倍の呪力: {:?}\", doubled_powers);

    // フィルタリング
    let high_powers = powers.filter(|&&x| x >= 1500);
    println!(\"高い呪力のみ: {:?}\", high_powers);

    // 畳み込み
    let total_power = powers.clone().fold(0, |acc, x| acc + x);
    println!(\"合計呪力: {}\", total_power);

    // Either型の使用例
    println!(\"\\n=== Either型の例 ===\");

    let valid_power = validate_power(1500);
    let invalid_power = validate_power(-100);

    println!(\"有効な呪力: {:?}\", valid_power);
    println!(\"無効な呪力: {:?}\", invalid_power);

    let processed_valid = valid_power.map(|x| x * 2);
    let processed_invalid = invalid_power.map(|x| x * 2);

    println!(\"処理後（有効）: {:?}\", processed_valid);
    println!(\"処理後（無効）: {:?}\", processed_invalid);

    // 呪術師システムでの応用
    println!(\"\\n=== 呪術師システムでの応用 ===\");

    let sorcerers = vec![
        Sorcerer::new(\"五条悟\", 3000),
        Sorcerer::new(\"虎杖悠仁\", 1200),
        Sorcerer::new(\"伏黒恵\", 1000),
    ];

    // 呪術師検索
    let found_sorcerer = find_sorcerer(&sorcerers, \"五条悟\");
    let not_found = find_sorcerer(&sorcerers, \"存在しない呪術師\");

    match found_sorcerer {
        Maybe::Just(sorcerer) => println!(\"見つかった: {} (呪力: {})\", sorcerer.name, sorcerer.power),
        Maybe::Nothing => println!(\"見つからない\"),
    }

    match not_found {
        Maybe::Just(sorcerer) => println!(\"見つかった: {} (呪力: {})\", sorcerer.name, sorcerer.power),
        Maybe::Nothing => println!(\"存在しない呪術師は見つからない（当然）\"),
    }

    // 関数合成の例
    println!(\"\\n=== 関数合成の例 ===\");

    let double = |x: i32| x * 2;
    let add_ten = |x: i32| x + 10;
    let double_then_add_ten = compose(double, add_ten);

    let result = double_then_add_ten(5);
    println!(\"5を2倍してから10足す: {}\", result);  // (5 * 2) + 10 = 20

    // カリー化の例
    let add = |x: i32, y: i32| x + y;
    let add_curried = curry(add);
    let add_five = add_curried(5);

    let result = add_five(3);
    println!(\"カリー化された加算 5 + 3: {}\", result);

    // リストの高度な操作
    println!(\"\\n=== リストの高度な操作 ===\");

    let sorcerer_powers = List::from_vec(vec![3000, 1200, 1000, 800]);

    // 呪力を等級に変換
    let grades = sorcerer_powers.map_fn(|&power| {
        match power {
            p if p >= 3000 => \"特級\",
            p if p >= 2000 => \"1級\",
            p if p >= 1000 => \"2級\",
            _ => \"3級以下\",
        }
    });

    println!(\"呪力から等級: {:?}\", grades);

    // 平均呪力の計算
    let total = sorcerer_powers.clone().fold(0, |acc, x| acc + x);
    let count = sorcerer_powers.len();
    let average = if count > 0 { total / count as i32 } else { 0 };

    println!(\"平均呪力: {}\", average);

    println!(\"\\n=== 関数型プログラミングライブラリテスト完了 ===\");
}
```

</div>
</details>

### 問題6: 総合問題 - 呪術学園管理システム

これまで学んだすべての技術を統合した大規模システムを実装せよ。

<div class=\"exercise\">

**要件:**

- ジェネリックなデータ管理
- 複数のトレイト実装
- 型安全な操作
- 関数型プログラミング要素
- エラーハンドリング
- 設定管理
- 戦闘システム
- 統計・レポート機能

自由に設計して実装してみよう！

</div>

<details>
<summary>解答例を見る</summary>
<div class=\"solution\">

```rust
// （コードが非常に長くなるため、主要部分のみ表示）

use std::collections::HashMap;
use std::fmt;
use std::marker::PhantomData;

// 基本的なトレイト定義
trait Entity {
    type Id: Clone + Eq + std::hash::Hash + fmt::Debug;

    fn id(&self) -> &Self::Id;
    fn name(&self) -> &str;
    fn entity_type(&self) -> &str;
}

trait Manageable: Entity {
    fn create_date(&self) -> &str;
    fn last_updated(&self) -> &str;
    fn update_timestamp(&mut self);
}

// 汎用的なリポジトリシステム
#[derive(Debug)]
struct Repository<T: Entity> {
    entities: HashMap<T::Id, T>,
    _phantom: PhantomData<T>,
}

impl<T: Entity> Repository<T> {
    fn new() -> Self {
        Repository {
            entities: HashMap::new(),
            _phantom: PhantomData,
        }
    }

    fn add(&mut self, entity: T) -> Result<(), String> {
        let id = entity.id().clone();
        if self.entities.contains_key(&id) {
            return Err(format!(\"ID {:?} は既に存在します\", id));
        }

        self.entities.insert(id, entity);
        Ok(())
    }

    fn get(&self, id: &T::Id) -> Option<&T> {
        self.entities.get(id)
    }

    fn get_mut(&mut self, id: &T::Id) -> Option<&mut T> {
        self.entities.get_mut(id)
    }

    fn remove(&mut self, id: &T::Id) -> Option<T> {
        self.entities.remove(id)
    }

    fn list_all(&self) -> Vec<&T> {
        self.entities.values().collect()
    }

    fn count(&self) -> usize {
        self.entities.len()
    }
}

// 学生エンティティ
#[derive(Debug, Clone)]
struct Student {
    id: String,
    name: String,
    grade: u8,
    power_level: i32,
    techniques: Vec<String>,
    enrollment_date: String,
    last_updated: String,
}

impl Student {
    fn new(id: &str, name: &str, grade: u8, power_level: i32) -> Self {
        let now = chrono::Utc::now().to_rfc3339();
        Student {
            id: id.to_string(),
            name: name.to_string(),
            grade,
            power_level,
            techniques: Vec::new(),
            enrollment_date: now.clone(),
            last_updated: now,
        }
    }

    fn add_technique(&mut self, technique: String) {
        self.techniques.push(technique);
        self.update_timestamp();
    }

    fn power_up(&mut self, amount: i32) {
        self.power_level += amount;
        self.update_timestamp();
    }
}

impl Entity for Student {
    type Id = String;

    fn id(&self) -> &Self::Id {
        &self.id
    }

    fn name(&self) -> &str {
        &self.name
    }

    fn entity_type(&self) -> &str {
        \"学生\"
    }
}

impl Manageable for Student {
    fn create_date(&self) -> &str {
        &self.enrollment_date
    }

    fn last_updated(&self) -> &str {
        &self.last_updated
    }

    fn update_timestamp(&mut self) {
        self.last_updated = chrono::Utc::now().to_rfc3339();
    }
}

// 学園管理システム
struct JujutsuAcademy {
    students: Repository<Student>,
    settings: HashMap<String, Box<dyn std::any::Any>>,
}

impl JujutsuAcademy {
    fn new() -> Self {
        JujutsuAcademy {
            students: Repository::new(),
            settings: HashMap::new(),
        }
    }

    fn enroll_student(&mut self, student: Student) -> Result<(), String> {
        println!(\"{}を学園に入学させます\", student.name());
        self.students.add(student)
    }

    fn get_student(&self, id: &str) -> Option<&Student> {
        self.students.get(id)
    }

    fn get_student_mut(&mut self, id: &str) -> Option<&mut Student> {
        self.students.get_mut(id)
    }

    fn generate_report(&self) -> String {
        let mut report = String::from(\"=== 東京呪術高等専門学校 統計レポート ===\\n\\n\");

        let total_students = self.students.count();
        report.push_str(&format!(\"総学生数: {}\\n\", total_students));

        if total_students > 0 {
            let all_students = self.students.list_all();

            // 学年別統計
            let mut grade_stats = HashMap::new();
            for student in &all_students {
                *grade_stats.entry(student.grade).or_insert(0) += 1;
            }

            report.push_str(\"\\n学年別分布:\\n\");
            for (grade, count) in grade_stats {
                report.push_str(&format!(\"  {}年生: {}人\\n\", grade, count));
            }

            // 呪力統計
            let total_power: i32 = all_students.iter().map(|s| s.power_level).sum();
            let avg_power = total_power / all_students.len() as i32;
            let max_power = all_students.iter().map(|s| s.power_level).max().unwrap_or(0);
            let min_power = all_students.iter().map(|s| s.power_level).min().unwrap_or(0);

            report.push_str(&format!(\"\\n呪力統計:\\n\"));
            report.push_str(&format!(\"  平均呪力: {}\\n\", avg_power));
            report.push_str(&format!(\"  最大呪力: {}\\n\", max_power));
            report.push_str(&format!(\"  最小呪力: {}\\n\", min_power));

            // 最強の学生
            if let Some(strongest) = all_students.iter().max_by_key(|s| s.power_level) {
                report.push_str(&format!(\"\\n最強の学生: {} (呪力: {})\\n\",
                               strongest.name(), strongest.power_level));
            }

            // 技術統計
            let total_techniques: usize = all_students.iter()
                .map(|s| s.techniques.len())
                .sum();

            report.push_str(&format!(\"\\n習得済み術式総数: {}\\n\", total_techniques));

            if total_students > 0 {
                let avg_techniques = total_techniques as f64 / total_students as f64;
                report.push_str(&format!(\"学生あたり平均術式数: {:.1}\\n\", avg_techniques));
            }
        }

        report
    }
}

fn main() {
    println!(\"=== 呪術学園総合管理システム ===\");

    let mut academy = JujutsuAcademy::new();

    // 学生の登録
    let students_data = [
        (\"S001\", \"五条悟\", 3, 3000),
        (\"S002\", \"虎杖悠仁\", 1, 1200),
        (\"S003\", \"伏黒恵\", 1, 1000),
        (\"S004\", \"釘崎野薔薇\", 1, 900),
        (\"S005\", \"禪院真希\", 2, 800),
        (\"S006\", \"狗巻棘\", 2, 700),
        (\"S007\", \"パンダ\", 2, 600),
    ];

    for (id, name, grade, power) in students_data {
        let mut student = Student::new(id, name, grade, power);

        // 術式の追加
        match name {
            \"五条悟\" => {
                student.add_technique(\"無下限呪術\".to_string());
                student.add_technique(\"領域展開・無量空処\".to_string());
            },
            \"虎杖悠仁\" => {
                student.add_technique(\"黒閃\".to_string());
                student.add_technique(\"発散\".to_string());
            },
            \"伏黒恵\" => {
                student.add_technique(\"十種影法術\".to_string());
            },
            \"釘崎野薔薇\" => {
                student.add_technique(\"芻霊呪法\".to_string());
            },
            _ => {},
        }

        match academy.enroll_student(student) {
            Ok(_) => println!(\"✓ {}の入学完了\", name),
            Err(e) => println!(\"✗ {}の入学失敗: {}\", name, e),
        }
    }

    println!();

    // 学生情報の表示
    println!(\"=== 登録学生一覧 ===\");
    for student in academy.students.list_all() {
        println!(\"{}: {} ({}年生, 呪力: {}, 術式数: {})\",
                 student.id(), student.name(), student.grade,
                 student.power_level, student.techniques.len());
    }

    println!();

    // 特定学生の詳細情報
    if let Some(gojo) = academy.get_student(\"S001\") {
        println!(\"=== {}の詳細情報 ===\", gojo.name());
        println!(\"学生ID: {}\", gojo.id());
        println!(\"学年: {}年生\", gojo.grade);
        println!(\"呪力: {}\", gojo.power_level);
        println!(\"習得術式: {:?}\", gojo.techniques);
        println!(\"入学日: {}\", gojo.enrollment_date);
        println!(\"最終更新: {}\", gojo.last_updated);
    }

    println!();

    // 学生の成長シミュレーション
    println!(\"=== 成長シミュレーション ===\");

    if let Some(yuji) = academy.get_student_mut(\"S002\") {
        println!(\"{}の訓練前: 呪力 {}\", yuji.name(), yuji.power_level);
        yuji.power_up(300);
        yuji.add_technique(\"簡易領域\".to_string());
        println!(\"{}の訓練後: 呪力 {}, 新術式: 簡易領域\", yuji.name(), yuji.power_level);
    }

    println!();

    // 統計レポートの生成
    println!(\"{}\", academy.generate_report());

    // 呪力レベル別グループ化
    println!(\"=== 呪力レベル別分類 ===\");

    let all_students = academy.students.list_all();
    let mut power_groups: HashMap<&str, Vec<&Student>> = HashMap::new();

    for student in all_students {
        let group = match student.power_level {
            3000.. => \"最強級\",
            2000..=2999 => \"特級\",
            1500..=1999 => \"1級\",
            1000..=1499 => \"2級\",
            500..=999 => \"3級\",
            _ => \"4級\",
        };

        power_groups.entry(group).or_insert_with(Vec::new).push(student);
    }

    for (group, students) in power_groups {
        println!(\"{}:\", group);
        for student in students {
            println!(\"  {} (呪力: {})\", student.name(), student.power_level);
        }
    }

    println!(\"\\n=== 呪術学園管理システム完了 ===\");
}

// 注意: chrono クレートを使用する場合は Cargo.toml に以下を追加:
// [dependencies]
// chrono = { version = \"0.4\", features = [\"serde\"] }
```

</div>
</details>

## 章末総括

第4章の練習問題、素晴らしい領域展開だった！ジェネリクスとトレイトの全ての技術を実践で確認できたな。

これらの問題を通して学んだこと：

- **ジェネリクス** - 型の抽象化による再利用性
- **トレイト** - 振る舞いの抽象化による拡張性
- **高度なトレイト** - 型システムの限界への挑戦
- **関数型プログラミング** - 抽象化の極致
- **型安全性** - コンパイル時の安全性保証

!!! tip "五条先生の最終メッセージ"
    ジェネリクスとトレイトは抽象化の究極形だ。一度マスターすれば、どんな複雑なシステムも美しく設計できる。

    俺の領域展開「無量空処」のように、無限の可能性を包含する力を手に入れた。この力を使って、型安全で再利用可能な美しいコードを書け。

次は第5章「無下限呪術編」で非同期プログラミングとマクロを学ぼう。最後の秘奥義の習得だ。

______________________________________________________________________

*「ジェネリクスとトレイトを極めれば、型の概念すら自在に操れる」*
