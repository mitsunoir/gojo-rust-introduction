# トレイト - 振る舞いの抽象化

## トレイトとは - 術式の型

ジェネリクスで型を抽象化したが、今度は**振る舞い**を抽象化する番だ。トレイト（Trait）は俺の無下限術式のように、「何ができるか」を定義する型だ。

他の言語のインターフェースに似ているが、Rustのトレイトはもっと強力だ。既存の型に後から機能を追加できるし、型安全性も保証される。まさに呪術の型を定義する技術だ。

!!! note "五条先生の解説"
    トレイトは「共通の振る舞い」を定義する。例えば`Display`トレイトは「表示できる」、`Clone`トレイトは「複製できる」という能力を表す。
    型がトレイトを実装することで、その能力を獲得する。

## 基本的なトレイト定義

### シンプルなトレイト

```rust
// 戦闘能力を表すトレイト
trait Combatant {
    fn attack(&self) -> i32;
    fn defend(&self) -> i32;
    fn health(&self) -> i32;

    // デフォルト実装を持つメソッド
    fn is_alive(&self) -> bool {
        self.health() > 0
    }

    fn battle_power(&self) -> i32 {
        self.attack() + self.defend()
    }
}

// 呪術師構造体
struct Sorcerer {
    name: String,
    attack_power: i32,
    defense_power: i32,
    current_health: i32,
}

impl Sorcerer {
    fn new(name: &str, attack: i32, defense: i32, health: i32) -> Self {
        Sorcerer {
            name: name.to_string(),
            attack_power: attack,
            defense_power: defense,
            current_health: health,
        }
    }
}

// トレイトの実装
impl Combatant for Sorcerer {
    fn attack(&self) -> i32 {
        self.attack_power
    }

    fn defend(&self) -> i32 {
        self.defense_power
    }

    fn health(&self) -> i32 {
        self.current_health
    }

    // デフォルト実装をオーバーライドすることも可能
    fn battle_power(&self) -> i32 {
        // 呪術師は名前にボーナスがある
        let name_bonus = if self.name.contains("五条") { 1000 } else { 0 };
        self.attack() + self.defend() + name_bonus
    }
}

fn main() {
    let gojo = Sorcerer::new("五条悟", 1500, 1000, 2000);
    let yuji = Sorcerer::new("虎杖悠仁", 800, 600, 1200);

    println!("{}: 攻撃{}, 防御{}, 戦闘力{}",
             gojo.name, gojo.attack(), gojo.defend(), gojo.battle_power());

    println!("{}: 攻撃{}, 防御{}, 戦闘力{}",
             yuji.name, yuji.attack(), yuji.defend(), yuji.battle_power());

    println!("{}は生きているか: {}", gojo.name, gojo.is_alive());
}
```

### 複数の型での実装

```rust
// 呪霊構造体
struct Curse {
    grade: String,
    attack_power: i32,
    defense_power: i32,
    current_health: i32,
}

impl Curse {
    fn new(grade: &str, attack: i32, defense: i32, health: i32) -> Self {
        Curse {
            grade: grade.to_string(),
            attack_power: attack,
            defense_power: defense,
            current_health: health,
        }
    }
}

// 呪霊も戦闘可能
impl Combatant for Curse {
    fn attack(&self) -> i32 {
        self.attack_power
    }

    fn defend(&self) -> i32 {
        self.defense_power
    }

    fn health(&self) -> i32 {
        self.current_health
    }

    fn battle_power(&self) -> i32 {
        // 呪霊は等級によってボーナス
        let grade_bonus = match self.grade.as_str() {
            "特級" => 500,
            "1級" => 200,
            "2級" => 100,
            _ => 0,
        };
        self.attack() + self.defend() + grade_bonus
    }
}

// トレイトオブジェクトとして扱う関数
fn simulate_battle(fighter1: &dyn Combatant, fighter2: &dyn Combatant) {
    println!("=== 戦闘シミュレーション ===");
    println!("戦闘者1: 攻撃{}, 防御{}, HP{}, 戦闘力{}",
             fighter1.attack(), fighter1.defend(), fighter1.health(), fighter1.battle_power());
    println!("戦闘者2: 攻撃{}, 防御{}, HP{}, 戦闘力{}",
             fighter2.attack(), fighter2.defend(), fighter2.health(), fighter2.battle_power());

    if fighter1.battle_power() > fighter2.battle_power() {
        println!("戦闘者1の勝利！");
    } else if fighter2.battle_power() > fighter1.battle_power() {
        println!("戦闘者2の勝利！");
    } else {
        println!("引き分け！");
    }
}

fn main() {
    let gojo = Sorcerer::new("五条悟", 1500, 1000, 2000);
    let special_curse = Curse::new("特級", 1200, 800, 1800);

    // 異なる型同士の戦闘
    simulate_battle(&gojo, &special_curse);
}
```

## 標準ライブラリのトレイト

### Display と Debug

```rust
use std::fmt;

#[derive(Debug)]  // Debugトレイトを自動実装
struct Technique {
    name: String,
    power: i32,
    element: String,
}

impl Technique {
    fn new(name: &str, power: i32, element: &str) -> Self {
        Technique {
            name: name.to_string(),
            power,
            element: element.to_string(),
        }
    }
}

// Displayトレイトの手動実装
impl fmt::Display for Technique {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}（威力: {}, 属性: {}）", self.name, self.power, self.element)
    }
}

fn main() {
    let blue = Technique::new("術式順転『青』", 1000, "無下限");

    println!("Display: {}", blue);     // Displayトレイト使用
    println!("Debug: {:?}", blue);     // Debugトレイト使用
    println!("Pretty Debug: {:#?}", blue);  // 整形されたDebug
}
```

### Clone と Copy

```rust
#[derive(Debug, Clone)]
struct SorcererStats {
    power: i32,
    level: i32,
}

#[derive(Debug, Clone, Copy)]  // Copyも実装
struct Position {
    x: f64,
    y: f64,
}

fn main() {
    let stats = SorcererStats { power: 1500, level: 10 };
    let cloned_stats = stats.clone();  // 明示的にクローン

    println!("元: {:?}", stats);
    println!("クローン: {:?}", cloned_stats);

    let pos1 = Position { x: 1.0, y: 2.0 };
    let pos2 = pos1;  // Copyトレイトにより自動コピー

    println!("位置1: {:?}", pos1);  // まだ使える
    println!("位置2: {:?}", pos2);
}
```

### PartialEq と Eq

```rust
#[derive(Debug, PartialEq, Eq)]
struct Grade {
    level: u8,
    name: String,
}

impl Grade {
    fn new(level: u8, name: &str) -> Self {
        Grade {
            level,
            name: name.to_string(),
        }
    }
}

fn main() {
    let special = Grade::new(0, "特級");
    let special2 = Grade::new(0, "特級");
    let first = Grade::new(1, "1級");

    println!("特級 == 特級: {}", special == special2);  // true
    println!("特級 == 1級: {}", special == first);     // false
}
```

### PartialOrd と Ord

```rust
#[derive(Debug, PartialEq, Eq, PartialOrd, Ord)]
struct PowerLevel(i32);

impl PowerLevel {
    fn new(power: i32) -> Self {
        PowerLevel(power)
    }
}

fn main() {
    let weak = PowerLevel::new(500);
    let strong = PowerLevel::new(1500);
    let strongest = PowerLevel::new(3000);

    println!("{:?} < {:?}: {}", weak, strong, weak < strong);

    let mut powers = vec![strongest, weak, strong];
    powers.sort();  // Ordトレイトにより自動ソート

    println!("ソート後: {:?}", powers);
}
```

## ジェネリックトレイト

### 関連型を持つトレイト

```rust
trait Technique {
    type Output;  // 関連型
    type Error;

    fn cast(&self, power: i32) -> Result<Self::Output, Self::Error>;
    fn required_power(&self) -> i32;
}

struct FireTechnique {
    name: String,
    base_power: i32,
}

impl FireTechnique {
    fn new(name: &str, base_power: i32) -> Self {
        FireTechnique {
            name: name.to_string(),
            base_power,
        }
    }
}

impl Technique for FireTechnique {
    type Output = String;  // 成功時は説明文字列
    type Error = String;   // エラー時はエラーメッセージ

    fn cast(&self, power: i32) -> Result<Self::Output, Self::Error> {
        if power >= self.required_power() {
            Ok(format!("{}を発動！ダメージ: {}", self.name, power))
        } else {
            Err(format!("呪力不足。必要: {}, 現在: {}", self.required_power(), power))
        }
    }

    fn required_power(&self) -> i32 {
        self.base_power
    }
}

fn execute_technique<T: Technique>(technique: &T, available_power: i32)
where
    T::Output: std::fmt::Display,
    T::Error: std::fmt::Display,
{
    match technique.cast(available_power) {
        Ok(result) => println!("✓ {}", result),
        Err(error) => println!("✗ {}", error),
    }
}

fn main() {
    let fire_blast = FireTechnique::new("火炎術", 800);

    execute_technique(&fire_blast, 1000);  // 成功
    execute_technique(&fire_blast, 500);   // 失敗
}
```

### 型パラメータを持つトレイト

```rust
trait Converter<T, U> {
    fn convert(&self, input: T) -> U;
}

struct PowerConverter;

impl Converter<i32, String> for PowerConverter {
    fn convert(&self, power: i32) -> String {
        match power {
            0..=500 => "弱い".to_string(),
            501..=1500 => "普通".to_string(),
            1501..=3000 => "強い".to_string(),
            _ => "最強".to_string(),
        }
    }
}

impl Converter<String, i32> for PowerConverter {
    fn convert(&self, grade: String) -> i32 {
        match grade.as_str() {
            "4級" => 200,
            "3級" => 500,
            "2級" => 800,
            "1級" => 1200,
            "特級" => 2500,
            _ => 0,
        }
    }
}

fn main() {
    let converter = PowerConverter;

    // i32 -> String
    let description = converter.convert(1800);
    println!("呪力1800: {}", description);

    // String -> i32
    let power = converter.convert("特級".to_string());
    println!("特級の呪力: {}", power);
}
```

## 高度なトレイト使用法

### トレイト境界とwhere句

```rust
use std::fmt::Display;

// 複雑なトレイト境界
fn analyze_and_display<T>(item: T)
where
    T: Display + Clone + PartialOrd,
{
    let cloned = item.clone();
    println!("アイテム: {}", item);
    println!("クローン: {}", cloned);

    if item > cloned {
        println!("元 > クローン");
    } else {
        println!("元 <= クローン");
    }
}

// 複数のトレイト境界
fn compare_and_process<T, U>(item1: T, item2: U) -> String
where
    T: Display + PartialOrd<U>,
    U: Display,
{
    if item1 > item2 {
        format!("{} は {} より大きい", item1, item2)
    } else {
        format!("{} は {} 以下", item1, item2)
    }
}

fn main() {
    analyze_and_display(42);
    analyze_and_display("五条悟");

    let result = compare_and_process(1500, 1000);
    println!("{}", result);
}
```

### トレイトオブジェクト

```rust
trait Ability {
    fn name(&self) -> &str;
    fn activate(&self, power: i32) -> String;
}

struct Limitless {
    technique_name: String,
}

impl Limitless {
    fn new(name: &str) -> Self {
        Limitless {
            technique_name: name.to_string(),
        }
    }
}

impl Ability for Limitless {
    fn name(&self) -> &str {
        &self.technique_name
    }

    fn activate(&self, power: i32) -> String {
        format!("無下限術式『{}』発動！威力: {}", self.technique_name, power * 2)
    }
}

struct BlackFlash;

impl Ability for BlackFlash {
    fn name(&self) -> &str {
        "黒閃"
    }

    fn activate(&self, power: i32) -> String {
        format!("黒閃発動！威力: {}", power * power / 100)
    }
}

// トレイトオブジェクトのベクター
fn main() {
    let abilities: Vec<Box<dyn Ability>> = vec![
        Box::new(Limitless::new("青")),
        Box::new(Limitless::new("赤")),
        Box::new(BlackFlash),
    ];

    for ability in abilities {
        println!("能力: {}", ability.name());
        println!("結果: {}", ability.activate(1000));
        println!();
    }
}
```

## 実践例 - 総合的な呪術システム

```rust
use std::fmt;

// 基本的な能力トレイト
trait Entity {
    fn name(&self) -> &str;
    fn power_level(&self) -> i32;
    fn entity_type(&self) -> &str;
}

// 戦闘能力トレイト
trait Combat {
    fn attack_power(&self) -> i32;
    fn defense_power(&self) -> i32;

    fn battle_effectiveness(&self) -> f64 {
        (self.attack_power() + self.defense_power()) as f64 / 2.0
    }
}

// 術式使用能力トレイト
trait TechniqueUser {
    fn available_techniques(&self) -> &[String];
    fn cast_technique(&self, technique: &str) -> Result<String, String>;
    fn learn_technique(&mut self, technique: String) -> Result<(), String>;
}

// 成長能力トレイト
trait Growable {
    fn experience(&self) -> i32;
    fn add_experience(&mut self, exp: i32);
    fn level_up(&mut self) -> Option<String>;
}

// 表示トレイト
trait DetailedDisplay {
    fn display_stats(&self) -> String;
    fn display_summary(&self) -> String;
}

// 呪術師実装
#[derive(Debug, Clone)]
struct Sorcerer {
    name: String,
    base_power: i32,
    techniques: Vec<String>,
    experience: i32,
    level: i32,
}

impl Sorcerer {
    fn new(name: &str, base_power: i32) -> Self {
        Sorcerer {
            name: name.to_string(),
            base_power,
            techniques: Vec::new(),
            experience: 0,
            level: 1,
        }
    }
}

impl Entity for Sorcerer {
    fn name(&self) -> &str {
        &self.name
    }

    fn power_level(&self) -> i32 {
        self.base_power + (self.level * 100)
    }

    fn entity_type(&self) -> &str {
        "呪術師"
    }
}

impl Combat for Sorcerer {
    fn attack_power(&self) -> i32 {
        self.power_level() + (self.techniques.len() as i32 * 200)
    }

    fn defense_power(&self) -> i32 {
        self.power_level() / 2 + (self.level * 50)
    }
}

impl TechniqueUser for Sorcerer {
    fn available_techniques(&self) -> &[String] {
        &self.techniques
    }

    fn cast_technique(&self, technique: &str) -> Result<String, String> {
        if self.techniques.contains(&technique.to_string()) {
            let power = self.attack_power();
            Ok(format!("{}が{}を発動！威力: {}", self.name, technique, power))
        } else {
            Err(format!("{}は{}を習得していません", self.name, technique))
        }
    }

    fn learn_technique(&mut self, technique: String) -> Result<(), String> {
        if self.techniques.contains(&technique) {
            Err(format!("{}は既に習得済みです", technique))
        } else if self.techniques.len() >= (self.level as usize * 2) {
            Err("これ以上の術式は習得できません".to_string())
        } else {
            self.techniques.push(technique.clone());
            Ok(())
        }
    }
}

impl Growable for Sorcerer {
    fn experience(&self) -> i32 {
        self.experience
    }

    fn add_experience(&mut self, exp: i32) {
        self.experience += exp;
    }

    fn level_up(&mut self) -> Option<String> {
        let required_exp = self.level * 1000;
        if self.experience >= required_exp {
            self.level += 1;
            self.experience -= required_exp;
            Some(format!("{}がレベル{}に上がりました！", self.name, self.level))
        } else {
            None
        }
    }
}

impl DetailedDisplay for Sorcerer {
    fn display_stats(&self) -> String {
        format!(
            "=== {} ===\n種別: {}\n基本呪力: {}\n総合呪力: {}\n攻撃力: {}\n防御力: {}\nレベル: {}\n経験値: {}\n習得術式: {:?}\n戦闘効果: {:.1}",
            self.name(),
            self.entity_type(),
            self.base_power,
            self.power_level(),
            self.attack_power(),
            self.defense_power(),
            self.level,
            self.experience(),
            self.available_techniques(),
            self.battle_effectiveness()
        )
    }

    fn display_summary(&self) -> String {
        format!("{} (Lv.{}, 呪力: {}, 術式数: {})",
                 self.name(), self.level, self.power_level(), self.techniques.len())
    }
}

impl fmt::Display for Sorcerer {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.display_summary())
    }
}

// 汎用的な戦闘システム
fn conduct_battle<T1, T2>(fighter1: &T1, fighter2: &T2) -> String
where
    T1: Entity + Combat + DetailedDisplay,
    T2: Entity + Combat + DetailedDisplay,
{
    let mut result = String::new();

    result.push_str(&format!("=== 戦闘: {} vs {} ===\n",
                            fighter1.name(), fighter2.name()));

    result.push_str(&format!("{}の戦闘効果: {:.1}\n",
                            fighter1.name(), fighter1.battle_effectiveness()));
    result.push_str(&format!("{}の戦闘効果: {:.1}\n",
                            fighter2.name(), fighter2.battle_effectiveness()));

    if fighter1.battle_effectiveness() > fighter2.battle_effectiveness() {
        result.push_str(&format!("勝者: {}\n", fighter1.name()));
    } else if fighter2.battle_effectiveness() > fighter1.battle_effectiveness() {
        result.push_str(&format!("勝者: {}\n", fighter2.name()));
    } else {
        result.push_str("引き分け\n");
    }

    result
}

// 訓練システム
fn training_session<T>(trainee: &mut T, technique_name: &str, exp_gain: i32) -> Vec<String>
where
    T: TechniqueUser + Growable + Entity,
{
    let mut results = Vec::new();

    // 術式習得の試行
    match trainee.learn_technique(technique_name.to_string()) {
        Ok(_) => results.push(format!("{}が{}を習得しました！", trainee.name(), technique_name)),
        Err(e) => results.push(format!("習得失敗: {}", e)),
    }

    // 経験値獲得
    trainee.add_experience(exp_gain);
    results.push(format!("{}が{}の経験値を獲得", trainee.name(), exp_gain));

    // レベルアップ判定
    if let Some(levelup_msg) = trainee.level_up() {
        results.push(levelup_msg);
    }

    results
}

fn main() {
    println!("=== 総合呪術システム ===");

    // 呪術師の作成
    let mut gojo = Sorcerer::new("五条悟", 2500);
    let mut yuji = Sorcerer::new("虎杖悠仁", 1000);

    println!("{}", gojo.display_stats());
    println!();
    println!("{}", yuji.display_stats());
    println!();

    // 初期戦闘
    println!("{}", conduct_battle(&gojo, &yuji));

    // 訓練セッション
    println!("=== 虎杖の訓練 ===");
    let training_results = training_session(&mut yuji, "黒閃", 800);
    for result in training_results {
        println!("{}", result);
    }

    let training_results2 = training_session(&mut yuji, "発散", 600);
    for result in training_results2 {
        println!("{}", result);
    }

    println!("\n=== 訓練後の虎杖 ===");
    println!("{}", yuji.display_stats());

    // 再戦
    println!("\n=== 再戦 ===");
    println!("{}", conduct_battle(&gojo, &yuji));

    // 術式使用テスト
    println!("\n=== 術式使用テスト ===");
    match yuji.cast_technique("黒閃") {
        Ok(result) => println!("✓ {}", result),
        Err(error) => println!("✗ {}", error),
    }

    match yuji.cast_technique("存在しない術式") {
        Ok(result) => println!("✓ {}", result),
        Err(error) => println!("✗ {}", error),
    }
}
```

## まとめ

トレイトの力をマスターできたか？重要なポイント：

1. **振る舞いの抽象化** - 「何ができるか」を型で表現
1. **コードの再利用** - 同じトレイトを複数の型で実装
1. **型安全性** - コンパイル時に能力の有無を保証
1. **関連型** - トレイト固有の型定義
1. **トレイト境界** - ジェネリクスとの組み合わせ

これで型に振る舞いを与える力を手に入れた。まさに俺の領域展開のように、抽象的な概念を具体的な力に変換できるようになったな。

次は高度なトレイトテクニックを学ぼう。より複雑で強力な抽象化の世界だ。

______________________________________________________________________

*「トレイトを極めれば、型に魂を宿すことができる」*
