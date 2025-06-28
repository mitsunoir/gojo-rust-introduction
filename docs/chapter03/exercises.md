# 第3章 練習問題 - 反転術式の試練

## Option型・Result型・エラーハンドリングの総合演習

第3章で学んだ反転術式編の内容を実践で確認しよう。Option型、Result型、高度なエラーハンドリング - これらは俺の反転術式のように、一見ネガティブな状況を強力な力に変える技術だ。

!!! tip "五条先生からのアドバイス"
    この章の問題は、実際のプロジェクトでよく遭遇するパターンばかりだ。
    エラーハンドリングは最初は面倒に感じるかもしれないが、一度慣れれば絶対的な安全性を提供してくれる。

## 初級編 - Option型の基本操作

### 問題1: 呪術師検索システム

呪術師のデータベースからOption型を使って安全に検索するシステムを作成せよ。

<div class=\"exercise\">

```rust
#[derive(Debug, Clone)]
struct Sorcerer {
    name: String,
    power: i32,
    grade: String,
}

struct SorcererDatabase {
    sorcerers: Vec<Sorcerer>,
}

impl SorcererDatabase {
    fn new() -> Self {
        // ここに実装
    }

    fn add_sorcerer(&mut self, sorcerer: Sorcerer) {
        // ここに実装
    }

    // 以下のメソッドを実装せよ：
    // 1. 名前で検索 (完全一致)
    // 2. 名前で検索 (部分一致)
    // 3. 最も強い呪術師を取得
    // 4. 特定の呪力以上の呪術師を検索
    // 5. 特定の等級の呪術師を検索
}

fn main() {
    // テスト用データ
    let sorcerers = vec![
        Sorcerer { name: \"五条悟\".to_string(), power: 3000, grade: \"特級\".to_string() },
        Sorcerer { name: \"両面宿儺\".to_string(), power: 2800, grade: \"特級\".to_string() },
        Sorcerer { name: \"虎杖悠仁\".to_string(), power: 1200, grade: \"1級\".to_string() },
    ];

    // システムのテスト
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
#[derive(Debug, Clone)]
struct Sorcerer {
    name: String,
    power: i32,
    grade: String,
}

struct SorcererDatabase {
    sorcerers: Vec<Sorcerer>,
}

impl SorcererDatabase {
    fn new() -> Self {
        SorcererDatabase {
            sorcerers: Vec::new(),
        }
    }

    fn add_sorcerer(&mut self, sorcerer: Sorcerer) {
        self.sorcerers.push(sorcerer);
    }

    // 1. 名前で検索 (完全一致)
    fn find_by_name(&self, name: &str) -> Option<&Sorcerer> {
        self.sorcerers.iter()
            .find(|sorcerer| sorcerer.name == name)
    }

    // 2. 名前で検索 (部分一致)
    fn find_by_partial_name(&self, partial_name: &str) -> Option<&Sorcerer> {
        self.sorcerers.iter()
            .find(|sorcerer| sorcerer.name.contains(partial_name))
    }

    // 3. 最も強い呪術師を取得
    fn find_strongest(&self) -> Option<&Sorcerer> {
        self.sorcerers.iter()
            .max_by_key(|sorcerer| sorcerer.power)
    }

    // 4. 特定の呪力以上の呪術師を検索
    fn find_by_min_power(&self, min_power: i32) -> Option<Vec<&Sorcerer>> {
        let filtered: Vec<&Sorcerer> = self.sorcerers.iter()
            .filter(|sorcerer| sorcerer.power >= min_power)
            .collect();

        if filtered.is_empty() {
            None
        } else {
            Some(filtered)
        }
    }

    // 5. 特定の等級の呪術師を検索
    fn find_by_grade(&self, grade: &str) -> Option<Vec<&Sorcerer>> {
        let filtered: Vec<&Sorcerer> = self.sorcerers.iter()
            .filter(|sorcerer| sorcerer.grade == grade)
            .collect();

        if filtered.is_empty() {
            None
        } else {
            Some(filtered)
        }
    }

    // ボーナス: 複合検索
    fn find_by_criteria(&self, min_power: Option<i32>, grade: Option<&str>) -> Vec<&Sorcerer> {
        self.sorcerers.iter()
            .filter(|sorcerer| {
                let power_ok = min_power.map_or(true, |p| sorcerer.power >= p);
                let grade_ok = grade.map_or(true, |g| sorcerer.grade == g);
                power_ok && grade_ok
            })
            .collect()
    }
}

fn main() {
    let mut db = SorcererDatabase::new();

    // テスト用データ
    let sorcerers = vec![
        Sorcerer { name: \"五条悟\".to_string(), power: 3000, grade: \"特級\".to_string() },
        Sorcerer { name: \"両面宿儺\".to_string(), power: 2800, grade: \"特級\".to_string() },
        Sorcerer { name: \"虎杖悠仁\".to_string(), power: 1200, grade: \"1級\".to_string() },
        Sorcerer { name: \"伏黒恵\".to_string(), power: 1000, grade: \"2級\".to_string() },
        Sorcerer { name: \"釘崎野薔薇\".to_string(), power: 900, grade: \"3級\".to_string() },
    ];

    for sorcerer in sorcerers {
        db.add_sorcerer(sorcerer);
    }

    println!(\"=== 呪術師検索システム ===\");

    // 1. 完全一致検索
    match db.find_by_name(\"五条悟\") {
        Some(sorcerer) => println!(\"✓ 見つかりました: {} (呪力: {})\", sorcerer.name, sorcerer.power),
        None => println!(\"✗ 見つかりませんでした\"),
    }

    // 2. 部分一致検索
    match db.find_by_partial_name(\"虎杖\") {
        Some(sorcerer) => println!(\"✓ 部分一致: {} (呪力: {})\", sorcerer.name, sorcerer.power),
        None => println!(\"✗ 部分一致なし\"),
    }

    // 3. 最強検索
    if let Some(strongest) = db.find_strongest() {
        println!(\"✓ 最強: {} (呪力: {})\", strongest.name, strongest.power);
    }

    // 4. 呪力フィルタ
    match db.find_by_min_power(2000) {
        Some(powerful) => {
            println!(\"✓ 呪力2000以上:\");
            for sorcerer in powerful {
                println!(\"  {} (呪力: {})\", sorcerer.name, sorcerer.power);
            }
        },
        None => println!(\"✗ 該当者なし\"),
    }

    // 5. 等級検索
    match db.find_by_grade(\"特級\") {
        Some(special_grade) => {
            println!(\"✓ 特級呪術師:\");
            for sorcerer in special_grade {
                println!(\"  {} (呪力: {})\", sorcerer.name, sorcerer.power);
            }
        },
        None => println!(\"✗ 特級呪術師なし\"),
    }

    // ボーナス: 複合検索
    let criteria_result = db.find_by_criteria(Some(1000), Some(\"特級\"));
    println!(\"\\n複合検索（呪力1000以上 かつ 特級）:\");
    for sorcerer in criteria_result {
        println!(\"  {} (呪力: {}, 等級: {})\", sorcerer.name, sorcerer.power, sorcerer.grade);
    }
}
```

</div>
</details>

### 問題2: Option型のチェーン操作

複数のOption型を組み合わせて、安全にデータを処理するシステムを作成せよ。

<div class=\"exercise\">

```rust
#[derive(Debug)]
struct TechniqueInfo {
    name: String,
    power: i32,
    element: String,
}

struct TechniqueDatabase {
    techniques: Vec<TechniqueInfo>,
}

// 以下の関数を実装せよ：
// 1. 術式名から威力を取得
// 2. 威力から推奨レベルを計算
// 3. 複数の術式の合計威力を計算
// 4. 術式の組み合わせ効果を計算

fn main() {
    let techniques = vec![
        TechniqueInfo { name: \"蒼\".to_string(), power: 1000, element: \"無下限\".to_string() },
        TechniqueInfo { name: \"赫\".to_string(), power: 1500, element: \"無下限\".to_string() },
        TechniqueInfo { name: \"茈\".to_string(), power: 3000, element: \"無下限\".to_string() },
    ];

    // チェーン操作のテスト
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
#[derive(Debug)]
struct TechniqueInfo {
    name: String,
    power: i32,
    element: String,
}

struct TechniqueDatabase {
    techniques: Vec<TechniqueInfo>,
}

impl TechniqueDatabase {
    fn new(techniques: Vec<TechniqueInfo>) -> Self {
        TechniqueDatabase { techniques }
    }

    // 1. 術式名から威力を取得
    fn get_power(&self, name: &str) -> Option<i32> {
        self.techniques.iter()
            .find(|tech| tech.name == name)
            .map(|tech| tech.power)
    }

    // 2. 威力から推奨レベルを計算
    fn calculate_recommended_level(&self, power: i32) -> Option<String> {
        if power <= 0 {
            return None;
        }

        let level = match power {
            1..=500 => \"初心者\",
            501..=1000 => \"中級者\",
            1001..=2000 => \"上級者\",
            2001..=3000 => \"特級\",
            _ => \"最強級\",
        };

        Some(level.to_string())
    }

    // 3. 複数の術式の合計威力を計算
    fn calculate_total_power(&self, technique_names: &[&str]) -> Option<i32> {
        let mut total = 0;

        for name in technique_names {
            let power = self.get_power(name)?;  // 1つでも見つからなければNone
            total += power;
        }

        Some(total)
    }

    // 4. 術式の組み合わせ効果を計算
    fn calculate_combo_effect(&self, tech1: &str, tech2: &str) -> Option<(i32, String)> {
        let power1 = self.get_power(tech1)?;
        let power2 = self.get_power(tech2)?;

        let (combo_power, combo_name) = match (tech1, tech2) {
            (\"蒼\", \"赫\") | (\"赫\", \"蒼\") => {
                (power1 + power2 + 500, \"虚式『茈』\".to_string())
            },
            (a, b) if a == b => {
                (power1 * 2, format!(\"強化『{}』\", a))
            },
            (a, b) => {
                let base_power = power1 + power2;
                (base_power + (base_power / 10), format!(\"{}×{}コンボ\", a, b))
            },
        };

        Some((combo_power, combo_name))
    }

    // ボーナス: チェーン操作の実例
    fn get_technique_analysis(&self, name: &str) -> Option<String> {
        self.get_power(name)
            .and_then(|power| self.calculate_recommended_level(power))
            .map(|level| format!(\"{}は{}向けの術式です\", name, level))
    }

    // 複雑なチェーン操作
    fn analyze_combo_viability(&self, tech1: &str, tech2: &str, user_level: i32) -> Option<String> {
        self.calculate_combo_effect(tech1, tech2)
            .and_then(|(power, name)| {
                if power <= user_level * 10 {
                    Some(format!(\"{}（威力: {}）は実行可能です\", name, power))
                } else {
                    Some(format!(\"{}（威力: {}）は実行には危険すぎます\", name, power))
                }
            })
    }
}

fn main() {
    let techniques = vec![
        TechniqueInfo { name: \"蒼\".to_string(), power: 1000, element: \"無下限\".to_string() },
        TechniqueInfo { name: \"赫\".to_string(), power: 1500, element: \"無下限\".to_string() },
        TechniqueInfo { name: \"茈\".to_string(), power: 3000, element: \"無下限\".to_string() },
        TechniqueInfo { name: \"黒閃\".to_string(), power: 800, element: \"物理\".to_string() },
    ];

    let db = TechniqueDatabase::new(techniques);

    println!(\"=== Option型チェーン操作システム ===\");

    // 1. 基本的な威力取得
    let techniques_to_test = [\"蒼\", \"赫\", \"存在しない術式\"];
    for tech in techniques_to_test.iter() {
        match db.get_power(tech) {
            Some(power) => println!(\"✓ {}: 威力 {}\", tech, power),
            None => println!(\"✗ {}: 見つかりません\", tech),
        }
    }

    // 2. 推奨レベル計算のチェーン
    println!(\"\\n=== 術式分析 ===\");
    for tech in [\"蒼\", \"茈\", \"黒閃\"].iter() {
        match db.get_technique_analysis(tech) {
            Some(analysis) => println!(\"✓ {}\", analysis),
            None => println!(\"✗ {}の分析に失敗\", tech),
        }
    }

    // 3. 合計威力計算
    println!(\"\\n=== 合計威力計算 ===\");
    let combos = vec![
        vec![\"蒼\", \"赫\"],
        vec![\"蒼\", \"赫\", \"茈\"],
        vec![\"蒼\", \"存在しない術式\"],
    ];

    for combo in combos {
        match db.calculate_total_power(&combo) {
            Some(total) => {
                let level = db.calculate_recommended_level(total)
                    .unwrap_or(\"不明\".to_string());
                println!(\"✓ {:?}: 合計威力 {} ({}レベル)\", combo, total, level);
            },
            None => println!(\"✗ {:?}: 計算失敗（存在しない術式を含む）\", combo),
        }
    }

    // 4. コンボ効果計算
    println!(\"\\n=== コンボ効果分析 ===\");
    let combo_pairs = vec![
        (\"蒼\", \"赫\"),
        (\"蒼\", \"蒼\"),
        (\"茈\", \"黒閃\"),
    ];

    for (tech1, tech2) in combo_pairs {
        match db.calculate_combo_effect(tech1, tech2) {
            Some((power, name)) => {
                println!(\"✓ {} + {} = {} (威力: {})\", tech1, tech2, name, power);
            },
            None => println!(\"✗ {} + {}: コンボ計算失敗\", tech1, tech2),
        }
    }

    // 5. 複雑なチェーン操作
    println!(\"\\n=== コンボ実行可能性分析 ===\");
    let user_levels = [100, 200, 300];

    for level in user_levels.iter() {
        println!(\"\\nユーザーレベル: {}\", level);
        for (tech1, tech2) in [(\"蒼\", \"赫\"), (\"茈\", \"黒閃\")].iter() {
            match db.analyze_combo_viability(tech1, tech2, *level) {
                Some(analysis) => println!(\"  {}\", analysis),
                None => println!(\"  {}/{}: 分析失敗\", tech1, tech2),
            }
        }
    }
}
```

</div>
</details>

## 中級編 - Result型とエラーハンドリング

### 問題3: 呪術師登録システム

バリデーション機能付きの呪術師登録システムをResult型を使って実装せよ。

<div class=\"exercise\">

```rust
#[derive(Debug)]
enum RegistrationError {
    InvalidName(String),
    InvalidPower(String),
    InvalidGrade(String),
    DuplicateEntry(String),
    SystemError(String),
}

#[derive(Debug, Clone)]
struct Sorcerer {
    name: String,
    power: i32,
    grade: String,
    techniques: Vec<String>,
}

struct RegistrationSystem {
    sorcerers: Vec<Sorcerer>,
}

impl RegistrationSystem {
    fn new() -> Self {
        // 実装
    }

    // 以下のメソッドを実装せよ：
    // 1. 名前のバリデーション
    // 2. 呪力のバリデーション
    // 3. 等級のバリデーション
    // 4. 重複チェック
    // 5. 呪術師の登録
    // 6. 術式の追加
}

fn main() {
    // テストケース
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
use std::fmt;

#[derive(Debug)]
enum RegistrationError {
    InvalidName(String),
    InvalidPower(String),
    InvalidGrade(String),
    DuplicateEntry(String),
    SystemError(String),
}

impl fmt::Display for RegistrationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            RegistrationError::InvalidName(msg) => write!(f, \"名前エラー: {}\", msg),
            RegistrationError::InvalidPower(msg) => write!(f, \"呪力エラー: {}\", msg),
            RegistrationError::InvalidGrade(msg) => write!(f, \"等級エラー: {}\", msg),
            RegistrationError::DuplicateEntry(name) => write!(f, \"重複エラー: {}は既に登録済みです\", name),
            RegistrationError::SystemError(msg) => write!(f, \"システムエラー: {}\", msg),
        }
    }
}

impl std::error::Error for RegistrationError {}

#[derive(Debug, Clone)]
struct Sorcerer {
    name: String,
    power: i32,
    grade: String,
    techniques: Vec<String>,
}

impl Sorcerer {
    fn new(name: String, power: i32, grade: String) -> Self {
        Sorcerer {
            name,
            power,
            grade,
            techniques: Vec::new(),
        }
    }
}

struct RegistrationSystem {
    sorcerers: Vec<Sorcerer>,
}

impl RegistrationSystem {
    fn new() -> Self {
        RegistrationSystem {
            sorcerers: Vec::new(),
        }
    }

    // 1. 名前のバリデーション
    fn validate_name(&self, name: &str) -> Result<(), RegistrationError> {
        if name.is_empty() {
            return Err(RegistrationError::InvalidName(\"名前が空です\".to_string()));
        }

        if name.len() < 2 {
            return Err(RegistrationError::InvalidName(\"名前は2文字以上である必要があります\".to_string()));
        }

        if name.len() > 50 {
            return Err(RegistrationError::InvalidName(\"名前は50文字以下である必要があります\".to_string()));
        }

        // 特殊文字チェック
        if name.chars().any(|c| c.is_ascii_digit()) {
            return Err(RegistrationError::InvalidName(\"名前に数字を含めることはできません\".to_string()));
        }

        Ok(())
    }

    // 2. 呪力のバリデーション
    fn validate_power(&self, power: i32) -> Result<(), RegistrationError> {
        if power < 0 {
            return Err(RegistrationError::InvalidPower(\"呪力は0以上である必要があります\".to_string()));
        }

        if power > 10000 {
            return Err(RegistrationError::InvalidPower(\"呪力は10000以下である必要があります\".to_string()));
        }

        Ok(())
    }

    // 3. 等級のバリデーション
    fn validate_grade(&self, grade: &str) -> Result<(), RegistrationError> {
        let valid_grades = [\"特級\", \"1級\", \"2級\", \"3級\", \"4級\"];

        if !valid_grades.contains(&grade) {
            return Err(RegistrationError::InvalidGrade(
                format!(\"無効な等級: {}。有効な等級: {:?}\", grade, valid_grades)
            ));
        }

        Ok(())
    }

    // 4. 重複チェック
    fn check_duplicate(&self, name: &str) -> Result<(), RegistrationError> {
        if self.sorcerers.iter().any(|s| s.name == name) {
            return Err(RegistrationError::DuplicateEntry(name.to_string()));
        }

        Ok(())
    }

    // 5. 呪術師の登録
    fn register_sorcerer(&mut self, name: String, power: i32, grade: String)
        -> Result<usize, RegistrationError> {

        // すべてのバリデーションを実行
        self.validate_name(&name)?;
        self.validate_power(power)?;
        self.validate_grade(&grade)?;
        self.check_duplicate(&name)?;

        // 等級と呪力の整合性チェック
        let min_power = match grade.as_str() {
            \"特級\" => 2000,
            \"1級\" => 1000,
            \"2級\" => 500,
            \"3級\" => 200,
            \"4級\" => 0,
            _ => return Err(RegistrationError::SystemError(\"想定外の等級\".to_string())),
        };

        if power < min_power {
            return Err(RegistrationError::InvalidPower(
                format!(\"{}等級には最低{}の呪力が必要です\", grade, min_power)
            ));
        }

        // 登録実行
        let sorcerer = Sorcerer::new(name, power, grade);
        self.sorcerers.push(sorcerer);

        Ok(self.sorcerers.len() - 1)  // インデックスを返す
    }

    // 6. 術式の追加
    fn add_technique(&mut self, sorcerer_index: usize, technique: String)
        -> Result<(), RegistrationError> {

        if technique.is_empty() {
            return Err(RegistrationError::SystemError(\"術式名が空です\".to_string()));
        }

        let sorcerer = self.sorcerers.get_mut(sorcerer_index)
            .ok_or_else(|| RegistrationError::SystemError(\"無効な呪術師インデックス\".to_string()))?;

        if sorcerer.techniques.contains(&technique) {
            return Err(RegistrationError::SystemError(
                format!(\"{}は既に{}を習得済みです\", sorcerer.name, technique)
            ));
        }

        // 等級による技数制限
        let max_techniques = match sorcerer.grade.as_str() {
            \"特級\" => 10,
            \"1級\" => 5,
            \"2級\" => 3,
            \"3級\" => 2,
            \"4級\" => 1,
            _ => return Err(RegistrationError::SystemError(\"想定外の等級\".to_string())),
        };

        if sorcerer.techniques.len() >= max_techniques {
            return Err(RegistrationError::SystemError(
                format!(\"{}等級は最大{}個の術式しか習得できません\",
                        sorcerer.grade, max_techniques)
            ));
        }

        sorcerer.techniques.push(technique);
        Ok(())
    }

    // ボーナス: 一括登録
    fn batch_register(&mut self, registrations: Vec<(String, i32, String)>)
        -> Vec<Result<usize, RegistrationError>> {

        registrations.into_iter()
            .map(|(name, power, grade)| self.register_sorcerer(name, power, grade))
            .collect()
    }

    // 呪術師情報の取得
    fn get_sorcerer(&self, index: usize) -> Result<&Sorcerer, RegistrationError> {
        self.sorcerers.get(index)
            .ok_or_else(|| RegistrationError::SystemError(\"無効なインデックス\".to_string()))
    }

    // 統計情報
    fn get_statistics(&self) -> Result<String, RegistrationError> {
        if self.sorcerers.is_empty() {
            return Err(RegistrationError::SystemError(\"登録された呪術師がいません\".to_string()));
        }

        let total = self.sorcerers.len();
        let avg_power = self.sorcerers.iter().map(|s| s.power).sum::<i32>() / total as i32;

        let mut grade_counts = std::collections::HashMap::new();
        for sorcerer in &self.sorcerers {
            *grade_counts.entry(&sorcerer.grade).or_insert(0) += 1;
        }

        let mut stats = format!(\"登録呪術師数: {}\\n平均呪力: {}\\n\\n等級分布:\\n\", total, avg_power);
        for (grade, count) in grade_counts {
            stats.push_str(&format!(\"{}: {}人\\n\", grade, count));
        }

        Ok(stats)
    }
}

fn main() {
    let mut system = RegistrationSystem::new();

    println!(\"=== 呪術師登録システム ===\");

    // 正常ケース
    let valid_registrations = vec![
        (\"五条悟\".to_string(), 3000, \"特級\".to_string()),
        (\"虎杖悠仁\".to_string(), 1200, \"1級\".to_string()),
        (\"伏黒恵\".to_string(), 800, \"2級\".to_string()),
    ];

    for (name, power, grade) in valid_registrations {
        match system.register_sorcerer(name.clone(), power, grade) {
            Ok(index) => {
                println!(\"✓ {}を登録しました (ID: {})\", name, index);

                // 術式の追加テスト
                let techniques = match name.as_str() {
                    \"五条悟\" => vec![\"無下限呪術\", \"術式順転『蒼』\"],
                    \"虎杖悠仁\" => vec![\"黒閃\"],
                    \"伏黒恵\" => vec![\"十種影法術\"],
                    _ => vec![],
                };

                for technique in techniques {
                    match system.add_technique(index, technique.to_string()) {
                        Ok(_) => println!(\"  ✓ {}を習得\", technique),
                        Err(e) => println!(\"  ✗ 術式追加エラー: {}\", e),
                    }
                }
            },
            Err(error) => println!(\"✗ {}の登録失敗: {}\", name, error),
        }
    }

    // エラーケース
    println!(\"\\n=== エラーケーステスト ===\");

    let error_cases = vec![
        (\"\".to_string(), 1000, \"1級\".to_string()),           // 空の名前
        (\"テスト1\".to_string(), 1000, \"1級\".to_string()),      // 数字を含む名前
        (\"テスト\".to_string(), -100, \"1級\".to_string()),       // 負の呪力
        (\"テスト\".to_string(), 1000, \"無効等級\".to_string()),   // 無効な等級
        (\"五条悟\".to_string(), 2000, \"特級\".to_string()),      // 重複登録
        (\"弱い特級\".to_string(), 500, \"特級\".to_string()),     // 等級と呪力の不整合
    ];

    for (name, power, grade) in error_cases {
        match system.register_sorcerer(name.clone(), power, grade.clone()) {
            Ok(index) => println!(\"✓ {}を登録しました (ID: {})\", name, index),
            Err(error) => println!(\"✗ 予期されたエラー: {}\", error),
        }
    }

    // 一括登録テスト
    println!(\"\\n=== 一括登録テスト ===\");
    let batch_registrations = vec![
        (\"新人A\".to_string(), 300, \"3級\".to_string()),
        (\"新人B\".to_string(), 600, \"2級\".to_string()),
        (\"\".to_string(), 1000, \"1級\".to_string()),  // エラーケース
    ];

    let results = system.batch_register(batch_registrations);
    for (i, result) in results.iter().enumerate() {
        match result {
            Ok(index) => println!(\"✓ 一括登録 {}: 成功 (ID: {})\", i, index),
            Err(error) => println!(\"✗ 一括登録 {}: {}\", i, error),
        }
    }

    // 統計情報
    println!(\"\\n=== 統計情報 ===\");
    match system.get_statistics() {
        Ok(stats) => println!(\"{}\", stats),
        Err(error) => println!(\"統計エラー: {}\", error),
    }
}
```

</div>
</details>

### 問題4: ファイルI/Oとエラーハンドリング

呪術師データをファイルに保存・読み込みするシステムを実装せよ。複数のエラー型を統合したカスタムエラーを使用すること。

<div class=\"exercise\">

```rust
use std::fs;
use std::io;

// カスタムエラー型を定義し、以下の機能を実装せよ：
// 1. ファイルからの呪術師データ読み込み
// 2. ファイルへの呪術師データ保存
// 3. バックアップ機能
// 4. データの整合性チェック
// 5. 複数ファイルの一括処理

fn main() {
    // テスト用コード
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
use std::fmt;
use std::fs;
use std::io;

#[derive(Debug)]
enum FileSystemError {
    IoError(io::Error),
    ParseError(String),
    ValidationError(String),
    CorruptedData(String),
    BackupError(String),
}

impl fmt::Display for FileSystemError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            FileSystemError::IoError(err) => write!(f, \"I/Oエラー: {}\", err),
            FileSystemError::ParseError(msg) => write!(f, \"パースエラー: {}\", msg),
            FileSystemError::ValidationError(msg) => write!(f, \"バリデーションエラー: {}\", msg),
            FileSystemError::CorruptedData(msg) => write!(f, \"データ破損: {}\", msg),
            FileSystemError::BackupError(msg) => write!(f, \"バックアップエラー: {}\", msg),
        }
    }
}

impl std::error::Error for FileSystemError {}

impl From<io::Error> for FileSystemError {
    fn from(err: io::Error) -> Self {
        FileSystemError::IoError(err)
    }
}

#[derive(Debug, Clone)]
struct Sorcerer {
    id: u32,
    name: String,
    power: i32,
    grade: String,
    techniques: Vec<String>,
}

impl Sorcerer {
    fn new(id: u32, name: String, power: i32, grade: String) -> Self {
        Sorcerer {
            id,
            name,
            power,
            grade,
            techniques: Vec::new(),
        }
    }

    // CSV形式での文字列化
    fn to_csv_line(&self) -> String {
        format!(\"{},{},{},{},{}\",
                self.id,
                self.name,
                self.power,
                self.grade,
                self.techniques.join(\";\"))
    }

    // CSV行からの復元
    fn from_csv_line(line: &str) -> Result<Self, FileSystemError> {
        let parts: Vec<&str> = line.split(',').collect();

        if parts.len() != 5 {
            return Err(FileSystemError::ParseError(
                format!(\"無効なCSV形式: {}\", line)
            ));
        }

        let id: u32 = parts[0].parse()
            .map_err(|_| FileSystemError::ParseError(\"IDのパースに失敗\".to_string()))?;

        let power: i32 = parts[2].parse()
            .map_err(|_| FileSystemError::ParseError(\"呪力のパースに失敗\".to_string()))?;

        let techniques = if parts[4].is_empty() {
            Vec::new()
        } else {
            parts[4].split(';').map(|s| s.to_string()).collect()
        };

        let mut sorcerer = Sorcerer::new(id, parts[1].to_string(), power, parts[3].to_string());
        sorcerer.techniques = techniques;

        Ok(sorcerer)
    }

    // データの整合性チェック
    fn validate(&self) -> Result<(), FileSystemError> {
        if self.name.is_empty() {
            return Err(FileSystemError::ValidationError(\"名前が空です\".to_string()));
        }

        if self.power < 0 {
            return Err(FileSystemError::ValidationError(\"呪力が負の値です\".to_string()));
        }

        let valid_grades = [\"特級\", \"1級\", \"2級\", \"3級\", \"4級\"];
        if !valid_grades.contains(&self.grade.as_str()) {
            return Err(FileSystemError::ValidationError(
                format!(\"無効な等級: {}\", self.grade)
            ));
        }

        Ok(())
    }
}

struct SorcererFileManager {
    filename: String,
    backup_dir: String,
}

impl SorcererFileManager {
    fn new(filename: &str, backup_dir: &str) -> Self {
        SorcererFileManager {
            filename: filename.to_string(),
            backup_dir: backup_dir.to_string(),
        }
    }

    // 1. ファイルからの読み込み
    fn load_sorcerers(&self) -> Result<Vec<Sorcerer>, FileSystemError> {
        let content = fs::read_to_string(&self.filename)?;

        let mut sorcerers = Vec::new();
        let mut line_number = 0;

        for line in content.lines() {
            line_number += 1;

            if line.trim().is_empty() {
                continue;
            }

            let sorcerer = Sorcerer::from_csv_line(line)
                .map_err(|e| FileSystemError::ParseError(
                    format!(\"行{}: {}\", line_number, e)
                ))?;

            sorcerer.validate()
                .map_err(|e| FileSystemError::CorruptedData(
                    format!(\"行{}のデータが無効: {}\", line_number, e)
                ))?;

            sorcerers.push(sorcerer);
        }

        // 重複IDチェック
        self.check_duplicate_ids(&sorcerers)?;

        Ok(sorcerers)
    }

    // 2. ファイルへの保存
    fn save_sorcerers(&self, sorcerers: &[Sorcerer]) -> Result<(), FileSystemError> {
        // 保存前にバリデーション
        for sorcerer in sorcerers {
            sorcerer.validate()?;
        }

        self.check_duplicate_ids(sorcerers)?;

        // CSV形式で保存
        let mut content = String::new();
        content.push_str(\"# 呪術師データベース\\n\");
        content.push_str(\"# ID,名前,呪力,等級,術式(;区切り)\\n\");

        for sorcerer in sorcerers {
            content.push_str(&sorcerer.to_csv_line());
            content.push('\\n');
        }

        fs::write(&self.filename, content)?;
        Ok(())
    }

    // 3. バックアップ機能
    fn create_backup(&self) -> Result<String, FileSystemError> {
        // バックアップディレクトリの作成
        fs::create_dir_all(&self.backup_dir)
            .map_err(|e| FileSystemError::BackupError(format!(\"ディレクトリ作成失敗: {}\", e)))?;

        // タイムスタンプ付きバックアップファイル名
        let timestamp = std::time::SystemTime::now()
            .duration_since(std::time::UNIX_EPOCH)
            .map_err(|e| FileSystemError::BackupError(format!(\"タイムスタンプ取得失敗: {}\", e)))?
            .as_secs();

        let backup_filename = format!(\"{}/sorcerers_backup_{}.csv\", self.backup_dir, timestamp);

        // ファイルをバックアップ
        let content = fs::read_to_string(&self.filename)
            .map_err(|e| FileSystemError::BackupError(format!(\"元ファイル読み込み失敗: {}\", e)))?;

        fs::write(&backup_filename, content)
            .map_err(|e| FileSystemError::BackupError(format!(\"バックアップ書き込み失敗: {}\", e)))?;

        Ok(backup_filename)
    }

    fn restore_from_backup(&self, backup_filename: &str) -> Result<(), FileSystemError> {
        let content = fs::read_to_string(backup_filename)
            .map_err(|e| FileSystemError::BackupError(format!(\"バックアップ読み込み失敗: {}\", e)))?;

        // バックアップデータの検証
        let _sorcerers = self.parse_content(&content)?;

        fs::write(&self.filename, content)
            .map_err(|e| FileSystemError::BackupError(format!(\"復元書き込み失敗: {}\", e)))?;

        Ok(())
    }

    // 4. データの整合性チェック
    fn verify_data_integrity(&self) -> Result<String, FileSystemError> {
        let sorcerers = self.load_sorcerers()?;

        let mut report = String::new();
        report.push_str(\"=== データ整合性レポート ===\\n\");
        report.push_str(&format!(\"総レコード数: {}\\n\", sorcerers.len()));

        // 等級分布
        let mut grade_counts = std::collections::HashMap::new();
        for sorcerer in &sorcerers {
            *grade_counts.entry(&sorcerer.grade).or_insert(0) += 1;
        }

        report.push_str(\"\\n等級分布:\\n\");
        for (grade, count) in grade_counts {
            report.push_str(&format!(\"  {}: {}人\\n\", grade, count));
        }

        // 呪力統計
        let powers: Vec<i32> = sorcerers.iter().map(|s| s.power).collect();
        let avg_power = powers.iter().sum::<i32>() / powers.len() as i32;
        let max_power = powers.iter().max().unwrap_or(&0);
        let min_power = powers.iter().min().unwrap_or(&0);

        report.push_str(&format!(\"\\n呪力統計:\\n\"));
        report.push_str(&format!(\"  平均: {}\\n\", avg_power));
        report.push_str(&format!(\"  最大: {}\\n\", max_power));
        report.push_str(&format!(\"  最小: {}\\n\", min_power));

        // 異常データのチェック
        let mut anomalies = Vec::new();
        for sorcerer in &sorcerers {
            let expected_min_power = match sorcerer.grade.as_str() {
                \"特級\" => 2000,
                \"1級\" => 1000,
                \"2級\" => 500,
                \"3級\" => 200,
                \"4級\" => 0,
                _ => 0,
            };

            if sorcerer.power < expected_min_power {
                anomalies.push(format!(\"{}(ID:{}): {}等級なのに呪力が{}しかない\",
                    sorcerer.name, sorcerer.id, sorcerer.grade, sorcerer.power));
            }
        }

        if !anomalies.is_empty() {
            report.push_str(\"\\n⚠️ 異常データ:\\n\");
            for anomaly in anomalies {
                report.push_str(&format!(\"  {}\", anomaly));
            }
        } else {
            report.push_str(\"\\n✅ 異常データは見つかりませんでした\\n\");
        }

        Ok(report)
    }

    // 5. 複数ファイルの一括処理
    fn batch_process_files(&self, file_patterns: &[&str]) -> Result<Vec<String>, FileSystemError> {
        let mut results = Vec::new();

        for pattern in file_patterns {
            let files = glob::glob(pattern)
                .map_err(|e| FileSystemError::IoError(
                    io::Error::new(io::ErrorKind::InvalidInput, format!(\"Globパターンエラー: {}\", e))
                ))?;

            for file_result in files {
                let file_path = file_result
                    .map_err(|e| FileSystemError::IoError(
                        io::Error::new(io::ErrorKind::NotFound, format!(\"ファイルパスエラー: {}\", e))
                    ))?;

                let temp_manager = SorcererFileManager::new(
                    file_path.to_str().unwrap_or(\"unknown\"),
                    &self.backup_dir
                );

                match temp_manager.verify_data_integrity() {
                    Ok(report) => {
                        results.push(format!(\"✅ {}: 整合性チェック完了\",
                                           file_path.display()));
                    },
                    Err(e) => {
                        results.push(format!(\"❌ {}: {}\",
                                           file_path.display(), e));
                    }
                }
            }
        }

        Ok(results)
    }

    // ヘルパーメソッド
    fn check_duplicate_ids(&self, sorcerers: &[Sorcerer]) -> Result<(), FileSystemError> {
        let mut seen_ids = std::collections::HashSet::new();

        for sorcerer in sorcerers {
            if !seen_ids.insert(sorcerer.id) {
                return Err(FileSystemError::CorruptedData(
                    format!(\"重複したID: {}\", sorcerer.id)
                ));
            }
        }

        Ok(())
    }

    fn parse_content(&self, content: &str) -> Result<Vec<Sorcerer>, FileSystemError> {
        let mut sorcerers = Vec::new();

        for line in content.lines() {
            if line.trim().is_empty() || line.starts_with('#') {
                continue;
            }

            let sorcerer = Sorcerer::from_csv_line(line)?;
            sorcerer.validate()?;
            sorcerers.push(sorcerer);
        }

        Ok(sorcerers)
    }
}

fn main() {
    let manager = SorcererFileManager::new(\"sorcerers.csv\", \"backups\");

    println!(\"=== 呪術師ファイル管理システム ===\");

    // テスト用データの作成
    let mut test_sorcerers = vec![
        Sorcerer::new(1, \"五条悟\".to_string(), 3000, \"特級\".to_string()),
        Sorcerer::new(2, \"虎杖悠仁\".to_string(), 1200, \"1級\".to_string()),
        Sorcerer::new(3, \"伏黒恵\".to_string(), 800, \"2級\".to_string()),
    ];

    // 術式の追加
    test_sorcerers[0].techniques.push(\"無下限呪術\".to_string());
    test_sorcerers[0].techniques.push(\"術式順転『蒼』\".to_string());
    test_sorcerers[1].techniques.push(\"黒閃\".to_string());
    test_sorcerers[2].techniques.push(\"十種影法術\".to_string());

    // 1. データの保存
    match manager.save_sorcerers(&test_sorcerers) {
        Ok(_) => println!(\"✅ データ保存完了\"),
        Err(e) => println!(\"❌ 保存エラー: {}\", e),
    }

    // 2. データの読み込み
    match manager.load_sorcerers() {
        Ok(sorcerers) => {
            println!(\"✅ データ読み込み完了: {}件\", sorcerers.len());
            for sorcerer in &sorcerers[..2] {  // 最初の2件のみ表示
                println!(\"  {} (ID:{}, 呪力:{}, 等級:{})\",
                         sorcerer.name, sorcerer.id, sorcerer.power, sorcerer.grade);
            }
        },
        Err(e) => println!(\"❌ 読み込みエラー: {}\", e),
    }

    // 3. バックアップの作成
    match manager.create_backup() {
        Ok(backup_file) => println!(\"✅ バックアップ作成: {}\", backup_file),
        Err(e) => println!(\"❌ バックアップエラー: {}\", e),
    }

    // 4. データ整合性チェック
    match manager.verify_data_integrity() {
        Ok(report) => println!(\"\\n{}\", report),
        Err(e) => println!(\"❌ 整合性チェックエラー: {}\", e),
    }

    // 5. エラーケースのテスト
    println!(\"\\n=== エラーケーステスト ===\");

    // 不正なデータでテスト
    let invalid_content = \"1,五条悟,3000,特級,無下限呪術\\ninvalid,line,format\\n\";
    if let Err(e) = fs::write(\"invalid_test.csv\", invalid_content) {
        println!(\"テストファイル作成エラー: {}\", e);
    } else {
        let invalid_manager = SorcererFileManager::new(\"invalid_test.csv\", \"backups\");
        match invalid_manager.load_sorcerers() {
            Ok(_) => println!(\"❌ 無効データが読み込まれてしまいました\"),
            Err(e) => println!(\"✅ 予期されたエラー: {}\", e),
        }

        // テストファイルの削除
        let _ = fs::remove_file(\"invalid_test.csv\");
    }

    println!(\"\\n✅ ファイル管理システムテスト完了\");
}

// 注意: このコードを実行するには Cargo.toml に以下を追加してください:
// [dependencies]
// glob = \"0.3\"
```

</div>
</details>

## 上級編 - 高度なエラーハンドリングパターン

### 問題5: マルチスレッド対応エラーハンドリング

複数のスレッドで並行処理を行い、各スレッドのエラーを適切に集約するシステムを実装せよ。

<div class=\"exercise\">

```rust
use std::sync::{Arc, Mutex};
use std::thread;

// マルチスレッドでの呪術師データ処理システム
// 要件：
// 1. 複数スレッドでデータを並行処理
// 2. 各スレッドのエラーを収集
// 3. 処理結果とエラーの統合
// 4. 安全なリソース共有

fn main() {
    // 実装
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;
use std::time::Duration;

#[derive(Debug, Clone)]
enum ProcessingError {
    InvalidData(String),
    CalculationError(String),
    ThreadError(String),
    TimeoutError,
}

impl std::fmt::Display for ProcessingError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            ProcessingError::InvalidData(msg) => write!(f, \"データエラー: {}\", msg),
            ProcessingError::CalculationError(msg) => write!(f, \"計算エラー: {}\", msg),
            ProcessingError::ThreadError(msg) => write!(f, \"スレッドエラー: {}\", msg),
            ProcessingError::TimeoutError => write!(f, \"タイムアウトエラー\"),
        }
    }
}

impl std::error::Error for ProcessingError {}

#[derive(Debug, Clone)]
struct SorcererData {
    id: u32,
    name: String,
    power: i32,
    techniques: Vec<String>,
}

impl SorcererData {
    fn new(id: u32, name: String, power: i32) -> Self {
        SorcererData {
            id,
            name,
            power,
            techniques: Vec::new(),
        }
    }

    // 複雑な計算処理をシミュレート
    fn calculate_combat_rating(&self) -> Result<f64, ProcessingError> {
        if self.power <= 0 {
            return Err(ProcessingError::InvalidData(
                format!(\"無効な呪力: {}\", self.power)
            ));
        }

        // 意図的に時間のかかる処理
        thread::sleep(Duration::from_millis(100));

        let base_rating = self.power as f64;
        let technique_bonus = self.techniques.len() as f64 * 100.0;

        // 意図的なエラーケース
        if self.name.contains(\"エラー\") {
            return Err(ProcessingError::CalculationError(
                \"計算中にエラーが発生しました\".to_string()
            ));
        }

        let rating = base_rating + technique_bonus;

        if rating > 10000.0 {
            Err(ProcessingError::CalculationError(
                \"戦闘力が上限を超えました\".to_string()
            ))
        } else {
            Ok(rating)
        }
    }
}

#[derive(Debug)]
struct ProcessingResult {
    sorcerer_id: u32,
    result: Result<f64, ProcessingError>,
    thread_id: thread::ThreadId,
}

struct ParallelProcessor {
    thread_count: usize,
    timeout_ms: u64,
}

impl ParallelProcessor {
    fn new(thread_count: usize, timeout_ms: u64) -> Self {
        ParallelProcessor {
            thread_count,
            timeout_ms,
        }
    }

    fn process_sorcerers(&self, sorcerers: Vec<SorcererData>)
        -> Result<Vec<ProcessingResult>, ProcessingError> {

        let sorcerers = Arc::new(Mutex::new(sorcerers));
        let (tx, rx) = mpsc::channel();
        let mut handles = Vec::new();

        // ワーカースレッドの起動
        for worker_id in 0..self.thread_count {
            let sorcerers_clone = Arc::clone(&sorcerers);
            let tx_clone = tx.clone();

            let handle = thread::spawn(move || {
                let thread_id = thread::current().id();

                loop {
                    // 次の処理対象を取得
                    let sorcerer = {
                        let mut sorcerers_guard = sorcerers_clone.lock()
                            .map_err(|_| ProcessingError::ThreadError(\"ロック取得失敗\".to_string()))?;

                        sorcerers_guard.pop()
                    };

                    match sorcerer {
                        Some(sorcerer) => {
                            let sorcerer_id = sorcerer.id;
                            let result = sorcerer.calculate_combat_rating();

                            let processing_result = ProcessingResult {
                                sorcerer_id,
                                result,
                                thread_id,
                            };

                            if tx_clone.send(processing_result).is_err() {
                                break; // チャンネルが閉じられた
                            }
                        },
                        None => break, // 処理する項目がない
                    }
                }

                Ok::<(), ProcessingError>(())
            });

            handles.push(handle);
        }

        // 送信者のクローンを削除（受信者が終了を検知できるように）
        drop(tx);

        // 結果の収集
        let mut results = Vec::new();
        let start_time = std::time::Instant::now();

        while let Ok(result) = rx.recv() {
            results.push(result);

            // タイムアウトチェック
            if start_time.elapsed().as_millis() > self.timeout_ms as u128 {
                return Err(ProcessingError::TimeoutError);
            }
        }

        // スレッドの終了を待機
        for handle in handles {
            match handle.join() {
                Ok(Ok(())) => {},
                Ok(Err(e)) => return Err(e),
                Err(_) => return Err(ProcessingError::ThreadError(\"スレッド終了エラー\".to_string())),
            }
        }

        Ok(results)
    }

    fn analyze_results(&self, results: &[ProcessingResult]) -> ProcessingSummary {
        let mut successful = 0;
        let mut failed = 0;
        let mut total_rating = 0.0;
        let mut errors_by_type = std::collections::HashMap::new();
        let mut results_by_thread = std::collections::HashMap::new();

        for result in results {
            // スレッド別統計
            let thread_stats = results_by_thread.entry(result.thread_id).or_insert((0, 0));

            match &result.result {
                Ok(rating) => {
                    successful += 1;
                    total_rating += rating;
                    thread_stats.0 += 1;
                },
                Err(error) => {
                    failed += 1;
                    thread_stats.1 += 1;

                    let error_type = match error {
                        ProcessingError::InvalidData(_) => \"InvalidData\",
                        ProcessingError::CalculationError(_) => \"CalculationError\",
                        ProcessingError::ThreadError(_) => \"ThreadError\",
                        ProcessingError::TimeoutError => \"TimeoutError\",
                    };

                    *errors_by_type.entry(error_type).or_insert(0) += 1;
                },
            }
        }

        ProcessingSummary {
            total_processed: results.len(),
            successful,
            failed,
            average_rating: if successful > 0 { total_rating / successful as f64 } else { 0.0 },
            errors_by_type,
            results_by_thread,
        }
    }
}

#[derive(Debug)]
struct ProcessingSummary {
    total_processed: usize,
    successful: usize,
    failed: usize,
    average_rating: f64,
    errors_by_type: std::collections::HashMap<&'static str, i32>,
    results_by_thread: std::collections::HashMap<thread::ThreadId, (i32, i32)>, // (成功数, 失敗数)
}

impl ProcessingSummary {
    fn generate_report(&self) -> String {
        let mut report = String::new();

        report.push_str(\"=== 並行処理結果レポート ===\\n\");
        report.push_str(&format!(\"総処理数: {}\\n\", self.total_processed));
        report.push_str(&format!(\"成功: {} ({:.1}%)\\n\",
            self.successful,
            (self.successful as f64 / self.total_processed as f64) * 100.0));
        report.push_str(&format!(\"失敗: {} ({:.1}%)\\n\",
            self.failed,
            (self.failed as f64 / self.total_processed as f64) * 100.0));

        if self.successful > 0 {
            report.push_str(&format!(\"平均戦闘力: {:.1}\\n\", self.average_rating));
        }

        if !self.errors_by_type.is_empty() {
            report.push_str(\"\\nエラー種別:\\n\");
            for (error_type, count) in &self.errors_by_type {
                report.push_str(&format!(\"  {}: {}件\\n\", error_type, count));
            }
        }

        report.push_str(&format!(\"\\nスレッド別処理結果:\\n\"));
        for (thread_id, (success, failure)) in &self.results_by_thread {
            report.push_str(&format!(\"  {:?}: 成功{}件, 失敗{}件\\n\",
                thread_id, success, failure));
        }

        report
    }
}

fn create_test_data() -> Vec<SorcererData> {
    let mut sorcerers = vec![
        SorcererData::new(1, \"五条悟\".to_string(), 3000),
        SorcererData::new(2, \"両面宿儺\".to_string(), 2800),
        SorcererData::new(3, \"虎杖悠仁\".to_string(), 1200),
        SorcererData::new(4, \"伏黒恵\".to_string(), 1000),
        SorcererData::new(5, \"釘崎野薔薇\".to_string(), 900),
        SorcererData::new(6, \"禪院真希\".to_string(), 800),
        SorcererData::new(7, \"狗巻棘\".to_string(), 700),
        SorcererData::new(8, \"パンダ\".to_string(), 600),
        SorcererData::new(9, \"エラーテスト\".to_string(), 1000), // 意図的なエラー
        SorcererData::new(10, \"無効呪力\".to_string(), -100),     // 無効データ
    ];

    // 術式の追加
    sorcerers[0].techniques.extend(vec![\"無下限呪術\".to_string(), \"領域展開\".to_string()]);
    sorcerers[1].techniques.extend(vec![\"解\".to_string(), \"捌\".to_string()]);
    sorcerers[2].techniques.push(\"黒閃\".to_string());
    sorcerers[3].techniques.push(\"十種影法術\".to_string());

    // 追加のテストデータ
    for i in 11..=20 {
        let mut sorcerer = SorcererData::new(i, format!(\"呪術師{}\", i), 500 + (i as i32 * 50));
        sorcerer.techniques.push(format!(\"術式{}\", i));
        sorcerers.push(sorcerer);
    }

    sorcerers
}

fn main() {
    println!(\"=== マルチスレッド呪術師処理システム ===\");

    let processor = ParallelProcessor::new(4, 5000); // 4スレッド、5秒タイムアウト
    let test_data = create_test_data();

    println!(\"処理対象: {}件の呪術師データ\", test_data.len());
    println!(\"スレッド数: {}\", processor.thread_count);
    println!(\"タイムアウト: {}ms\\n\", processor.timeout_ms);

    let start_time = std::time::Instant::now();

    match processor.process_sorcerers(test_data) {
        Ok(results) => {
            let elapsed = start_time.elapsed();
            println!(\"✅ 処理完了（実行時間: {:.2}秒）\\n\", elapsed.as_secs_f64());

            // 結果の詳細表示（最初の5件）
            println!(\"=== 処理結果詳細（最初の5件）===\");
            for (i, result) in results.iter().take(5).enumerate() {
                match &result.result {
                    Ok(rating) => {
                        println!(\"{}. ID {}: 戦闘力 {:.1} (スレッド: {:?})\",
                                 i + 1, result.sorcerer_id, rating, result.thread_id);
                    },
                    Err(error) => {
                        println!(\"{}. ID {}: エラー - {} (スレッド: {:?})\",
                                 i + 1, result.sorcerer_id, error, result.thread_id);
                    }
                }
            }

            // 統計分析
            let summary = processor.analyze_results(&results);
            println!(\"\\n{}\", summary.generate_report());

        },
        Err(error) => {
            println!(\"❌ 処理失敗: {}\", error);
        }
    }

    // パフォーマンステスト
    println!(\"\\n=== パフォーマンステスト ===\");

    let large_dataset: Vec<SorcererData> = (1..=1000)
        .map(|i| {
            let mut sorcerer = SorcererData::new(i, format!(\"呪術師{}\", i), 500 + (i % 1000));
            sorcerer.techniques.push(format!(\"術式{}\", i));
            sorcerer
        })
        .collect();

    println!(\"大規模データセット: {}件\", large_dataset.len());

    let large_processor = ParallelProcessor::new(8, 10000); // 8スレッド、10秒タイムアウト
    let start_time = std::time::Instant::now();

    match large_processor.process_sorcerers(large_dataset) {
        Ok(results) => {
            let elapsed = start_time.elapsed();
            println!(\"✅ 大規模処理完了（実行時間: {:.2}秒）\", elapsed.as_secs_f64());

            let summary = large_processor.analyze_results(&results);
            println!(\"\\n{}\", summary.generate_report());
        },
        Err(error) => {
            println!(\"❌ 大規模処理失敗: {}\", error);
        }
    }
}
```

</div>
</details>

## 総合問題

### 問題6: 呪術師管理マイクロサービス

これまで学んだすべての要素を統合して、マイクロサービス風の呪術師管理システムを実装せよ。

**要件:**

- RESTful API風のインターフェース
- 複数のサービス間でのエラー伝播
- 設定管理とバリデーション
- ログ記録システム
- リトライ機構

<div class=\"exercise\">

自由に設計して実装してみよう！すべてのエラーハンドリング技術を駆使すること。

</div>

<details>
<summary>解答例を見る</summary>
<div class=\"solution\">

```rust
use std::collections::HashMap;
use std::fmt;
use std::time::{Duration, Instant};

// サービス層のエラー型
#[derive(Debug)]
enum ServiceError {
    ValidationError(String),
    NotFound(String),
    Conflict(String),
    InternalError(String),
    ExternalServiceError(String),
    TimeoutError,
    RateLimitExceeded,
}

impl fmt::Display for ServiceError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ServiceError::ValidationError(msg) => write!(f, \"[400] Validation: {}\", msg),
            ServiceError::NotFound(resource) => write!(f, \"[404] Not Found: {}\", resource),
            ServiceError::Conflict(msg) => write!(f, \"[409] Conflict: {}\", msg),
            ServiceError::InternalError(msg) => write!(f, \"[500] Internal: {}\", msg),
            ServiceError::ExternalServiceError(msg) => write!(f, \"[502] External: {}\", msg),
            ServiceError::TimeoutError => write!(f, \"[504] Timeout\"),
            ServiceError::RateLimitExceeded => write!(f, \"[429] Rate Limit Exceeded\"),
        }
    }
}

impl std::error::Error for ServiceError {}

// APIレスポンス型
#[derive(Debug)]
struct ApiResponse<T> {
    success: bool,
    data: Option<T>,
    error: Option<String>,
    timestamp: String,
    request_id: String,
}

impl<T> ApiResponse<T> {
    fn success(data: T, request_id: String) -> Self {
        ApiResponse {
            success: true,
            data: Some(data),
            error: None,
            timestamp: chrono::Utc::now().to_rfc3339(),
            request_id,
        }
    }

    fn error(error: ServiceError, request_id: String) -> ApiResponse<()> {
        ApiResponse {
            success: false,
            data: None,
            error: Some(error.to_string()),
            timestamp: chrono::Utc::now().to_rfc3339(),
            request_id,
        }
    }
}

// 設定管理
#[derive(Debug, Clone)]
struct ServiceConfig {
    max_retry_attempts: u32,
    retry_delay_ms: u64,
    timeout_ms: u64,
    rate_limit_per_minute: u32,
    log_level: String,
}

impl Default for ServiceConfig {
    fn default() -> Self {
        ServiceConfig {
            max_retry_attempts: 3,
            retry_delay_ms: 1000,
            timeout_ms: 5000,
            rate_limit_per_minute: 100,
            log_level: \"INFO\".to_string(),
        }
    }
}

// ログ記録システム
struct Logger {
    level: String,
}

impl Logger {
    fn new(level: String) -> Self {
        Logger { level }
    }

    fn info(&self, message: &str, context: &HashMap<String, String>) {
        println!(\"[INFO] {} | {:?}\", message, context);
    }

    fn warn(&self, message: &str, context: &HashMap<String, String>) {
        println!(\"[WARN] {} | {:?}\", message, context);
    }

    fn error(&self, message: &str, error: &dyn std::error::Error, context: &HashMap<String, String>) {
        println!(\"[ERROR] {} | Error: {} | {:?}\", message, error, context);
    }
}

// レート制限管理
struct RateLimiter {
    requests: HashMap<String, Vec<Instant>>,
    limit_per_minute: u32,
}

impl RateLimiter {
    fn new(limit_per_minute: u32) -> Self {
        RateLimiter {
            requests: HashMap::new(),
            limit_per_minute,
        }
    }

    fn check_rate_limit(&mut self, client_id: &str) -> Result<(), ServiceError> {
        let now = Instant::now();
        let minute_ago = now - Duration::from_secs(60);

        let requests = self.requests.entry(client_id.to_string()).or_insert_with(Vec::new);

        // 1分以上前のリクエストを削除
        requests.retain(|&time| time > minute_ago);

        if requests.len() >= self.limit_per_minute as usize {
            return Err(ServiceError::RateLimitExceeded);
        }

        requests.push(now);
        Ok(())
    }
}

// リトライ機構
struct RetryPolicy {
    max_attempts: u32,
    delay_ms: u64,
}

impl RetryPolicy {
    fn new(max_attempts: u32, delay_ms: u64) -> Self {
        RetryPolicy { max_attempts, delay_ms }
    }

    fn execute<T, F>(&self, operation: F) -> Result<T, ServiceError>
    where
        F: Fn() -> Result<T, ServiceError>,
    {
        let mut last_error = ServiceError::InternalError(\"No attempts made\".to_string());

        for attempt in 1..=self.max_attempts {
            match operation() {
                Ok(result) => return Ok(result),
                Err(error) => {
                    last_error = error;
                    if attempt < self.max_attempts {
                        std::thread::sleep(Duration::from_millis(self.delay_ms * attempt as u64));
                    }
                }
            }
        }

        Err(last_error)
    }
}

// 呪術師エンティティ
#[derive(Debug, Clone)]
struct Sorcerer {
    id: String,
    name: String,
    power: i32,
    grade: String,
    techniques: Vec<String>,
    active: bool,
    created_at: String,
    updated_at: String,
}

impl Sorcerer {
    fn new(id: String, name: String, power: i32, grade: String) -> Result<Self, ServiceError> {
        if name.is_empty() {
            return Err(ServiceError::ValidationError(\"名前は必須です\".to_string()));
        }

        let valid_grades = [\"特級\", \"1級\", \"2級\", \"3級\", \"4級\"];
        if !valid_grades.contains(&grade.as_str()) {
            return Err(ServiceError::ValidationError(format!(\"無効な等級: {}\", grade)));
        }

        if power < 0 || power > 10000 {
            return Err(ServiceError::ValidationError(\"呪力は0-10000の範囲である必要があります\".to_string()));
        }

        let now = chrono::Utc::now().to_rfc3339();

        Ok(Sorcerer {
            id,
            name,
            power,
            grade,
            techniques: Vec::new(),
            active: true,
            created_at: now.clone(),
            updated_at: now,
        })
    }

    fn add_technique(&mut self, technique: String) -> Result<(), ServiceError> {
        if technique.is_empty() {
            return Err(ServiceError::ValidationError(\"術式名は必須です\".to_string()));
        }

        if self.techniques.contains(&technique) {
            return Err(ServiceError::Conflict(format!(\"術式'{}'は既に習得済みです\", technique)));
        }

        self.techniques.push(technique);
        self.updated_at = chrono::Utc::now().to_rfc3339();
        Ok(())
    }
}

// データストア（簡易版）
struct SorcererStore {
    data: HashMap<String, Sorcerer>,
    next_id: u32,
}

impl SorcererStore {
    fn new() -> Self {
        SorcererStore {
            data: HashMap::new(),
            next_id: 1,
        }
    }

    fn create(&mut self, name: String, power: i32, grade: String) -> Result<Sorcerer, ServiceError> {
        let id = format!(\"sorcerer_{}\", self.next_id);
        self.next_id += 1;

        // 重複チェック
        if self.data.values().any(|s| s.name == name) {
            return Err(ServiceError::Conflict(format!(\"名前'{}'は既に使用されています\", name)));
        }

        let sorcerer = Sorcerer::new(id.clone(), name, power, grade)?;
        self.data.insert(id.clone(), sorcerer.clone());

        Ok(sorcerer)
    }

    fn get(&self, id: &str) -> Result<&Sorcerer, ServiceError> {
        self.data.get(id)
            .ok_or_else(|| ServiceError::NotFound(format!(\"呪術師ID: {}\", id)))
    }

    fn get_mut(&mut self, id: &str) -> Result<&mut Sorcerer, ServiceError> {
        self.data.get_mut(id)
            .ok_or_else(|| ServiceError::NotFound(format!(\"呪術師ID: {}\", id)))
    }

    fn list(&self) -> Vec<&Sorcerer> {
        self.data.values().filter(|s| s.active).collect()
    }

    fn update(&mut self, id: &str, update_fn: impl FnOnce(&mut Sorcerer) -> Result<(), ServiceError>)
        -> Result<&Sorcerer, ServiceError> {

        let sorcerer = self.get_mut(id)?;
        update_fn(sorcerer)?;
        sorcerer.updated_at = chrono::Utc::now().to_rfc3339();

        Ok(&*sorcerer)
    }

    fn delete(&mut self, id: &str) -> Result<(), ServiceError> {
        let sorcerer = self.get_mut(id)?;
        sorcerer.active = false;
        sorcerer.updated_at = chrono::Utc::now().to_rfc3339();
        Ok(())
    }
}

// 外部サービス（モック）
struct ExternalGradeService;

impl ExternalGradeService {
    fn validate_grade_eligibility(&self, power: i32, grade: &str) -> Result<bool, ServiceError> {
        // 外部サービス呼び出しをシミュレート
        std::thread::sleep(Duration::from_millis(100));

        // 意図的にエラーを発生させる場合
        if power == 9999 {
            return Err(ServiceError::ExternalServiceError(\"等級サービスが利用できません\".to_string()));
        }

        let min_power = match grade {
            \"特級\" => 2000,
            \"1級\" => 1000,
            \"2級\" => 500,
            \"3級\" => 200,
            \"4級\" => 0,
            _ => return Ok(false),
        };

        Ok(power >= min_power)
    }
}

// メインサービス
struct SorcererService {
    store: SorcererStore,
    config: ServiceConfig,
    logger: Logger,
    rate_limiter: RateLimiter,
    retry_policy: RetryPolicy,
    grade_service: ExternalGradeService,
}

impl SorcererService {
    fn new(config: ServiceConfig) -> Self {
        let logger = Logger::new(config.log_level.clone());
        let rate_limiter = RateLimiter::new(config.rate_limit_per_minute);
        let retry_policy = RetryPolicy::new(config.max_retry_attempts, config.retry_delay_ms);

        SorcererService {
            store: SorcererStore::new(),
            config,
            logger,
            rate_limiter,
            retry_policy,
            grade_service: ExternalGradeService,
        }
    }

    fn create_sorcerer(&mut self, request_id: String, client_id: String, name: String, power: i32, grade: String)
        -> ApiResponse<Sorcerer> {

        let mut context = HashMap::new();
        context.insert(\"request_id\".to_string(), request_id.clone());
        context.insert(\"client_id\".to_string(), client_id.clone());
        context.insert(\"name\".to_string(), name.clone());

        // レート制限チェック
        if let Err(error) = self.rate_limiter.check_rate_limit(&client_id) {
            self.logger.warn(\"Rate limit exceeded\", &context);
            return ApiResponse::error(error, request_id);
        }

        // 外部サービスでの等級検証（リトライ付き）
        let grade_validation = self.retry_policy.execute(|| {
            self.grade_service.validate_grade_eligibility(power, &grade)
        });

        match grade_validation {
            Ok(is_eligible) => {
                if !is_eligible {
                    let error = ServiceError::ValidationError(
                        format!(\"呪力{}では{}等級の資格がありません\", power, grade)
                    );
                    self.logger.warn(\"Grade validation failed\", &context);
                    return ApiResponse::error(error, request_id);
                }
            },
            Err(error) => {
                self.logger.error(\"External grade validation failed\", &error, &context);
                return ApiResponse::error(error, request_id);
            }
        }

        // 呪術師作成
        match self.store.create(name, power, grade) {
            Ok(sorcerer) => {
                self.logger.info(\"Sorcerer created successfully\", &context);
                ApiResponse::success(sorcerer, request_id)
            },
            Err(error) => {
                self.logger.error(\"Failed to create sorcerer\", &error, &context);
                ApiResponse::error(error, request_id)
            }
        }
    }

    fn get_sorcerer(&mut self, request_id: String, client_id: String, id: String)
        -> ApiResponse<Sorcerer> {

        let mut context = HashMap::new();
        context.insert(\"request_id\".to_string(), request_id.clone());
        context.insert(\"client_id\".to_string(), client_id.clone());
        context.insert(\"sorcerer_id\".to_string(), id.clone());

        if let Err(error) = self.rate_limiter.check_rate_limit(&client_id) {
            return ApiResponse::error(error, request_id);
        }

        match self.store.get(&id) {
            Ok(sorcerer) => {
                self.logger.info(\"Sorcerer retrieved\", &context);
                ApiResponse::success(sorcerer.clone(), request_id)
            },
            Err(error) => {
                self.logger.warn(\"Sorcerer not found\", &context);
                ApiResponse::error(error, request_id)
            }
        }
    }

    fn add_technique(&mut self, request_id: String, client_id: String, id: String, technique: String)
        -> ApiResponse<Sorcerer> {

        let mut context = HashMap::new();
        context.insert(\"request_id\".to_string(), request_id.clone());
        context.insert(\"sorcerer_id\".to_string(), id.clone());
        context.insert(\"technique\".to_string(), technique.clone());

        if let Err(error) = self.rate_limiter.check_rate_limit(&client_id) {
            return ApiResponse::error(error, request_id);
        }

        let result = self.store.update(&id, |sorcerer| {
            sorcerer.add_technique(technique)
        });

        match result {
            Ok(sorcerer) => {
                self.logger.info(\"Technique added successfully\", &context);
                ApiResponse::success(sorcerer.clone(), request_id)
            },
            Err(error) => {
                self.logger.error(\"Failed to add technique\", &error, &context);
                ApiResponse::error(error, request_id)
            }
        }
    }

    fn list_sorcerers(&mut self, request_id: String, client_id: String)
        -> ApiResponse<Vec<Sorcerer>> {

        if let Err(error) = self.rate_limiter.check_rate_limit(&client_id) {
            return ApiResponse::error(error, request_id);
        }

        let sorcerers: Vec<Sorcerer> = self.store.list().into_iter().cloned().collect();

        let mut context = HashMap::new();
        context.insert(\"request_id\".to_string(), request_id.clone());
        context.insert(\"count\".to_string(), sorcerers.len().to_string());

        self.logger.info(\"Sorcerers listed\", &context);
        ApiResponse::success(sorcerers, request_id)
    }
}

fn main() {
    println!(\"=== 呪術師管理マイクロサービス ===\");

    let config = ServiceConfig::default();
    let mut service = SorcererService::new(config);

    // テストクライアント
    let client_id = \"test_client\".to_string();
    let mut request_counter = 1;

    // 呪術師作成テスト
    println!(\"\\n1. 呪術師作成テスト\");

    let test_cases = vec![
        (\"五条悟\", 3000, \"特級\"),
        (\"虎杖悠仁\", 1200, \"1級\"),
        (\"無効テスト\", 100, \"特級\"), // 等級不適格
        (\"エラーテスト\", 9999, \"特級\"), // 外部サービスエラー
    ];

    let mut created_ids = Vec::new();

    for (name, power, grade) in test_cases {
        let request_id = format!(\"req_{}\", request_counter);
        request_counter += 1;

        let response = service.create_sorcerer(
            request_id,
            client_id.clone(),
            name.to_string(),
            power,
            grade.to_string()
        );

        if response.success {
            if let Some(sorcerer) = response.data {
                println!(\"✅ 作成成功: {} (ID: {})\", sorcerer.name, sorcerer.id);
                created_ids.push(sorcerer.id);
            }
        } else {
            println!(\"❌ 作成失敗: {}\", response.error.unwrap_or(\"Unknown error\".to_string()));
        }
    }

    // 呪術師取得テスト
    println!(\"\\n2. 呪術師取得テスト\");

    for id in &created_ids {
        let request_id = format!(\"req_{}\", request_counter);
        request_counter += 1;

        let response = service.get_sorcerer(request_id, client_id.clone(), id.clone());

        if response.success {
            if let Some(sorcerer) = response.data {
                println!(\"✅ 取得成功: {} (呪力: {})\", sorcerer.name, sorcerer.power);
            }
        } else {
            println!(\"❌ 取得失敗: {}\", response.error.unwrap_or(\"Unknown error\".to_string()));
        }
    }

    // 術式追加テスト
    println!(\"\\n3. 術式追加テスト\");

    if !created_ids.is_empty() {
        let techniques = vec![\"無下限呪術\", \"術式順転『蒼』\", \"黒閃\"];

        for (i, technique) in techniques.iter().enumerate() {
            if i < created_ids.len() {
                let request_id = format!(\"req_{}\", request_counter);
                request_counter += 1;

                let response = service.add_technique(
                    request_id,
                    client_id.clone(),
                    created_ids[i].clone(),
                    technique.to_string()
                );

                if response.success {
                    println!(\"✅ 術式追加成功: {}\", technique);
                } else {
                    println!(\"❌ 術式追加失敗: {}\", response.error.unwrap_or(\"Unknown error\".to_string()));
                }
            }
        }
    }

    // 一覧取得テスト
    println!(\"\\n4. 一覧取得テスト\");

    let request_id = format!(\"req_{}\", request_counter);
    let response = service.list_sorcerers(request_id, client_id.clone());

    if response.success {
        if let Some(sorcerers) = response.data {
            println!(\"✅ 一覧取得成功: {}件\", sorcerers.len());
            for sorcerer in sorcerers {
                println!(\"  - {} (ID: {}, 呪力: {}, 術式数: {})\",
                         sorcerer.name, sorcerer.id, sorcerer.power, sorcerer.techniques.len());
            }
        }
    } else {
        println!(\"❌ 一覧取得失敗: {}\", response.error.unwrap_or(\"Unknown error\".to_string()));
    }

    // レート制限テスト
    println!(\"\\n5. レート制限テスト\");

    for i in 1..=5 {
        let request_id = format!(\"rate_test_{}\", i);
        let response = service.list_sorcerers(request_id, \"rate_test_client\".to_string());

        if response.success {
            println!(\"✅ リクエスト{}: 成功\", i);
        } else {
            println!(\"❌ リクエスト{}: {}\", i, response.error.unwrap_or(\"Unknown error\".to_string()));
        }
    }

    println!(\"\\n=== マイクロサービステスト完了 ===\");
}

// 注意: このコードを実行するには Cargo.toml に以下を追加してください:
// [dependencies]
// chrono = { version = \"0.4\", features = [\"serde\"] }
```

</div>
</details>

## 章末総括

第3章の練習問題、素晴らしい戦いだった！反転術式編の全ての技術を実践で確認できたな。

これらの問題を通して学んだこと：

- **Option型** - 安全な不在の表現
- **Result型** - 型安全なエラーハンドリング
- **カスタムエラー** - ドメイン固有の表現力
- **エラーの合成** - 複雑なシステムでの統合
- **実用的パターン** - 実際のプロジェクトでの応用

!!! tip "五条先生の最終アドバイス"
    エラーハンドリングは最初は面倒に感じるかもしれない。でも一度マスターすれば、プログラムがクラッシュすることはほぼなくなる。

    俺の反転術式と同じで、一見ネガティブな「エラー」という概念を、型安全で表現力豊かな力に変換できるようになった。

次は第4章「領域展開編」でジェネリクスとトレイトを学ぼう。より抽象的で強力な概念の習得だ。

______________________________________________________________________

*「エラーハンドリングを極めれば、失敗すら成功の一部となる」*
