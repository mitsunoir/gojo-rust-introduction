# Result型 - エラーを支配する力

## Result型とは - 失敗を成功に変える反転術式

Option型が「値があるかもしれない」を表現するなら、Result型は「成功するかもしれないし、失敗するかもしれない」操作を表現する。これこそ俺の反転術式の真髄だ。

従来の言語では例外処理でエラーを扱っていたが、これは予測不可能で制御しづらい。Rustは違う。**Result型**でエラーも値として扱い、完璧に制御する。

!!! note "五条先生の解説"
    Result型は成功時の値とエラー時の情報を型安全に表現する。
    `Ok(T)`で成功、`Err(E)`で失敗を表す。例外は発生せず、すべてのエラーが明示的に処理される。

## Result型の基本

### Result型の定義

```rust
// Rustの標準ライブラリで定義されている
enum Result<T, E> {
    Ok(T),   // 成功時の値
    Err(E),  // エラー時の情報
}
```

### 基本的な使用法

```rust
fn main() {
    // 成功の場合
    let success: Result<i32, &str> = Ok(1500);

    // 失敗の場合
    let failure: Result<i32, &str> = Err(\"呪力不足\");

    println!(\"成功: {:?}\", success);  // Ok(1500)
    println!(\"失敗: {:?}\", failure);  // Err(\"呪力不足\")
}
```

## Result型を返す関数

### 基本的なエラーハンドリング

```rust
fn cast_technique(power: i32, required_power: i32) -> Result<String, String> {
    if power >= required_power {
        Ok(format!(\"術式発動成功！ (使用呪力: {})\", required_power))
    } else {
        Err(format!(\"呪力不足。必要: {}, 現在: {}\", required_power, power))
    }
}

fn divide_power(power: i32, divisor: i32) -> Result<i32, String> {
    if divisor == 0 {
        Err(\"0で割ることはできません\".to_string())
    } else {
        Ok(power / divisor)
    }
}

fn main() {
    // 成功例
    let success = cast_technique(2000, 1500);
    println!(\"{:?}\", success);

    // 失敗例
    let failure = cast_technique(800, 1500);
    println!(\"{:?}\", failure);

    // 除算の例
    println!(\"{:?}\", divide_power(1000, 5));  // Ok(200)
    println!(\"{:?}\", divide_power(1000, 0));  // Err
}
```

### より実用的な例

```rust
#[derive(Debug)]
enum TechniqueError {
    InsufficientPower,
    TechniqueNotFound,
    InvalidTarget,
    CooldownActive,
}

impl std::fmt::Display for TechniqueError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            TechniqueError::InsufficientPower => write!(f, \"呪力が不足しています\"),
            TechniqueError::TechniqueNotFound => write!(f, \"術式が見つかりません\"),
            TechniqueError::InvalidTarget => write!(f, \"無効な対象です\"),
            TechniqueError::CooldownActive => write!(f, \"術式がクールダウン中です\"),
        }
    }
}

struct Sorcerer {
    name: String,
    power: i32,
    techniques: Vec<String>,
    cooldowns: Vec<bool>,
}

impl Sorcerer {
    fn new(name: &str, power: i32) -> Self {
        Sorcerer {
            name: String::from(name),
            power,
            techniques: Vec::new(),
            cooldowns: Vec::new(),
        }
    }

    fn add_technique(&mut self, technique: String) {
        self.techniques.push(technique);
        self.cooldowns.push(false);
    }

    fn use_technique(&mut self, technique_name: &str, required_power: i32)
        -> Result<String, TechniqueError> {

        // 呪力チェック
        if self.power < required_power {
            return Err(TechniqueError::InsufficientPower);
        }

        // 術式の存在チェック
        let technique_index = self.techniques.iter()
            .position(|tech| tech == technique_name)
            .ok_or(TechniqueError::TechniqueNotFound)?;

        // クールダウンチェック
        if self.cooldowns[technique_index] {
            return Err(TechniqueError::CooldownActive);
        }

        // 術式使用
        self.power -= required_power;
        self.cooldowns[technique_index] = true;

        Ok(format!(\"{} が {} を使用しました！ (残り呪力: {})\",
                   self.name, technique_name, self.power))
    }

    fn reset_cooldowns(&mut self) {
        for cooldown in &mut self.cooldowns {
            *cooldown = false;
        }
    }
}

fn main() {
    let mut gojo = Sorcerer::new(\"五条悟\", 3000);
    gojo.add_technique(\"術式順転『蒼』\".to_string());
    gojo.add_technique(\"術式反転『赫』\".to_string());

    // 成功例
    match gojo.use_technique(\"術式順転『蒼』\", 500) {
        Ok(message) => println!(\"✓ {}\", message),
        Err(error) => println!(\"✗ エラー: {}\", error),
    }

    // クールダウンエラー
    match gojo.use_technique(\"術式順転『蒼』\", 500) {
        Ok(message) => println!(\"✓ {}\", message),
        Err(error) => println!(\"✗ エラー: {}\", error),
    }

    // 存在しない術式エラー
    match gojo.use_technique(\"存在しない術式\", 100) {
        Ok(message) => println!(\"✓ {}\", message),
        Err(error) => println!(\"✗ エラー: {}\", error),
    }

    // クールダウンリセット後に再実行
    gojo.reset_cooldowns();
    match gojo.use_technique(\"術式順転『蒼』\", 500) {
        Ok(message) => println!(\"✓ {}\", message),
        Err(error) => println!(\"✗ エラー: {}\", error),
    }
}
```

## Resultの処理方法

### match式による処理

```rust
fn process_technique_result(result: Result<String, TechniqueError>) {
    match result {
        Ok(message) => {
            println!(\"成功: {}\", message);
        },
        Err(TechniqueError::InsufficientPower) => {
            println!(\"呪力が足りません。修行しましょう。\");
        },
        Err(TechniqueError::TechniqueNotFound) => {
            println!(\"その術式は習得していません。\");
        },
        Err(error) => {
            println!(\"予期しないエラー: {}\", error);
        },
    }
}
```

### if let を使った処理

```rust
fn main() {
    let result = cast_technique(2000, 1500);

    // 成功時のみ処理
    if let Ok(message) = result {
        println!(\"術式成功: {}\", message);
    }

    let error_result = cast_technique(500, 1500);

    // エラー時のみ処理
    if let Err(error) = error_result {
        println!(\"術式失敗: {}\", error);
    }
}
```

## Resultのメソッド

### unwrap系メソッド

```rust
fn main() {
    let success: Result<i32, &str> = Ok(1500);
    let failure: Result<i32, &str> = Err(\"エラー\");

    // unwrap() - Okの値を取り出すが、Errの場合はパニック
    let power = success.unwrap();
    println!(\"呪力: {}\", power);

    // failure.unwrap(); // パニックする！

    // unwrap_or() - Errの場合はデフォルト値
    let safe_power = failure.unwrap_or(0);
    println!(\"安全な呪力: {}\", safe_power);

    // unwrap_or_else() - Errの場合はクロージャを実行
    let calculated_power = failure.unwrap_or_else(|error| {
        println!(\"エラーが発生: {}\", error);
        100  // デフォルト値
    });
    println!(\"計算された呪力: {}\", calculated_power);
}
```

### expect() メソッド

```rust
fn main() {
    let technique_result: Result<String, &str> = Ok(\"術式順転『蒼』\".to_string());

    // expect() - unwrap()と同じだが、エラーメッセージを指定
    let technique = technique_result.expect(\"術式の取得に失敗しました\");
    println!(\"技: {}\", technique);
}
```

### is_ok() と is_err()

```rust
fn check_technique_status(result: Result<String, TechniqueError>) {
    if result.is_ok() {
        println!(\"術式が正常に実行されました\");
    } else {
        println!(\"術式の実行に失敗しました\");
    }

    // エラーの詳細確認
    if result.is_err() {
        if let Err(error) = result {
            println!(\"エラーの詳細: {}\", error);
        }
    }
}
```

### map() と map_err()

```rust
fn main() {
    let power_result: Result<i32, &str> = Ok(1500);
    let error_result: Result<i32, &str> = Err(\"計算エラー\");

    // map() - Okの値を変換
    let doubled_power = power_result.map(|power| power * 2);
    println!(\"2倍の呪力: {:?}\", doubled_power);  // Ok(3000)

    // map_err() - Errの値を変換
    let formatted_error = error_result.map_err(|err| format!(\"発生したエラー: {}\", err));
    println!(\"フォーマット済みエラー: {:?}\", formatted_error);

    // 複雑な変換
    let power_description = Ok(2500)
        .map(|power| {
            if power >= 3000 {
                \"最強級\"
            } else if power >= 2000 {
                \"特級\"
            } else {
                \"一般\"
            }
        });

    println!(\"呪力レベル: {:?}\", power_description);
}
```

### and_then() メソッド

```rust
fn get_sorcerer_power(name: &str) -> Result<i32, String> {
    match name {
        \"五条悟\" => Ok(3000),
        \"虎杖悠仁\" => Ok(1200),
        \"伏黒恵\" => Ok(1000),
        _ => Err(format!(\"{}は見つかりません\", name)),
    }
}

fn calculate_technique_damage(power: i32) -> Result<i32, String> {
    if power >= 1000 {
        Ok(power / 2)
    } else {
        Err(\"呪力が低すぎてダメージ計算ができません\".to_string())
    }
}

fn main() {
    let name = \"五条悟\";

    // and_then() でResultを返す関数をチェーン
    let damage = get_sorcerer_power(name)
        .and_then(calculate_technique_damage);

    println!(\"{} のダメージ: {:?}\", name, damage);

    // より複雑なチェーン
    let result = get_sorcerer_power(\"虎杖悠仁\")
        .and_then(calculate_technique_damage)
        .map(|damage| format!(\"最終ダメージ: {}\", damage));

    println!(\"計算結果: {:?}\", result);

    // 失敗例
    let failed_result = get_sorcerer_power(\"存在しない呪術師\")
        .and_then(calculate_technique_damage);

    println!(\"失敗例: {:?}\", failed_result);
}
```

## ?演算子 - エラーの優雅な伝播

```rust
fn complex_technique_calculation(sorcerer_name: &str) -> Result<String, String> {
    let power = get_sorcerer_power(sorcerer_name)?;  // エラーの場合、早期リターン
    let damage = calculate_technique_damage(power)?;  // エラーの場合、早期リターン

    Ok(format!(\"{} の術式ダメージ: {}\", sorcerer_name, damage))
}

fn multiple_operations() -> Result<i32, String> {
    let power1 = get_sorcerer_power(\"五条悟\")?;
    let power2 = get_sorcerer_power(\"虎杖悠仁\")?;
    let total_power = power1 + power2;

    if total_power > 5000 {
        Err(\"呪力が強すぎます\".to_string())
    } else {
        Ok(total_power)
    }
}

fn main() {
    // ?演算子を使った複雑な計算
    match complex_technique_calculation(\"五条悟\") {
        Ok(result) => println!(\"✓ {}\", result),
        Err(error) => println!(\"✗ {}\", error),
    }

    // 存在しない呪術師での失敗例
    match complex_technique_calculation(\"存在しない\") {
        Ok(result) => println!(\"✓ {}\", result),
        Err(error) => println!(\"✗ {}\", error),
    }

    // 複数操作の例
    match multiple_operations() {
        Ok(total) => println!(\"✓ 合計呪力: {}\", total),
        Err(error) => println!(\"✗ {}\", error),
    }
}
```

## 実践例 - 呪術学校成績管理システム

```rust
use std::collections::HashMap;

#[derive(Debug)]
enum GradeError {
    StudentNotFound,
    SubjectNotFound,
    InvalidGrade,
    DatabaseError(String),
}

impl std::fmt::Display for GradeError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            GradeError::StudentNotFound => write!(f, \"学生が見つかりません\"),
            GradeError::SubjectNotFound => write!(f, \"科目が見つかりません\"),
            GradeError::InvalidGrade => write!(f, \"無効な成績です\"),
            GradeError::DatabaseError(msg) => write!(f, \"データベースエラー: {}\", msg),
        }
    }
}

#[derive(Debug, Clone)]
struct Student {
    name: String,
    grades: HashMap<String, i32>,
}

impl Student {
    fn new(name: &str) -> Self {
        Student {
            name: String::from(name),
            grades: HashMap::new(),
        }
    }

    fn add_grade(&mut self, subject: String, grade: i32) -> Result<(), GradeError> {
        if grade < 0 || grade > 100 {
            return Err(GradeError::InvalidGrade);
        }

        self.grades.insert(subject, grade);
        Ok(())
    }

    fn get_grade(&self, subject: &str) -> Result<i32, GradeError> {
        self.grades.get(subject)
            .copied()
            .ok_or(GradeError::SubjectNotFound)
    }

    fn get_average(&self) -> Result<f64, GradeError> {
        if self.grades.is_empty() {
            return Err(GradeError::DatabaseError(\"成績データがありません\".to_string()));
        }

        let total: i32 = self.grades.values().sum();
        let average = total as f64 / self.grades.len() as f64;
        Ok(average)
    }

    fn get_grade_level(&self) -> Result<String, GradeError> {
        let average = self.get_average()?;

        let level = match average as i32 {
            90..=100 => \"優秀\",
            80..=89 => \"良好\",
            70..=79 => \"普通\",
            60..=69 => \"要努力\",
            _ => \"要指導\",
        };

        Ok(level.to_string())
    }
}

struct GradeManager {
    students: HashMap<String, Student>,
}

impl GradeManager {
    fn new() -> Self {
        GradeManager {
            students: HashMap::new(),
        }
    }

    fn add_student(&mut self, student: Student) {
        self.students.insert(student.name.clone(), student);
    }

    fn get_student(&self, name: &str) -> Result<&Student, GradeError> {
        self.students.get(name).ok_or(GradeError::StudentNotFound)
    }

    fn get_student_mut(&mut self, name: &str) -> Result<&mut Student, GradeError> {
        self.students.get_mut(name).ok_or(GradeError::StudentNotFound)
    }

    fn add_grade(&mut self, student_name: &str, subject: String, grade: i32)
        -> Result<(), GradeError> {
        let student = self.get_student_mut(student_name)?;
        student.add_grade(subject, grade)?;
        Ok(())
    }

    fn get_student_report(&self, name: &str) -> Result<String, GradeError> {
        let student = self.get_student(name)?;
        let average = student.get_average()?;
        let level = student.get_grade_level()?;

        let mut report = format!(\"学生: {}\\n平均点: {:.1}\\n評価: {}\\n\\n科目別成績:\\n\",
                                student.name, average, level);

        for (subject, grade) in &student.grades {
            report.push_str(&format!(\"  {}: {}点\\n\", subject, grade));
        }

        Ok(report)
    }

    fn compare_students(&self, name1: &str, name2: &str) -> Result<String, GradeError> {
        let student1 = self.get_student(name1)?;
        let student2 = self.get_student(name2)?;

        let avg1 = student1.get_average()?;
        let avg2 = student2.get_average()?;

        let comparison = if avg1 > avg2 {
            format!(\"{} の方が優秀 (平均点差: {:.1})\",
                    student1.name, avg1 - avg2)
        } else if avg2 > avg1 {
            format!(\"{} の方が優秀 (平均点差: {:.1})\",
                    student2.name, avg2 - avg1)
        } else {
            \"同じ成績\".to_string()
        };

        Ok(comparison)
    }

    fn get_class_statistics(&self) -> Result<String, GradeError> {
        if self.students.is_empty() {
            return Err(GradeError::DatabaseError(\"学生データがありません\".to_string()));
        }

        let mut total_average = 0.0;
        let mut student_count = 0;
        let mut grade_levels = HashMap::new();

        for student in self.students.values() {
            let average = student.get_average()?;
            let level = student.get_grade_level()?;

            total_average += average;
            student_count += 1;
            *grade_levels.entry(level).or_insert(0) += 1;
        }

        let class_average = total_average / student_count as f64;

        let mut stats = format!(\"クラス統計\\n学生数: {}\\nクラス平均: {:.1}\\n\\n評価分布:\\n\",
                               student_count, class_average);

        for (level, count) in grade_levels {
            stats.push_str(&format!(\"  {}: {}人\\n\", level, count));
        }

        Ok(stats)
    }
}

fn main() {
    let mut manager = GradeManager::new();

    // 学生データの作成
    let mut gojo = Student::new(\"五条悟\");
    let mut yuji = Student::new(\"虎杖悠仁\");
    let mut megumi = Student::new(\"伏黒恵\");

    // 成績の追加（エラーハンドリング付き）
    let subjects = vec![\"呪術理論\", \"実技\", \"呪具学\", \"結界術\"];
    let gojo_grades = vec![98, 95, 92, 96];
    let yuji_grades = vec![75, 88, 70, 73];
    let megumi_grades = vec![89, 85, 91, 87];

    // 五条悟の成績
    for (subject, grade) in subjects.iter().zip(gojo_grades.iter()) {
        if let Err(e) = gojo.add_grade(subject.to_string(), *grade) {
            println!(\"エラー: {}\", e);
        }
    }

    // 虎杖悠仁の成績
    for (subject, grade) in subjects.iter().zip(yuji_grades.iter()) {
        if let Err(e) = yuji.add_grade(subject.to_string(), *grade) {
            println!(\"エラー: {}\", e);
        }
    }

    // 伏黒恵の成績
    for (subject, grade) in subjects.iter().zip(megumi_grades.iter()) {
        if let Err(e) = megumi.add_grade(subject.to_string(), *grade) {
            println!(\"エラー: {}\", e);
        }
    }

    manager.add_student(gojo);
    manager.add_student(yuji);
    manager.add_student(megumi);

    println!(\"=== 呪術学校成績管理システム ===\");

    // 学生レポート
    match manager.get_student_report(\"五条悟\") {
        Ok(report) => println!(\"\\n{}\", report),
        Err(error) => println!(\"エラー: {}\", error),
    }

    // 学生比較
    match manager.compare_students(\"五条悟\", \"虎杖悠仁\") {
        Ok(comparison) => println!(\"比較結果: {}\\n\", comparison),
        Err(error) => println!(\"比較エラー: {}\", error),
    }

    // クラス統計
    match manager.get_class_statistics() {
        Ok(stats) => println!(\"{}\", stats),
        Err(error) => println!(\"統計エラー: {}\", error),
    }

    // エラーケースのテスト
    match manager.get_student_report(\"存在しない学生\") {
        Ok(report) => println!(\"{}\", report),
        Err(error) => println!(\"予期されたエラー: {}\", error),
    }

    // 無効な成績の追加テスト
    match manager.add_grade(\"五条悟\", \"新科目\".to_string(), 150) {
        Ok(_) => println!(\"成績追加成功\"),
        Err(error) => println!(\"予期されたエラー: {}\", error),
    }
}
```

## まとめ

Result型の力はマスターできたか？重要なポイント：

1. **型安全なエラーハンドリング** - 例外ではなく値として扱う
1. **明示的な処理** - すべてのエラーが型で表現される
1. **?演算子** - エラーの優雅な伝播
1. **関数型メソッド** - map、and_then、map_errなど
1. **合成可能性** - 複数の操作を安全にチェーン

これは俺の反転術式の極意だ。エラーという一見ネガティブな状況を、型安全という絶対的な力に変換する。

次はより高度なエラーハンドリングのパターンを学ぼう。複数のエラー型を統合し、より大規模なシステムでのエラー管理を習得する。

______________________________________________________________________

*「Resultを極めれば、失敗すら成功への道標となる」*
