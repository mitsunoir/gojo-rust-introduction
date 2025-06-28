# 高度なトレイト - 領域展開の奥義

## 高度なトレイト技術 - 無量空処の完成形

基本的なトレイトを習得したが、ここからが本当の力だ。高度なトレイト技術は俺の領域展開「無量空処」の完成形のように、**概念そのものを操作する**究極の抽象化技術だ。

複数のトレイトの組み合わせ、動的ディスパッチ、トレイトオブジェクト、そして高階トレイト境界。これらを習得すれば、Rustの真の力を引き出せる。

!!! note "五条先生の解説"
    高度なトレイト技術では、型システムの限界を押し広げ、コンパイル時とランタイムの両方で柔軟性を獲得する。
    まさに無限の可能性を秘めた領域だ。

## 高階トレイト境界（HRTB）

### ライフタイムを持つクロージャ

```rust
// 高階トレイト境界の例
fn apply_to_all<F>(texts: &[&str], func: F) -> Vec<String>
where
    F: for<'a> Fn(&'a str) -> String,  // あらゆるライフタイムで動作
{
    texts.iter().map(|&text| func(text)).collect()
}

fn process_sorcerer_names() {
    let names = ["五条悟", "虎杖悠仁", "伏黒恵"];

    // すべての名前に敬称を付ける
    let processed = apply_to_all(&names, |name| {
        format!("{}先輩", name)
    });

    println!("処理後: {:?}", processed);

    // 名前の長さを取得
    let lengths = apply_to_all(&names, |name| {
        format!("{}文字", name.len())
    });

    println!("長さ: {:?}", lengths);
}

fn main() {
    process_sorcerer_names();
}
```

### 複雑なクロージャ制約

```rust
use std::fmt::Display;

// 複数のライフタイムパラメータを持つ高階境界
fn complex_operation<F, T>(data: &[T], processor: F) -> Vec<String>
where
    F: for<'a, 'b> Fn(&'a T, &'b str) -> String,
    T: Display,
{
    let suffix = "の結果";
    data.iter()
        .map(|item| processor(item, suffix))
        .collect()
}

fn main() {
    let powers = [1000, 1500, 3000];

    let results = complex_operation(&powers, |power, suffix| {
        format!("呪力{}{}", power, suffix)
    });

    println!("結果: {:?}", results);
}
```

## 動的ディスパッチとトレイトオブジェクト

### 基本的なトレイトオブジェクト

```rust
trait Technique {
    fn name(&self) -> &str;
    fn cast(&self, power: i32) -> String;
    fn required_power(&self) -> i32;
}

struct Limitless {
    variant: String,
    base_power: i32,
}

impl Limitless {
    fn new(variant: &str, base_power: i32) -> Self {
        Limitless {
            variant: variant.to_string(),
            base_power,
        }
    }
}

impl Technique for Limitless {
    fn name(&self) -> &str {
        &self.variant
    }

    fn cast(&self, power: i32) -> String {
        if power >= self.required_power() {
            format!("無下限術式『{}』発動！空間を操作した。", self.variant)
        } else {
            format!("呪力不足で{}の発動に失敗", self.variant)
        }
    }

    fn required_power(&self) -> i32 {
        self.base_power
    }
}

struct BlackFlash {
    combo_count: u32,
}

impl BlackFlash {
    fn new() -> Self {
        BlackFlash { combo_count: 0 }
    }
}

impl Technique for BlackFlash {
    fn name(&self) -> &str {
        "黒閃"
    }

    fn cast(&self, power: i32) -> String {
        if power >= self.required_power() {
            let multiplier = 1.0 + (self.combo_count as f64 * 0.2);
            let damage = (power as f64 * multiplier) as i32;
            format!("黒閃発動！{} コンボで {} ダメージ！", self.combo_count + 1, damage)
        } else {
            "黒閃のタイミングが合わない".to_string()
        }
    }

    fn required_power(&self) -> i32 {
        800
    }
}

// トレイトオブジェクトを使った戦闘システム
struct CombatSystem {
    techniques: Vec<Box<dyn Technique>>,
}

impl CombatSystem {
    fn new() -> Self {
        CombatSystem {
            techniques: Vec::new(),
        }
    }

    fn add_technique(&mut self, technique: Box<dyn Technique>) {
        self.techniques.push(technique);
    }

    fn execute_combo(&self, available_power: i32) -> Vec<String> {
        let mut results = Vec::new();
        let mut remaining_power = available_power;

        println!("=== コンボ攻撃開始（使用可能呪力: {}）===", available_power);

        for technique in &self.techniques {
            if remaining_power >= technique.required_power() {
                let result = technique.cast(remaining_power);
                remaining_power -= technique.required_power();
                results.push(format!("✓ {}: {}", technique.name(), result));
            } else {
                results.push(format!("✗ {}: 呪力不足（必要: {}, 残り: {}）",
                           technique.name(), technique.required_power(), remaining_power));
            }
        }

        results.push(format!("残り呪力: {}", remaining_power));
        results
    }

    fn list_techniques(&self) -> Vec<String> {
        self.techniques.iter()
            .map(|t| format!("{} (必要呪力: {})", t.name(), t.required_power()))
            .collect()
    }
}

fn main() {
    let mut combat = CombatSystem::new();

    // 様々な術式を追加
    combat.add_technique(Box::new(Limitless::new("蒼", 500)));
    combat.add_technique(Box::new(BlackFlash::new()));
    combat.add_technique(Box::new(Limitless::new("赤", 800)));
    combat.add_technique(Box::new(Limitless::new("茈", 1500)));

    println!("登録された術式:");
    for technique in combat.list_techniques() {
        println!("  {}", technique);
    }

    println!();

    // コンボ実行
    let results = combat.execute_combo(2500);
    for result in results {
        println!("{}", result);
    }
}
```

### より複雑なトレイトオブジェクト

```rust
use std::any::Any;

// オブジェクトセーフなトレイト
trait Entity: std::fmt::Debug {
    fn name(&self) -> &str;
    fn entity_type(&self) -> &str;
    fn power_level(&self) -> i32;

    // ダウンキャスト用
    fn as_any(&self) -> &dyn Any;
}

trait Combatant: Entity {
    fn attack(&self, target: &mut dyn Entity) -> String;
    fn take_damage(&mut self, damage: i32);
    fn is_alive(&self) -> bool;
}

#[derive(Debug)]
struct Sorcerer {
    name: String,
    power: i32,
    health: i32,
    max_health: i32,
}

impl Sorcerer {
    fn new(name: &str, power: i32, health: i32) -> Self {
        Sorcerer {
            name: name.to_string(),
            power,
            health,
            max_health: health,
        }
    }

    fn heal(&mut self, amount: i32) {
        self.health = (self.health + amount).min(self.max_health);
    }
}

impl Entity for Sorcerer {
    fn name(&self) -> &str {
        &self.name
    }

    fn entity_type(&self) -> &str {
        "呪術師"
    }

    fn power_level(&self) -> i32 {
        self.power
    }

    fn as_any(&self) -> &dyn Any {
        self
    }
}

impl Combatant for Sorcerer {
    fn attack(&self, target: &mut dyn Entity) -> String {
        let damage = self.power / 10;

        // ダウンキャストして特殊処理
        if let Some(target_sorcerer) = target.as_any().downcast_ref::<Sorcerer>() {
            if target_sorcerer.name.contains("五条") {
                return format!("{}の攻撃は{}によって無効化された", self.name, target.name());
            }
        }

        if let Some(target_combatant) = (target as &mut dyn Any).downcast_mut::<Sorcerer>() {
            target_combatant.take_damage(damage);
            format!("{}が{}を攻撃！{}ダメージ", self.name, target.name(), damage)
        } else {
            format!("{}の攻撃が失敗", self.name)
        }
    }

    fn take_damage(&mut self, damage: i32) {
        self.health = (self.health - damage).max(0);
    }

    fn is_alive(&self) -> bool {
        self.health > 0
    }
}

#[derive(Debug)]
struct Curse {
    grade: String,
    power: i32,
    health: i32,
}

impl Curse {
    fn new(grade: &str, power: i32, health: i32) -> Self {
        Curse {
            grade: grade.to_string(),
            power,
            health,
        }
    }
}

impl Entity for Curse {
    fn name(&self) -> &str {
        &self.grade
    }

    fn entity_type(&self) -> &str {
        "呪霊"
    }

    fn power_level(&self) -> i32 {
        self.power
    }

    fn as_any(&self) -> &dyn Any {
        self
    }
}

impl Combatant for Curse {
    fn attack(&self, target: &mut dyn Entity) -> String {
        let damage = self.power / 8;  // 呪霊は少し強い

        if let Some(target_combatant) = (target as &mut dyn Any).downcast_mut::<Sorcerer>() {
            target_combatant.take_damage(damage);
            format!("{}級呪霊が{}を攻撃！{}ダメージ", self.grade, target.name(), damage)
        } else {
            format!("{}級呪霊の攻撃が失敗", self.grade)
        }
    }

    fn take_damage(&mut self, damage: i32) {
        self.health = (self.health - damage).max(0);
    }

    fn is_alive(&self) -> bool {
        self.health > 0
    }
}

// 戦闘管理システム
struct BattleArena {
    combatants: Vec<Box<dyn Combatant>>,
}

impl BattleArena {
    fn new() -> Self {
        BattleArena {
            combatants: Vec::new(),
        }
    }

    fn add_combatant(&mut self, combatant: Box<dyn Combatant>) {
        println!("{}（{}）が戦場に参戦！", combatant.name(), combatant.entity_type());
        self.combatants.push(combatant);
    }

    fn battle_royale(&mut self) -> String {
        let mut round = 1;
        let mut log = String::new();

        log.push_str("=== バトルロワイヤル開始 ===\n");

        while self.alive_count() > 1 && round <= 10 {
            log.push_str(&format!("\n--- ラウンド {} ---\n", round));

            let alive_indices: Vec<usize> = self.combatants.iter()
                .enumerate()
                .filter(|(_, c)| c.is_alive())
                .map(|(i, _)| i)
                .collect();

            if alive_indices.len() < 2 {
                break;
            }

            // 最初の生存者が次の生存者を攻撃
            let attacker_idx = alive_indices[0];
            let target_idx = alive_indices[1];

            // 借用の問題を回避するため、一時的に取り出す
            let attack_result = {
                let attacker = &self.combatants[attacker_idx];
                let attacker_name = attacker.name().to_string();
                let target_name = self.combatants[target_idx].name().to_string();

                format!("{}が{}を攻撃しようとしている...", attacker_name, target_name)
            };

            log.push_str(&format!("{}\n", attack_result));

            // 実際の攻撃は簡易的にダメージ計算
            let damage = self.combatants[attacker_idx].power_level() / 10;

            if let Some(target) = self.combatants.get_mut(target_idx) {
                target.take_damage(damage);
                log.push_str(&format!("{}ダメージ！\n", damage));

                if !target.is_alive() {
                    log.push_str(&format!("{}が倒れた！\n", target.name()));
                }
            }

            round += 1;
        }

        // 勝者の発表
        let winners: Vec<&Box<dyn Combatant>> = self.combatants.iter()
            .filter(|c| c.is_alive())
            .collect();

        if winners.len() == 1 {
            log.push_str(&format!("\n勝者: {}！\n", winners[0].name()));
        } else if winners.len() > 1 {
            log.push_str("\n複数の生存者あり：\n");
            for winner in winners {
                log.push_str(&format!("  {}\n", winner.name()));
            }
        } else {
            log.push_str("\n全員倒れた...\n");
        }

        log
    }

    fn alive_count(&self) -> usize {
        self.combatants.iter().filter(|c| c.is_alive()).count()
    }

    fn status_report(&self) -> String {
        let mut report = String::from("=== 戦場状況 ===\n");

        for combatant in &self.combatants {
            let status = if combatant.is_alive() { "生存" } else { "戦闘不能" };
            report.push_str(&format!("{} ({}): {}\n",
                           combatant.name(), combatant.entity_type(), status));
        }

        report
    }
}

fn main() {
    let mut arena = BattleArena::new();

    // 参戦者追加
    arena.add_combatant(Box::new(Sorcerer::new("五条悟", 3000, 2000)));
    arena.add_combatant(Box::new(Sorcerer::new("虎杖悠仁", 1200, 1500)));
    arena.add_combatant(Box::new(Curse::new("特級", 2000, 1800)));
    arena.add_combatant(Box::new(Curse::new("1級", 800, 1000)));

    println!("{}", arena.status_report());

    // バトル開始
    let battle_log = arena.battle_royale();
    println!("{}", battle_log);

    println!("{}", arena.status_report());
}
```

## 関連型と型族

### 複雑な関連型の使用

```rust
trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // デフォルト実装
    fn collect<B: FromIterator<Self::Item>>(self) -> B
    where
        Self: Sized,
    {
        FromIterator::from_iter(self)
    }
}

trait FromIterator<T> {
    fn from_iter<I: Iterator<Item = T>>(iter: I) -> Self;
}

// カスタムイテレータの実装
struct PowerSequence {
    current: i32,
    max: i32,
    step: i32,
}

impl PowerSequence {
    fn new(start: i32, max: i32, step: i32) -> Self {
        PowerSequence {
            current: start,
            max,
            step,
        }
    }
}

impl Iterator for PowerSequence {
    type Item = i32;

    fn next(&mut self) -> Option<Self::Item> {
        if self.current <= self.max {
            let current = self.current;
            self.current += self.step;
            Some(current)
        } else {
            None
        }
    }
}

// 複雑な関連型の例
trait TechniqueSystem {
    type TechniqueId;
    type Result;
    type Error;

    fn register_technique(&mut self, id: Self::TechniqueId, name: String) -> Result<(), Self::Error>;
    fn execute_technique(&self, id: &Self::TechniqueId, power: i32) -> Result<Self::Result, Self::Error>;
    fn list_techniques(&self) -> Vec<(Self::TechniqueId, String)>
    where
        Self::TechniqueId: Clone;
}

#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct TechniqueId(u32);

#[derive(Debug)]
enum SystemError {
    TechniqueNotFound,
    InsufficientPower,
    SystemFailure(String),
}

struct SorcererTechniqueSystem {
    techniques: std::collections::HashMap<TechniqueId, (String, i32)>, // (名前, 必要呪力)
    next_id: u32,
}

impl SorcererTechniqueSystem {
    fn new() -> Self {
        SorcererTechniqueSystem {
            techniques: std::collections::HashMap::new(),
            next_id: 1,
        }
    }
}

impl TechniqueSystem for SorcererTechniqueSystem {
    type TechniqueId = TechniqueId;
    type Result = String;
    type Error = SystemError;

    fn register_technique(&mut self, id: Self::TechniqueId, name: String) -> Result<(), Self::Error> {
        let required_power = match name.as_str() {
            n if n.contains("蒼") => 500,
            n if n.contains("赤") => 800,
            n if n.contains("茈") => 1500,
            n if n.contains("紫") => 3000,
            _ => 300,
        };

        self.techniques.insert(id, (name, required_power));
        Ok(())
    }

    fn execute_technique(&self, id: &Self::TechniqueId, power: i32) -> Result<Self::Result, Self::Error> {
        match self.techniques.get(id) {
            Some((name, required_power)) => {
                if power >= *required_power {
                    Ok(format!("{}を発動！威力: {}", name, power))
                } else {
                    Err(SystemError::InsufficientPower)
                }
            },
            None => Err(SystemError::TechniqueNotFound),
        }
    }

    fn list_techniques(&self) -> Vec<(Self::TechniqueId, String)> {
        self.techniques.iter()
            .map(|(id, (name, _))| (id.clone(), name.clone()))
            .collect()
    }
}

fn main() {
    println!("=== 関連型システムデモ ===");

    // カスタムイテレータ
    let power_sequence = PowerSequence::new(100, 1000, 200);
    let powers: Vec<i32> = power_sequence.collect();
    println!("呪力シーケンス: {:?}", powers);

    // 術式システム
    let mut system = SorcererTechniqueSystem::new();

    let blue_id = TechniqueId(1);
    let red_id = TechniqueId(2);
    let hollow_id = TechniqueId(3);

    system.register_technique(blue_id.clone(), "術式順転『蒼』".to_string()).unwrap();
    system.register_technique(red_id.clone(), "術式反転『赤』".to_string()).unwrap();
    system.register_technique(hollow_id.clone(), "虚式『茈』".to_string()).unwrap();

    println!("\n登録された術式:");
    for (id, name) in system.list_techniques() {
        println!("  ID {:?}: {}", id, name);
    }

    println!("\n術式実行テスト:");
    match system.execute_technique(&blue_id, 600) {
        Ok(result) => println!("✓ {}", result),
        Err(e) => println!("✗ {:?}", e),
    }

    match system.execute_technique(&hollow_id, 1000) {
        Ok(result) => println!("✓ {}", result),
        Err(e) => println!("✗ {:?}", e),
    }
}
```

## マーカートレイト

### 型の性質を表すマーカー

```rust
// 安全性を表すマーカートレイト
trait Safe {}
trait Dangerous {}

// 威力レベルを表すマーカー
trait LowPower {}
trait HighPower {}

struct BasicTechnique {
    name: String,
    power: i32,
}

struct AdvancedTechnique {
    name: String,
    power: i32,
    side_effects: Vec<String>,
}

impl BasicTechnique {
    fn new(name: &str, power: i32) -> Self {
        BasicTechnique {
            name: name.to_string(),
            power,
        }
    }
}

impl AdvancedTechnique {
    fn new(name: &str, power: i32, side_effects: Vec<String>) -> Self {
        AdvancedTechnique {
            name: name.to_string(),
            power,
            side_effects,
        }
    }
}

// マーカートレイトの実装
impl Safe for BasicTechnique {}
impl LowPower for BasicTechnique {}

impl Dangerous for AdvancedTechnique {}
impl HighPower for AdvancedTechnique {}

// マーカートレイトを使った型制約
fn safe_training<T: Safe>(technique: &T) -> String {
    "安全な訓練を実施".to_string()
}

fn dangerous_training<T: Dangerous>(technique: &T) -> String {
    "⚠️ 危険な訓練 - 細心の注意が必要".to_string()
}

fn low_power_practice<T: LowPower>(technique: &T) -> String {
    "低威力練習 - 初心者向け".to_string()
}

fn high_power_practice<T: HighPower>(technique: &T) -> String {
    "高威力練習 - 上級者専用".to_string()
}

// 複数のマーカーを要求
fn specialized_training<T>(technique: &T) -> String
where
    T: Dangerous + HighPower,
{
    "特殊訓練 - 最高レベルの注意が必要".to_string()
}

fn main() {
    let basic = BasicTechnique::new("基本術式", 300);
    let advanced = AdvancedTechnique::new("禁断の術式", 2000, vec!["呪力消耗".to_string()]);

    println!("{}", safe_training(&basic));
    println!("{}", low_power_practice(&basic));

    println!("{}", dangerous_training(&advanced));
    println!("{}", high_power_practice(&advanced));
    println!("{}", specialized_training(&advanced));

    // コンパイルエラーになる例（コメントアウト）
    // println!("{}", dangerous_training(&basic));  // BasicはDangerousではない
    // println!("{}", safe_training(&advanced));    // AdvancedはSafeではない
}
```

## 型レベルプログラミング

### 幽霊型（Phantom Types）

```rust
use std::marker::PhantomData;

// 状態を型で表現
struct Sealed;
struct Unsealed;

struct Prepared;
struct Unprepared;

// 幽霊型を使った術式状態管理
struct Technique<State, Preparation> {
    name: String,
    power: i32,
    _state: PhantomData<State>,
    _preparation: PhantomData<Preparation>,
}

impl<State, Preparation> Technique<State, Preparation> {
    fn name(&self) -> &str {
        &self.name
    }

    fn power(&self) -> i32 {
        self.power
    }
}

impl Technique<Unsealed, Unprepared> {
    fn new(name: &str, power: i32) -> Self {
        Technique {
            name: name.to_string(),
            power,
            _state: PhantomData,
            _preparation: PhantomData,
        }
    }

    fn prepare(self) -> Technique<Unsealed, Prepared> {
        Technique {
            name: self.name,
            power: self.power,
            _state: PhantomData,
            _preparation: PhantomData,
        }
    }
}

impl Technique<Unsealed, Prepared> {
    fn seal(self) -> Technique<Sealed, Prepared> {
        Technique {
            name: self.name,
            power: self.power * 2,  // 封印により威力アップ
            _state: PhantomData,
            _preparation: PhantomData,
        }
    }

    fn cast(self) -> String {
        format!("{}を発動！威力: {}", self.name, self.power)
    }
}

impl Technique<Sealed, Prepared> {
    fn unseal(self) -> Technique<Unsealed, Prepared> {
        Technique {
            name: self.name,
            power: self.power / 2,  // 封印解除で元の威力に
            _state: PhantomData,
            _preparation: PhantomData,
        }
    }

    fn forbidden_cast(self) -> String {
        format!("封印された{}を強制発動！威力: {} (危険)", self.name, self.power)
    }
}

// 型安全な術式管理システム
struct TechniqueInventory {
    unprepared: Vec<Technique<Unsealed, Unprepared>>,
    prepared: Vec<Technique<Unsealed, Prepared>>,
    sealed: Vec<Technique<Sealed, Prepared>>,
}

impl TechniqueInventory {
    fn new() -> Self {
        TechniqueInventory {
            unprepared: Vec::new(),
            prepared: Vec::new(),
            sealed: Vec::new(),
        }
    }

    fn add_technique(&mut self, name: &str, power: i32) {
        let technique = Technique::new(name, power);
        self.unprepared.push(technique);
    }

    fn prepare_technique(&mut self, index: usize) -> Option<()> {
        if index < self.unprepared.len() {
            let technique = self.unprepared.remove(index);
            self.prepared.push(technique.prepare());
            Some(())
        } else {
            None
        }
    }

    fn seal_technique(&mut self, index: usize) -> Option<()> {
        if index < self.prepared.len() {
            let technique = self.prepared.remove(index);
            self.sealed.push(technique.seal());
            Some(())
        } else {
            None
        }
    }

    fn cast_prepared(&mut self, index: usize) -> Option<String> {
        if index < self.prepared.len() {
            let technique = self.prepared.remove(index);
            Some(technique.cast())
        } else {
            None
        }
    }

    fn cast_sealed(&mut self, index: usize) -> Option<String> {
        if index < self.sealed.len() {
            let technique = self.sealed.remove(index);
            Some(technique.forbidden_cast())
        } else {
            None
        }
    }

    fn status(&self) -> String {
        format!(
            "未準備: {}個, 準備済み: {}個, 封印済み: {}個",
            self.unprepared.len(),
            self.prepared.len(),
            self.sealed.len()
        )
    }
}

fn main() {
    println!("=== 型安全な術式管理システム ===");

    let mut inventory = TechniqueInventory::new();

    // 術式を追加
    inventory.add_technique("術式順転『蒼』", 1000);
    inventory.add_technique("術式反転『赤』", 1500);
    inventory.add_technique("虚式『茈』", 3000);

    println!("初期状態: {}", inventory.status());

    // 段階的に処理
    inventory.prepare_technique(0).unwrap();
    inventory.prepare_technique(0).unwrap();  // インデックスが変わることに注意
    inventory.prepare_technique(0).unwrap();

    println!("準備後: {}", inventory.status());

    // 一つを封印
    inventory.seal_technique(2).unwrap();  // 茈を封印

    println!("封印後: {}", inventory.status());

    // 術式発動
    if let Some(result) = inventory.cast_prepared(0) {
        println!("通常発動: {}", result);
    }

    if let Some(result) = inventory.cast_sealed(0) {
        println!("封印発動: {}", result);
    }

    println!("最終状態: {}", inventory.status());
}
```

## まとめ

高度なトレイト技術の習得は完了だ！重要なポイント：

1. **高階トレイト境界** - あらゆるライフタイムで動作する制約
1. **動的ディスパッチ** - ランタイムでの柔軟な型選択
1. **関連型** - トレイト固有の型定義
1. **マーカートレイト** - 型の性質を表現
1. **幽霊型** - 型レベルでの状態管理

これで領域展開の奥義を完全にマスターした。型システムを自在に操り、コンパイル時とランタイムの両方で絶対的な制御を行えるようになったな。

次は第4章の練習問題で、これまでの知識を実践的に確認しよう。

______________________________________________________________________

*「高度なトレイトを極めれば、型システムの神となれる」*
