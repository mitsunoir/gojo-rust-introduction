# 第4章：領域展開編 - ジェネリクス

## 領域展開の概念 - 抽象化の究極形

ついに来たな、領域展開編。これまでの基本術式、呪力操作、反転術式を習得した君なら、この最高位の技術を理解できるはずだ。

ジェネリクス（総称型）は俺の領域展開「無量空処」のように、**型という概念を抽象化し、あらゆる可能性を包含する**技術だ。一つのコードで複数の型を扱える、まさに無限の可能性を持つ力だ。

!!! note "五条先生の解説"
    ジェネリクスは「型をパラメータ化」する技術。同じロジックを異なる型で再利用できる。
    `Vec<i32>`、`Vec<String>`のように、一つのVecという概念で様々な型のコレクションを表現できる。

## ジェネリクスの基本

### 基本的なジェネリック関数

```rust
// 型パラメータT - あらゆる型を受け入れる
fn display_value<T>(value: T)
where
    T: std::fmt::Display,
{
    println!(\"値: {}\", value);
}

fn main() {
    display_value(42);              // i32
    display_value(\"五条悟\");       // &str
    display_value(3.14);            // f64
    display_value(true);            // bool
}
```

### 複数の型パラメータ

```rust
fn compare_and_display<T, U>(first: T, second: U)
where
    T: std::fmt::Display,
    U: std::fmt::Display,
{
    println!(\"比較: {} vs {}\", first, second);
}

fn main() {
    compare_and_display(\"五条悟\", 3000);
    compare_and_display(1500, \"虎杖悠仁\");
    compare_and_display(true, false);
}
```

### 戻り値のジェネリック

```rust
fn get_maximum<T>(a: T, b: T) -> T
where
    T: PartialOrd,
{
    if a > b { a } else { b }
}

fn main() {
    let max_power = get_maximum(2000, 3000);
    let max_name = get_maximum(\"虎杖\", \"五条\");  // 辞書順

    println!(\"最大呪力: {}\", max_power);
    println!(\"後の名前: {}\", max_name);
}
```

## ジェネリック構造体

### 基本的なジェネリック構造体

```rust
#[derive(Debug)]
struct Container<T> {
    value: T,
}

impl<T> Container<T> {
    fn new(value: T) -> Self {
        Container { value }
    }

    fn get(&self) -> &T {
        &self.value
    }

    fn set(&mut self, value: T) {
        self.value = value;
    }

    // 型変換メソッド
    fn map<U, F>(self, func: F) -> Container<U>
    where
        F: FnOnce(T) -> U,
    {
        Container {
            value: func(self.value),
        }
    }
}

fn main() {
    let mut power_container = Container::new(1500);
    println!(\"呪力: {:?}\", power_container.get());

    power_container.set(2000);
    println!(\"強化後: {:?}\", power_container);

    // 型変換
    let name_container = power_container.map(|power| {
        format!(\"呪力値: {}\", power)
    });

    println!(\"変換後: {:?}\", name_container);
}
```

### 複雑なジェネリック構造体

```rust
#[derive(Debug)]
struct Pair<T, U> {
    first: T,
    second: U,
}

impl<T, U> Pair<T, U> {
    fn new(first: T, second: U) -> Self {
        Pair { first, second }
    }

    fn get_first(&self) -> &T {
        &self.first
    }

    fn get_second(&self) -> &U {
        &self.second
    }

    // 要素を入れ替える
    fn swap(self) -> Pair<U, T> {
        Pair {
            first: self.second,
            second: self.first,
        }
    }

    // 両方の要素に関数を適用
    fn map<V, W, F, G>(self, f: F, g: G) -> Pair<V, W>
    where
        F: FnOnce(T) -> V,
        G: FnOnce(U) -> W,
    {
        Pair {
            first: f(self.first),
            second: g(self.second),
        }
    }
}

fn main() {
    let sorcerer_info = Pair::new(\"五条悟\", 3000);
    println!(\"呪術師情報: {:?}\", sorcerer_info);

    let swapped = sorcerer_info.swap();
    println!(\"入れ替え後: {:?}\", swapped);

    let processed = Pair::new(\"虎杖悠仁\", 1200).map(
        |name| format!(\"呪術師: {}\", name),
        |power| power * 2,
    );

    println!(\"処理後: {:?}\", processed);
}
```

## 実践例 - 汎用的な呪術師管理システム

```rust
use std::fmt::Display;
use std::collections::HashMap;

// ジェネリックなID型を持つエンティティ
#[derive(Debug, Clone, PartialEq)]
struct Entity<ID, DATA> {
    id: ID,
    data: DATA,
}

impl<ID, DATA> Entity<ID, DATA> {
    fn new(id: ID, data: DATA) -> Self {
        Entity { id, data }
    }

    fn id(&self) -> &ID {
        &self.id
    }

    fn data(&self) -> &DATA {
        &self.data
    }

    fn data_mut(&mut self) -> &mut DATA {
        &mut self.data
    }

    fn update_data(&mut self, new_data: DATA) {
        self.data = new_data;
    }
}

// 呪術師データ
#[derive(Debug, Clone)]
struct SorcererData {
    name: String,
    power: i32,
    grade: String,
    techniques: Vec<String>,
}

impl SorcererData {
    fn new(name: String, power: i32, grade: String) -> Self {
        SorcererData {
            name,
            power,
            grade,
            techniques: Vec::new(),
        }
    }

    fn add_technique(&mut self, technique: String) {
        self.techniques.push(technique);
    }
}

impl Display for SorcererData {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, \"{} (呪力: {}, 等級: {}, 技数: {})\",
               self.name, self.power, self.grade, self.techniques.len())
    }
}

// 汎用的なリポジトリ
#[derive(Debug)]
struct Repository<ID, DATA>
where
    ID: Clone + Eq + std::hash::Hash,
{
    entities: HashMap<ID, Entity<ID, DATA>>,
}

impl<ID, DATA> Repository<ID, DATA>
where
    ID: Clone + Eq + std::hash::Hash,
{
    fn new() -> Self {
        Repository {
            entities: HashMap::new(),
        }
    }

    fn add(&mut self, id: ID, data: DATA) -> Result<(), String> {
        if self.entities.contains_key(&id) {
            return Err(format!(\"ID {:?} は既に存在します\", id));
        }

        let entity = Entity::new(id.clone(), data);
        self.entities.insert(id, entity);
        Ok(())
    }

    fn get(&self, id: &ID) -> Option<&Entity<ID, DATA>> {
        self.entities.get(id)
    }

    fn get_mut(&mut self, id: &ID) -> Option<&mut Entity<ID, DATA>> {
        self.entities.get_mut(id)
    }

    fn remove(&mut self, id: &ID) -> Option<Entity<ID, DATA>> {
        self.entities.remove(id)
    }

    fn list_all(&self) -> Vec<&Entity<ID, DATA>> {
        self.entities.values().collect()
    }

    fn find_by<F>(&self, predicate: F) -> Vec<&Entity<ID, DATA>>
    where
        F: Fn(&Entity<ID, DATA>) -> bool,
    {
        self.entities.values().filter(|entity| predicate(entity)).collect()
    }

    fn update<F>(&mut self, id: &ID, updater: F) -> Result<(), String>
    where
        F: FnOnce(&mut DATA),
    {
        match self.entities.get_mut(id) {
            Some(entity) => {
                updater(&mut entity.data);
                Ok(())
            },
            None => Err(format!(\"ID {:?} が見つかりません\", id)),
        }
    }

    fn count(&self) -> usize {
        self.entities.len()
    }

    fn is_empty(&self) -> bool {
        self.entities.is_empty()
    }
}

// 統計機能付きリポジトリ
impl<ID, DATA> Repository<ID, DATA>
where
    ID: Clone + Eq + std::hash::Hash + Display,
    DATA: Clone,
{
    fn generate_report<F>(&self, analyzer: F) -> String
    where
        F: Fn(&[&Entity<ID, DATA>]) -> String,
    {
        let entities: Vec<&Entity<ID, DATA>> = self.list_all();
        analyzer(&entities)
    }
}

// 特化されたメソッド実装
impl Repository<String, SorcererData> {
    fn find_by_power_range(&self, min: i32, max: i32) -> Vec<&Entity<String, SorcererData>> {
        self.find_by(|entity| {
            let power = entity.data().power;
            power >= min && power <= max
        })
    }

    fn find_by_grade(&self, grade: &str) -> Vec<&Entity<String, SorcererData>> {
        self.find_by(|entity| entity.data().grade == grade)
    }

    fn get_average_power(&self) -> f64 {
        if self.is_empty() {
            return 0.0;
        }

        let total: i32 = self.entities.values()
            .map(|entity| entity.data().power)
            .sum();

        total as f64 / self.count() as f64
    }

    fn get_most_powerful(&self) -> Option<&Entity<String, SorcererData>> {
        self.entities.values()
            .max_by_key(|entity| entity.data().power)
    }
}

// 異なる型でのリポジトリ使用例
#[derive(Debug, Clone)]
struct TechniqueData {
    name: String,
    power: i32,
    element: String,
    difficulty: u8,
}

impl Display for TechniqueData {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, \"{} (威力: {}, 属性: {}, 難易度: {})\",
               self.name, self.power, self.element, self.difficulty)
    }
}

fn main() {
    println!(\"=== 汎用的な呪術師管理システム ===\");

    // 呪術師リポジトリ
    let mut sorcerer_repo: Repository<String, SorcererData> = Repository::new();

    // データの追加
    let mut gojo_data = SorcererData::new(\"五条悟\".to_string(), 3000, \"特級\".to_string());
    gojo_data.add_technique(\"無下限呪術\".to_string());
    gojo_data.add_technique(\"領域展開\".to_string());

    let mut yuji_data = SorcererData::new(\"虎杖悠仁\".to_string(), 1200, \"1級\".to_string());
    yuji_data.add_technique(\"黒閃\".to_string());

    let mut megumi_data = SorcererData::new(\"伏黒恵\".to_string(), 1000, \"2級\".to_string());
    megumi_data.add_technique(\"十種影法術\".to_string());

    // リポジトリに追加
    sorcerer_repo.add(\"gojo\".to_string(), gojo_data).unwrap();
    sorcerer_repo.add(\"yuji\".to_string(), yuji_data).unwrap();
    sorcerer_repo.add(\"megumi\".to_string(), megumi_data).unwrap();

    // 検索とフィルタリング
    println!(\"\\n=== 検索結果 ===\");

    // 特級呪術師を検索
    let special_grade = sorcerer_repo.find_by_grade(\"特級\");
    println!(\"特級呪術師: {}\", special_grade.len());
    for sorcerer in special_grade {
        println!(\"  {}: {}\", sorcerer.id(), sorcerer.data());
    }

    // 呪力範囲で検索
    let mid_power = sorcerer_repo.find_by_power_range(1000, 2000);
    println!(\"\\n呪力1000-2000の呪術師: {}\", mid_power.len());
    for sorcerer in mid_power {
        println!(\"  {}: {}\", sorcerer.id(), sorcerer.data());
    }

    // 統計情報
    println!(\"\\n=== 統計情報 ===\");
    println!(\"総呪術師数: {}\", sorcerer_repo.count());
    println!(\"平均呪力: {:.1}\", sorcerer_repo.get_average_power());

    if let Some(strongest) = sorcerer_repo.get_most_powerful() {
        println!(\"最強: {}: {}\", strongest.id(), strongest.data());
    }

    // 呪術師データの更新
    sorcerer_repo.update(&\"yuji\".to_string(), |data| {
        data.power += 300;  // パワーアップ
        data.add_technique(\"発散\".to_string());
    }).unwrap();

    println!(\"\\n=== 更新後の虎杖悠仁 ===\");
    if let Some(yuji) = sorcerer_repo.get(&\"yuji\".to_string()) {
        println!(\"{}: {}\", yuji.id(), yuji.data());
    }

    // レポート生成
    let report = sorcerer_repo.generate_report(|entities| {
        let mut report = String::from(\"=== 呪術師リポート ===\\n\");

        let total = entities.len();
        let avg_power: f64 = entities.iter()
            .map(|e| e.data().power)
            .sum::<i32>() as f64 / total as f64;

        report.push_str(&format!(\"総数: {}\\n\", total));
        report.push_str(&format!(\"平均呪力: {:.1}\\n\", avg_power));

        // 等級分布
        let mut grade_counts = std::collections::HashMap::new();
        for entity in entities {
            *grade_counts.entry(&entity.data().grade).or_insert(0) += 1;
        }

        report.push_str(\"\\n等級分布:\\n\");
        for (grade, count) in grade_counts {
            report.push_str(&format!(\"  {}: {}人\\n\", grade, count));
        }

        report
    });

    println!(\"\\n{}\", report);

    // 術式リポジトリの例
    println!(\"\\n=== 術式リポジトリ ===\");

    let mut technique_repo: Repository<u32, TechniqueData> = Repository::new();

    technique_repo.add(1, TechniqueData {
        name: \"術式順転『青』\".to_string(),
        power: 1000,
        element: \"無下限\".to_string(),
        difficulty: 8,
    }).unwrap();

    technique_repo.add(2, TechniqueData {
        name: \"黒閃\".to_string(),
        power: 800,
        element: \"物理\".to_string(),
        difficulty: 6,
    }).unwrap();

    println!(\"登録された術式:\");
    for technique in technique_repo.list_all() {
        println!(\"  ID {}: {}\", technique.id(), technique.data());
    }
}
```

## Option型とResult型のジェネリック実装

```rust
// Option型のような独自実装
#[derive(Debug, Clone, PartialEq)]
enum Maybe<T> {
    Some(T),
    None,
}

impl<T> Maybe<T> {
    fn is_some(&self) -> bool {
        matches!(self, Maybe::Some(_))
    }

    fn is_none(&self) -> bool {
        matches!(self, Maybe::None)
    }

    fn map<U, F>(self, func: F) -> Maybe<U>
    where
        F: FnOnce(T) -> U,
    {
        match self {
            Maybe::Some(value) => Maybe::Some(func(value)),
            Maybe::None => Maybe::None,
        }
    }

    fn and_then<U, F>(self, func: F) -> Maybe<U>
    where
        F: FnOnce(T) -> Maybe<U>,
    {
        match self {
            Maybe::Some(value) => func(value),
            Maybe::None => Maybe::None,
        }
    }

    fn unwrap_or(self, default: T) -> T {
        match self {
            Maybe::Some(value) => value,
            Maybe::None => default,
        }
    }
}

// Result型のような独自実装
#[derive(Debug, Clone, PartialEq)]
enum Outcome<T, E> {
    Success(T),
    Failure(E),
}

impl<T, E> Outcome<T, E> {
    fn is_success(&self) -> bool {
        matches!(self, Outcome::Success(_))
    }

    fn is_failure(&self) -> bool {
        matches!(self, Outcome::Failure(_))
    }

    fn map<U, F>(self, func: F) -> Outcome<U, E>
    where
        F: FnOnce(T) -> U,
    {
        match self {
            Outcome::Success(value) => Outcome::Success(func(value)),
            Outcome::Failure(error) => Outcome::Failure(error),
        }
    }

    fn map_err<F, G>(self, func: G) -> Outcome<T, F>
    where
        G: FnOnce(E) -> F,
    {
        match self {
            Outcome::Success(value) => Outcome::Success(value),
            Outcome::Failure(error) => Outcome::Failure(func(error)),
        }
    }

    fn and_then<U, F>(self, func: F) -> Outcome<U, E>
    where
        F: FnOnce(T) -> Outcome<U, E>,
    {
        match self {
            Outcome::Success(value) => func(value),
            Outcome::Failure(error) => Outcome::Failure(error),
        }
    }
}

fn main() {
    println!(\"=== カスタムジェネリック型 ===\");

    // Maybe型の使用例
    let power: Maybe<i32> = Maybe::Some(1500);
    let no_power: Maybe<i32> = Maybe::None;

    let doubled_power = power.map(|p| p * 2);
    let doubled_no_power = no_power.map(|p| p * 2);

    println!(\"倍増した呪力: {:?}\", doubled_power);      // Some(3000)
    println!(\"倍増した空の呪力: {:?}\", doubled_no_power); // None

    // チェーン操作
    let result = Maybe::Some(\"五条悟\")
        .map(|name| format!(\"呪術師: {}\", name))
        .map(|formatted| formatted.len());

    println!(\"チェーン結果: {:?}\", result);

    // Outcome型の使用例
    let success: Outcome<String, &str> = Outcome::Success(\"術式発動成功\".to_string());
    let failure: Outcome<String, &str> = Outcome::Failure(\"呪力不足\");

    let success_result = success.map(|msg| format!(\"[LOG] {}\", msg));
    let failure_result = failure.map_err(|err| format!(\"[ERROR] {}\", err));

    println!(\"成功結果: {:?}\", success_result);
    println!(\"失敗結果: {:?}\", failure_result);
}
```

## 型推論とターボフィッシュ

```rust
fn main() {
    println!(\"=== 型推論とターボフィッシュ ===\");

    // 型推論が効く場合
    let numbers = vec![1, 2, 3, 4, 5];
    let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();

    // 型推論が効かない場合 - ターボフィッシュ構文が必要
    let parsed_numbers: Vec<i32> = vec![\"1\", \"2\", \"3\"]
        .iter()
        .map(|s| s.parse::<i32>().unwrap())
        .collect();

    // または
    let parsed_numbers2 = vec![\"4\", \"5\", \"6\"]
        .iter()
        .map(|s| s.parse().unwrap())
        .collect::<Vec<i32>>();

    println!(\"倍増: {:?}\", doubled);
    println!(\"パース1: {:?}\", parsed_numbers);
    println!(\"パース2: {:?}\", parsed_numbers2);

    // より複雑な例
    let sorcerer_powers: std::collections::HashMap<String, i32> =
        [(\"五条悟\", 3000), (\"虎杖悠仁\", 1200)]
        .iter()
        .map(|(name, power)| (name.to_string(), *power))
        .collect();

    println!(\"呪術師の呪力: {:?}\", sorcerer_powers);
}
```

## まとめ

ジェネリクスの基本はマスターできたか？重要なポイント：

1. **型パラメータ** - `<T>`で型を抽象化
1. **型制約** - `where`句でトレイト境界を指定
1. **型推論** - コンパイラが適切な型を推論
1. **再利用性** - 同じコードで複数の型を扱える
1. **型安全性** - コンパイル時に型の整合性を保証

これで型の概念を抽象化し、無限の可能性を持つコードが書けるようになった。まさに俺の領域展開「無量空処」のように、全ての型を包含する力を手に入れたな。

次はトレイトについて学ぼう。型に振る舞いを与える、さらに強力な抽象化技術だ。

______________________________________________________________________

*「ジェネリクスを極めれば、型の制約から解放される」*
