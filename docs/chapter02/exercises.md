# 第2章 練習問題 - 呪力操作の試練

## 所有権・借用・ライフタイムの総合演習

第2章で学んだ呪力操作編の内容を実践で確認しよう。所有権、借用、ライフタイム - これらは俺の無下限術式と同じく、一度マスターすれば絶対的な力になる。

!!! tip "五条先生からのアドバイス"
    この章の問題は少し手強いぞ。でも心配するな。基本を理解していれば必ず解ける。
    コンパイラのエラーメッセージをよく読んで、何が問題なのかを理解することも大切だ。

## 初級編 - 所有権の基本操作

### 問題1: 呪力の移譲システム

2人の呪術師間で呪力を移譲するシステムを作成せよ。以下の仕様を満たすこと：

- 呪術師は名前と呪力値を持つ
- 呪力を他の呪術師に移譲する関数
- 移譲後は元の呪術師の呪力は0になる

<div class="exercise">

```rust
struct Sorcerer {
    name: String,
    power: i32,
}

// ここに実装
fn main() {
    let mut gojo = Sorcerer {
        name: String::from("五条悟"),
        power: 2000,
    };

    let mut yuji = Sorcerer {
        name: String::from("虎杖悠仁"),
        power: 800,
    };

    // 呪力移譲の実装
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class="solution">

```rust
struct Sorcerer {
    name: String,
    power: i32,
}

impl Sorcerer {
    fn transfer_power(&mut self, amount: i32, recipient: &mut Sorcerer) -> bool {
        if self.power >= amount {
            self.power -= amount;
            recipient.power += amount;
            println!("{} が {} に {}の呪力を移譲",
                     self.name, recipient.name, amount);
            true
        } else {
            println!("{} の呪力が不足しています", self.name);
            false
        }
    }

    fn display_status(&self) {
        println!("{}: 呪力 {}", self.name, self.power);
    }
}

fn main() {
    let mut gojo = Sorcerer {
        name: String::from("五条悟"),
        power: 2000,
    };

    let mut yuji = Sorcerer {
        name: String::from("虎杖悠仁"),
        power: 800,
    };

    println!("=== 移譲前 ===");
    gojo.display_status();
    yuji.display_status();

    gojo.transfer_power(500, &mut yuji);

    println!("\\n=== 移譲後 ===");
    gojo.display_status();
    yuji.display_status();
}
```

</div>
</details>

### 問題2: 術式コレクションの管理

術式名のベクターを管理するシステムで、所有権の移動を正しく処理せよ。

<div class="exercise">

```rust
fn process_techniques(techniques: Vec<String>) -> Vec<String> {
    // 各術式に "習得済み:" を前置
    // 処理後のベクターを返す
}

fn count_powerful_techniques(techniques: &Vec<String>) -> usize {
    // "術式"を含む技の数を数える
}

fn main() {
    let techniques = vec![
        String::from("術式順転『蒼』"),
        String::from("基本攻撃"),
        String::from("術式反転『赫』"),
        String::from("防御技"),
    ];

    // ここに実装
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class="solution">

```rust
fn process_techniques(techniques: Vec<String>) -> Vec<String> {
    techniques.into_iter()
        .map(|tech| format!("習得済み: {}", tech))
        .collect()
}

fn count_powerful_techniques(techniques: &Vec<String>) -> usize {
    techniques.iter()
        .filter(|tech| tech.contains("術式"))
        .count()
}

fn main() {
    let techniques = vec![
        String::from("術式順転『蒼』"),
        String::from("基本攻撃"),
        String::from("術式反転『赫』"),
        String::from("防御技"),
    ];

    println!("元の技: {:?}", techniques);

    // 借用で技の数を数える
    let powerful_count = count_powerful_techniques(&techniques);
    println!("術式の数: {}", powerful_count);

    // 所有権を移動して処理
    let processed = process_techniques(techniques);
    println!("処理後: {:?}", processed);

    // techniques はもう使えない（所有権が移動した）
    // println!("{:?}", techniques); // エラー！
}
```

</div>
</details>

## 中級編 - 借用と参照の活用

### 問題3: 戦闘ログ分析システム

戦闘ログを分析して統計情報を提供するシステムを作成せよ。借用を適切に使用すること。

<div class="exercise">

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

    fn add_entry(&mut self, entry: String) {
        self.entries.push(entry);
    }
}

// 以下の関数を実装せよ：
// 1. ログの総数を返す関数（不変借用）
// 2. 特定のキーワードを含むエントリ数を返す関数（不変借用）
// 3. 最後のN件のエントリを返す関数（スライス）
// 4. すべてのエントリに接頭辞を追加する関数（可変借用）

fn main() {
    let mut log = BattleLog::new();
    log.add_entry(String::from("五条悟が術式順転『蒼』を使用"));
    log.add_entry(String::from("宿儺が解を発動"));
    log.add_entry(String::from("五条悟が術式反転『赫』を使用"));

    // テスト用コード
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class="solution">

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

    fn add_entry(&mut self, entry: String) {
        self.entries.push(entry);
    }
}

// 1. ログの総数を返す（不変借用）
fn count_entries(log: &BattleLog) -> usize {
    log.entries.len()
}

// 2. キーワードを含むエントリ数（不変借用）
fn count_entries_with_keyword(log: &BattleLog, keyword: &str) -> usize {
    log.entries.iter()
        .filter(|entry| entry.contains(keyword))
        .count()
}

// 3. 最後のN件のエントリ（スライス）
fn get_recent_entries(log: &BattleLog, count: usize) -> &[String] {
    let start = log.entries.len().saturating_sub(count);
    &log.entries[start..]
}

// 4. すべてのエントリに接頭辞を追加（可変借用）
fn add_prefix_to_all(log: &mut BattleLog, prefix: &str) {
    for entry in &mut log.entries {
        *entry = format!("{}: {}", prefix, entry);
    }
}

// 5. 特定の条件のエントリを取得（不変借用）
fn find_entries_by_user<'a>(log: &'a BattleLog, user: &str) -> Vec<&'a String> {
    log.entries.iter()
        .filter(|entry| entry.contains(user))
        .collect()
}

fn main() {
    let mut log = BattleLog::new();
    log.add_entry(String::from("五条悟が術式順転『蒼』を使用"));
    log.add_entry(String::from("宿儺が解を発動"));
    log.add_entry(String::from("五条悟が術式反転『赫』を使用"));
    log.add_entry(String::from("虎杖悠仁が黒閃を発動"));

    println!("=== 戦闘ログ分析 ===");

    // 統計情報
    println!("総エントリ数: {}", count_entries(&log));
    println!("五条悟の行動: {}回", count_entries_with_keyword(&log, "五条悟"));
    println!("術式使用: {}回", count_entries_with_keyword(&log, "術式"));

    // 最近のエントリ
    let recent = get_recent_entries(&log, 2);
    println!("\\n最近の2件:");
    for (i, entry) in recent.iter().enumerate() {
        println!("  {}. {}", i + 1, entry);
    }

    // 五条悟の行動のみ
    let gojo_entries = find_entries_by_user(&log, "五条悟");
    println!("\\n五条悟の行動:");
    for entry in gojo_entries {
        println!("  - {}", entry);
    }

    // 接頭辞追加
    add_prefix_to_all(&mut log, "[戦闘記録]");
    println!("\\n接頭辞追加後の最初のエントリ:");
    if let Some(first) = log.entries.first() {
        println!("  {}", first);
    }
}
```

</div>
</details>

### 問題4: 文字列スライス操作

文字列スライスを使って術式名を解析する関数群を作成せよ。

<div class="exercise">

```rust
// 以下の関数を実装せよ：
// 1. 術式名から種類を抽出（"術式順転『蒼』" → "順転"）
// 2. 術式名から技名を抽出（"術式順転『蒼』" → "蒼"）
// 3. 複数の術式名の中から最も長いものを返す
// 4. 術式名が有効かチェック（「術式」と「『』」を含む）

fn main() {
    let techniques = [
        "術式順転『蒼』",
        "術式反転『赫』",
        "虚式『茈』",
        "無効な技名",
        "基本攻撃",
    ];

    // テスト用コード
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class="solution">

```rust
// 1. 術式の種類を抽出
fn extract_technique_type(technique: &str) -> Option<&str> {
    if let Some(start) = technique.find("術式") {
        let after_jutsu = &technique[start + 6..]; // "術式"の後
        if let Some(end) = after_jutsu.find("『") {
            return Some(&after_jutsu[..end]);
        }
    }
    None
}

// 2. 技名を抽出
fn extract_technique_name(technique: &str) -> Option<&str> {
    if let Some(start) = technique.find("『") {
        if let Some(end) = technique.find("』") {
            if start < end {
                return Some(&technique[start + 3..end]); // 『の後から』の前まで
            }
        }
    }
    None
}

// 3. 最も長い術式名を返す
fn find_longest_technique<'a>(techniques: &[&'a str]) -> Option<&'a str> {
    techniques.iter()
        .max_by_key(|tech| tech.len())
        .copied()
}

// 4. 術式名の有効性チェック
fn is_valid_technique(technique: &str) -> bool {
    technique.contains("術式") && technique.contains("『") && technique.contains("』")
}

// 5. 術式を詳細分析
fn analyze_technique(technique: &str) -> String {
    let mut analysis = String::new();

    analysis.push_str(&format!("技名: {}\\n", technique));
    analysis.push_str(&format!("文字数: {}\\n", technique.chars().count()));
    analysis.push_str(&format!("有効: {}\\n", is_valid_technique(technique)));

    if let Some(tech_type) = extract_technique_type(technique) {
        analysis.push_str(&format!("種類: {}\\n", tech_type));
    }

    if let Some(name) = extract_technique_name(technique) {
        analysis.push_str(&format!("技名: {}\\n", name));
    }

    analysis
}

// 6. 複数技の統計
fn generate_technique_stats(techniques: &[&str]) -> String {
    let valid_count = techniques.iter()
        .filter(|&&tech| is_valid_technique(tech))
        .count();

    let total_chars: usize = techniques.iter()
        .map(|tech| tech.chars().count())
        .sum();

    let avg_length = if techniques.is_empty() {
        0.0
    } else {
        total_chars as f64 / techniques.len() as f64
    };

    format!(
        "技数: {}\\n有効技数: {}\\n平均文字数: {:.1}\\n最長技: {}",
        techniques.len(),
        valid_count,
        avg_length,
        find_longest_technique(techniques).unwrap_or("なし")
    )
}

fn main() {
    let techniques = [
        "術式順転『蒼』",
        "術式反転『赫』",
        "虚式『茈』",
        "無効な技名",
        "基本攻撃",
        "無下限呪術『紫』",
    ];

    println!("=== 術式分析システム ===");

    // 各技の詳細分析
    for technique in &techniques {
        println!("\\n--- {} ---", technique);

        if let Some(tech_type) = extract_technique_type(technique) {
            println!("種類: {}", tech_type);
        }

        if let Some(name) = extract_technique_name(technique) {
            println!("技名: {}", name);
        }

        println!("有効: {}", is_valid_technique(technique));
    }

    // 統計情報
    println!("\\n=== 統計情報 ===");
    println!("{}", generate_technique_stats(&techniques));

    // 有効な技のみフィルタ
    let valid_techniques: Vec<&str> = techniques.iter()
        .filter(|&&tech| is_valid_technique(tech))
        .copied()
        .collect();

    println!("\\n=== 有効な術式一覧 ===");
    for (i, technique) in valid_techniques.iter().enumerate() {
        println!("{}. {}", i + 1, technique);
    }
}
```

</div>
</details>

## 上級編 - ライフタイムの実践

### 問題5: 呪術師データベース

ライフタイム注釈を使って、呪術師の情報を管理するデータベースシステムを作成せよ。

<div class="exercise">

```rust
// 呪術師の参照を保持する構造体
struct SorcererRef<'a> {
    name: &'a str,
    grade: &'a str,
    techniques: Vec<&'a str>,
}

// データベース構造体
struct SorcererDatabase<'a> {
    sorcerers: Vec<SorcererRef<'a>>,
}

// 以下のメソッドを実装せよ：
// 1. 新しい呪術師を追加
// 2. 名前で検索
// 3. 等級で検索
// 4. 特定の技を使える呪術師を検索
// 5. 最も技数の多い呪術師を返す

fn main() {
    // テスト用データ
    let names = ["五条悟", "両面宿儺", "虎杖悠仁"];
    let grades = ["特級", "特級", "1級"];
    let techniques = [
        vec!["無下限呪術", "術式順転『蒼』", "術式反転『赫』"],
        vec!["解", "捌", "伏魔御廚子"],
        vec!["黒閃", "発散"],
    ];

    // データベース操作のテスト
}
```

</div>

<details>
<summary>解答を見る</summary>
<div class="solution">

```rust
#[derive(Debug)]
struct SorcererRef<'a> {
    name: &'a str,
    grade: &'a str,
    techniques: Vec<&'a str>,
}

impl<'a> SorcererRef<'a> {
    fn new(name: &'a str, grade: &'a str) -> Self {
        SorcererRef {
            name,
            grade,
            techniques: Vec::new(),
        }
    }

    fn add_technique(&mut self, technique: &'a str) {
        self.techniques.push(technique);
    }

    fn has_technique(&self, technique: &str) -> bool {
        self.techniques.iter().any(|&t| t == technique)
    }

    fn technique_count(&self) -> usize {
        self.techniques.len()
    }
}

struct SorcererDatabase<'a> {
    sorcerers: Vec<SorcererRef<'a>>,
}

impl<'a> SorcererDatabase<'a> {
    fn new() -> Self {
        SorcererDatabase {
            sorcerers: Vec::new(),
        }
    }

    // 1. 新しい呪術師を追加
    fn add_sorcerer(&mut self, sorcerer: SorcererRef<'a>) {
        self.sorcerers.push(sorcerer);
    }

    // 2. 名前で検索
    fn find_by_name(&self, name: &str) -> Option<&SorcererRef<'a>> {
        self.sorcerers.iter()
            .find(|sorcerer| sorcerer.name == name)
    }

    // 3. 等級で検索
    fn find_by_grade(&self, grade: &str) -> Vec<&SorcererRef<'a>> {
        self.sorcerers.iter()
            .filter(|sorcerer| sorcerer.grade == grade)
            .collect()
    }

    // 4. 特定の技を使える呪術師を検索
    fn find_by_technique(&self, technique: &str) -> Vec<&SorcererRef<'a>> {
        self.sorcerers.iter()
            .filter(|sorcerer| sorcerer.has_technique(technique))
            .collect()
    }

    // 5. 最も技数の多い呪術師を返す
    fn find_most_skilled(&self) -> Option<&SorcererRef<'a>> {
        self.sorcerers.iter()
            .max_by_key(|sorcerer| sorcerer.technique_count())
    }

    // 6. 統計情報を生成
    fn generate_statistics(&self) -> DatabaseStats<'a> {
        DatabaseStats {
            database: self,
        }
    }
}

struct DatabaseStats<'a> {
    database: &'a SorcererDatabase<'a>,
}

impl<'a> DatabaseStats<'a> {
    fn total_sorcerers(&self) -> usize {
        self.database.sorcerers.len()
    }

    fn average_techniques(&self) -> f64 {
        if self.database.sorcerers.is_empty() {
            0.0
        } else {
            let total: usize = self.database.sorcerers.iter()
                .map(|s| s.technique_count())
                .sum();
            total as f64 / self.database.sorcerers.len() as f64
        }
    }

    fn grade_distribution(&self) -> std::collections::HashMap<&'a str, usize> {
        let mut distribution = std::collections::HashMap::new();
        for sorcerer in &self.database.sorcerers {
            *distribution.entry(sorcerer.grade).or_insert(0) += 1;
        }
        distribution
    }

    fn generate_report(&self) -> String {
        let mut report = String::new();

        report.push_str("=== 呪術師データベース統計 ===\\n");
        report.push_str(&format!("総呪術師数: {}\\n", self.total_sorcerers()));
        report.push_str(&format!("平均技数: {:.1}\\n", self.average_techniques()));

        if let Some(most_skilled) = self.database.find_most_skilled() {
            report.push_str(&format!("最多技保有者: {} ({}技)\\n",
                most_skilled.name, most_skilled.technique_count()));
        }

        report.push_str("\\n等級分布:\\n");
        for (grade, count) in self.grade_distribution() {
            report.push_str(&format!("  {}: {}人\\n", grade, count));
        }

        report
    }
}

fn main() {
    // テスト用データ
    let names = ["五条悟", "両面宿儺", "虎杖悠仁", "伏黒恵", "釘崎野薔薇"];
    let grades = ["特級", "特級", "1級", "2級", "3級"];

    let mut database = SorcererDatabase::new();

    // 呪術師データの追加
    let mut gojo = SorcererRef::new(names[0], grades[0]);
    gojo.add_technique("無下限呪術");
    gojo.add_technique("術式順転『蒼』");
    gojo.add_technique("術式反転『赫』");
    gojo.add_technique("虚式『茈』");
    database.add_sorcerer(gojo);

    let mut sukuna = SorcererRef::new(names[1], grades[1]);
    sukuna.add_technique("解");
    sukuna.add_technique("捌");
    sukuna.add_technique("伏魔御廚子");
    database.add_sorcerer(sukuna);

    let mut yuji = SorcererRef::new(names[2], grades[2]);
    yuji.add_technique("黒閃");
    yuji.add_technique("発散");
    database.add_sorcerer(yuji);

    let mut megumi = SorcererRef::new(names[3], grades[3]);
    megumi.add_technique("十種影法術");
    megumi.add_technique("玉犬");
    database.add_sorcerer(megumi);

    let mut nobara = SorcererRef::new(names[4], grades[4]);
    nobara.add_technique("芻霊呪法");
    database.add_sorcerer(nobara);

    // データベース操作のテスト
    println!("=== 呪術師データベーステスト ===");

    // 名前検索
    if let Some(gojo) = database.find_by_name("五条悟") {
        println!("\\n五条悟の情報:");
        println!("  等級: {}", gojo.grade);
        println!("  技数: {}", gojo.technique_count());
        println!("  技: {:?}", gojo.techniques);
    }

    // 等級検索
    let special_grade = database.find_by_grade("特級");
    println!("\\n特級呪術師:");
    for sorcerer in special_grade {
        println!("  {}", sorcerer.name);
    }

    // 技検索
    let black_flash_users = database.find_by_technique("黒閃");
    println!("\\n黒閃使い:");
    for sorcerer in black_flash_users {
        println!("  {}", sorcerer.name);
    }

    // 統計情報
    let stats = database.generate_statistics();
    println!("\\n{}", stats.generate_report());
}
```

</div>
</details>

### 問題6: 複雑なライフタイム関係

複数の構造体間で参照を持つ複雑なシステムを実装せよ。

<div class="exercise">

```rust
// 学校、クラス、生徒の関係を表現
// 学校は複数のクラスを持つ
// クラスは複数の生徒を持つ
// 生徒は所属クラスへの参照を持つ

struct School<'a> {
    name: &'a str,
    classes: Vec<Class<'a>>,
}

struct Class<'a> {
    name: &'a str,
    teacher: &'a str,
    students: Vec<Student<'a>>,
}

struct Student<'a> {
    name: &'a str,
    grade: i32,
    class: Option<&'a str>, // クラス名への参照
}

// 学校システムの管理機能を実装せよ
```

</div>

<details>
<summary>解答を見る</summary>
<div class="solution">

```rust
#[derive(Debug)]
struct School<'a> {
    name: &'a str,
    classes: Vec<Class<'a>>,
}

#[derive(Debug)]
struct Class<'a> {
    name: &'a str,
    teacher: &'a str,
    students: Vec<Student<'a>>,
}

#[derive(Debug)]
struct Student<'a> {
    name: &'a str,
    grade: i32,
    class: Option<&'a str>,
}

impl<'a> School<'a> {
    fn new(name: &'a str) -> Self {
        School {
            name,
            classes: Vec::new(),
        }
    }

    fn add_class(&mut self, class: Class<'a>) {
        self.classes.push(class);
    }

    fn find_class(&self, class_name: &str) -> Option<&Class<'a>> {
        self.classes.iter()
            .find(|class| class.name == class_name)
    }

    fn find_class_mut(&mut self, class_name: &str) -> Option<&mut Class<'a>> {
        self.classes.iter_mut()
            .find(|class| class.name == class_name)
    }

    fn find_student(&self, student_name: &str) -> Option<(&Class<'a>, &Student<'a>)> {
        for class in &self.classes {
            if let Some(student) = class.find_student(student_name) {
                return Some((class, student));
            }
        }
        None
    }

    fn get_all_students(&self) -> Vec<&Student<'a>> {
        self.classes.iter()
            .flat_map(|class| class.students.iter())
            .collect()
    }

    fn get_top_students(&self, min_grade: i32) -> Vec<&Student<'a>> {
        self.get_all_students().into_iter()
            .filter(|student| student.grade >= min_grade)
            .collect()
    }

    fn generate_school_report(&self) -> String {
        let mut report = String::new();

        report.push_str(&format!("=== {} 学校レポート ===\\n", self.name));
        report.push_str(&format!("クラス数: {}\\n", self.classes.len()));

        let total_students: usize = self.classes.iter()
            .map(|class| class.students.len())
            .sum();
        report.push_str(&format!("総生徒数: {}\\n", total_students));

        if total_students > 0 {
            let total_grade: i32 = self.get_all_students().iter()
                .map(|student| student.grade)
                .sum();
            let avg_grade = total_grade as f64 / total_students as f64;
            report.push_str(&format!("平均成績: {:.1}\\n", avg_grade));
        }

        report.push_str("\\nクラス別詳細:\\n");
        for class in &self.classes {
            report.push_str(&class.generate_class_summary());
        }

        report
    }
}

impl<'a> Class<'a> {
    fn new(name: &'a str, teacher: &'a str) -> Self {
        Class {
            name,
            teacher,
            students: Vec::new(),
        }
    }

    fn add_student(&mut self, mut student: Student<'a>) {
        student.class = Some(self.name);
        self.students.push(student);
    }

    fn find_student(&self, student_name: &str) -> Option<&Student<'a>> {
        self.students.iter()
            .find(|student| student.name == student_name)
    }

    fn get_average_grade(&self) -> f64 {
        if self.students.is_empty() {
            0.0
        } else {
            let total: i32 = self.students.iter()
                .map(|student| student.grade)
                .sum();
            total as f64 / self.students.len() as f64
        }
    }

    fn get_top_student(&self) -> Option<&Student<'a>> {
        self.students.iter()
            .max_by_key(|student| student.grade)
    }

    fn generate_class_summary(&self) -> String {
        let mut summary = String::new();

        summary.push_str(&format!("  {} (担任: {})\\n", self.name, self.teacher));
        summary.push_str(&format!("    生徒数: {}\\n", self.students.len()));
        summary.push_str(&format!("    平均成績: {:.1}\\n", self.get_average_grade()));

        if let Some(top_student) = self.get_top_student() {
            summary.push_str(&format!("    トップ: {} ({}点)\\n",
                top_student.name, top_student.grade));
        }

        summary
    }
}

impl<'a> Student<'a> {
    fn new(name: &'a str, grade: i32) -> Self {
        Student {
            name,
            grade,
            class: None,
        }
    }

    fn get_class_name(&self) -> &str {
        self.class.unwrap_or("未所属")
    }

    fn get_grade_level(&self) -> &str {
        match self.grade {
            90..=100 => "優秀",
            80..=89 => "良好",
            70..=79 => "普通",
            60..=69 => "要努力",
            _ => "要指導",
        }
    }
}

fn main() {
    // 学校データの作成
    let mut jujutsu_school = School::new("東京呪術高等専門学校");

    // 1年生クラス
    let mut first_year = Class::new("1年A組", "五条悟");
    first_year.add_student(Student::new("虎杖悠仁", 85));
    first_year.add_student(Student::new("伏黒恵", 92));
    first_year.add_student(Student::new("釘崎野薔薇", 88));

    // 2年生クラス
    let mut second_year = Class::new("2年A組", "夜蛾正道");
    second_year.add_student(Student::new("禪院真希", 94));
    second_year.add_student(Student::new("狗巻棘", 90));
    second_year.add_student(Student::new("パンダ", 78));

    jujutsu_school.add_class(first_year);
    jujutsu_school.add_class(second_year);

    // 学校システムのテスト
    println!("=== 呪術高専管理システム ===");

    // 学校レポート
    println!("{}", jujutsu_school.generate_school_report());

    // 生徒検索
    if let Some((class, student)) = jujutsu_school.find_student("虎杖悠仁") {
        println!("\\n生徒情報:");
        println!("  名前: {}", student.name);
        println!("  成績: {} ({})", student.grade, student.get_grade_level());
        println!("  クラス: {}", class.name);
        println!("  担任: {}", class.teacher);
    }

    // 優秀な生徒一覧
    let top_students = jujutsu_school.get_top_students(90);
    println!("\\n優秀な生徒 (90点以上):");
    for student in top_students {
        println!("  {} - {}点 ({})",
            student.name, student.grade, student.get_class_name());
    }

    // クラス別詳細
    println!("\\n=== クラス別詳細 ===");
    for class in &jujutsu_school.classes {
        println!("{}:", class.name);
        for student in &class.students {
            println!("  {} - {}点 ({})",
                student.name, student.grade, student.get_grade_level());
        }
        println!();
    }
}
```

</div>
</details>

## 総合問題

### 問題7: 呪術戦闘管理システム

これまで学んだ所有権、借用、ライフタイムのすべてを使って、複雑な戦闘管理システムを実装せよ。

**要件:**

- 複数の戦闘同時管理
- 戦闘参加者の管理
- 技の使用履歴
- リアルタイム統計
- メモリ効率的な設計

<div class="exercise">

自由に設計して実装してみよう！所有権、借用、ライフタイムを適切に使い分けること。

</div>

<details>
<summary>解答例を見る</summary>
<div class="solution">

```rust
use std::collections::HashMap;

// 戦闘参加者
#[derive(Debug, Clone)]
struct Combatant {
    name: String,
    hp: i32,
    max_hp: i32,
    power: i32,
    techniques: Vec<String>,
}

impl Combatant {
    fn new(name: &str, hp: i32, power: i32) -> Self {
        Combatant {
            name: String::from(name),
            hp,
            max_hp: hp,
            power,
            techniques: Vec::new(),
        }
    }

    fn add_technique(&mut self, technique: String) {
        self.techniques.push(technique);
    }

    fn is_alive(&self) -> bool {
        self.hp > 0
    }

    fn take_damage(&mut self, damage: i32) {
        self.hp = (self.hp - damage).max(0);
    }

    fn get_hp_percentage(&self) -> f64 {
        (self.hp as f64 / self.max_hp as f64) * 100.0
    }
}

// 戦闘ログエントリ
#[derive(Debug, Clone)]
struct LogEntry {
    round: u32,
    action: String,
    timestamp: u64,
}

// 個別戦闘
struct Battle<'a> {
    id: u32,
    name: String,
    combatants: Vec<&'a mut Combatant>,
    log: Vec<LogEntry>,
    current_round: u32,
    is_active: bool,
}

impl<'a> Battle<'a> {
    fn new(id: u32, name: String) -> Self {
        Battle {
            id,
            name,
            combatants: Vec::new(),
            log: Vec::new(),
            current_round: 1,
            is_active: false,
        }
    }

    fn add_combatant(&mut self, combatant: &'a mut Combatant) {
        self.combatants.push(combatant);
    }

    fn start_battle(&mut self) {
        self.is_active = true;
        self.log_action(format!("戦闘開始: {}", self.name));
    }

    fn log_action(&mut self, action: String) {
        let entry = LogEntry {
            round: self.current_round,
            action,
            timestamp: self.log.len() as u64,
        };
        self.log.push(entry);
    }

    fn execute_round(&mut self) -> bool {
        if !self.is_active || self.combatants.len() < 2 {
            return false;
        }

        self.log_action(format!("--- ラウンド {} ---", self.current_round));

        // 生存者のみで戦闘
        let alive_combatants: Vec<_> = self.combatants.iter()
            .filter(|c| c.is_alive())
            .collect();

        if alive_combatants.len() < 2 {
            self.end_battle();
            return false;
        }

        // 簡易的な戦闘ロジック
        for i in 0..alive_combatants.len() {
            if !alive_combatants[i].is_alive() {
                continue;
            }

            // 攻撃対象を選択（次の生存者）
            let target_idx = (i + 1) % alive_combatants.len();
            if target_idx != i && alive_combatants[target_idx].is_alive() {
                let attacker_name = &alive_combatants[i].name;
                let target_name = &alive_combatants[target_idx].name;
                let damage = alive_combatants[i].power / 10;

                // 実際のダメージ適用は元の参照を通して
                for combatant in &mut self.combatants {
                    if combatant.name == *target_name {
                        combatant.take_damage(damage);
                        break;
                    }
                }

                self.log_action(format!("{} が {} を攻撃！ {}ダメージ",
                    attacker_name, target_name, damage));
            }
        }

        self.current_round += 1;
        true
    }

    fn end_battle(&mut self) {
        self.is_active = false;

        let winner = self.combatants.iter()
            .find(|c| c.is_alive())
            .map(|c| c.name.clone());

        if let Some(winner_name) = winner {
            self.log_action(format!("{} の勝利！", winner_name));
        } else {
            self.log_action("引き分け".to_string());
        }
    }

    fn get_battle_status(&self) -> BattleStatus {
        let alive_count = self.combatants.iter()
            .filter(|c| c.is_alive())
            .count();

        BattleStatus {
            battle_id: self.id,
            name: &self.name,
            is_active: self.is_active,
            current_round: self.current_round,
            alive_combatants: alive_count,
            total_combatants: self.combatants.len(),
            log_entries: self.log.len(),
        }
    }
}

// 戦闘状態の読み取り専用ビュー
#[derive(Debug)]
struct BattleStatus<'a> {
    battle_id: u32,
    name: &'a str,
    is_active: bool,
    current_round: u32,
    alive_combatants: usize,
    total_combatants: usize,
    log_entries: usize,
}

// 戦闘管理システム
struct BattleManager {
    combatants: HashMap<String, Combatant>,
    next_battle_id: u32,
}

impl BattleManager {
    fn new() -> Self {
        BattleManager {
            combatants: HashMap::new(),
            next_battle_id: 1,
        }
    }

    fn register_combatant(&mut self, combatant: Combatant) {
        self.combatants.insert(combatant.name.clone(), combatant);
    }

    fn create_battle(&mut self, name: String, participant_names: Vec<&str>) -> Option<Battle> {
        if participant_names.len() < 2 {
            return None;
        }

        let mut battle = Battle::new(self.next_battle_id, name);
        self.next_battle_id += 1;

        // 参加者の追加（借用で）
        for name in participant_names {
            if let Some(combatant) = self.combatants.get_mut(name) {
                // この部分は実際のライフタイム制約により複雑になる
                // 実用的には別のアプローチが必要
            }
        }

        Some(battle)
    }

    fn get_combatant(&self, name: &str) -> Option<&Combatant> {
        self.combatants.get(name)
    }

    fn get_combatant_mut(&mut self, name: &str) -> Option<&mut Combatant> {
        self.combatants.get_mut(name)
    }

    fn generate_overall_stats(&self) -> String {
        let mut stats = String::new();

        stats.push_str("=== 全体統計 ===\\n");
        stats.push_str(&format!("登録戦闘者数: {}\\n", self.combatants.len()));

        let alive_count = self.combatants.values()
            .filter(|c| c.is_alive())
            .count();
        stats.push_str(&format!("生存者数: {}\\n", alive_count));

        if let Some(strongest) = self.combatants.values()
            .filter(|c| c.is_alive())
            .max_by_key(|c| c.power) {
            stats.push_str(&format!("最強戦闘者: {} (呪力: {})\\n",
                strongest.name, strongest.power));
        }

        stats.push_str("\\n=== 戦闘者一覧 ===\\n");
        for combatant in self.combatants.values() {
            let status = if combatant.is_alive() { "生存" } else { "戦闘不能" };
            stats.push_str(&format!("{}: HP {}/{} ({}%) - {}\\n",
                combatant.name,
                combatant.hp,
                combatant.max_hp,
                combatant.get_hp_percentage(),
                status
            ));
        }

        stats
    }
}

fn main() {
    println!("=== 呪術戦闘管理システム ===");

    let mut manager = BattleManager::new();

    // 戦闘者の登録
    let mut gojo = Combatant::new("五条悟", 2000, 1800);
    gojo.add_technique("無下限呪術".to_string());
    gojo.add_technique("術式順転『蒼』".to_string());

    let mut sukuna = Combatant::new("両面宿儺", 1800, 1700);
    sukuna.add_technique("解".to_string());
    sukuna.add_technique("捌".to_string());

    let mut yuji = Combatant::new("虎杖悠仁", 1200, 900);
    yuji.add_technique("黒閃".to_string());

    let mut megumi = Combatant::new("伏黒恵", 1000, 800);
    megumi.add_technique("十種影法術".to_string());

    manager.register_combatant(gojo);
    manager.register_combatant(sukuna);
    manager.register_combatant(yuji);
    manager.register_combatant(megumi);

    // 初期状態表示
    println!("{}", manager.generate_overall_stats());

    // 模擬戦闘（簡易版）
    println!("\\n=== 模擬戦闘実行 ===");

    // 五条 vs 宿儺
    if let Some(gojo) = manager.get_combatant_mut("五条悟") {
        gojo.take_damage(300);
    }

    if let Some(sukuna) = manager.get_combatant_mut("両面宿儺") {
        sukuna.take_damage(400);
    }

    println!("戦闘後の状態:");
    if let Some(gojo) = manager.get_combatant("五条悟") {
        println!("五条悟: HP {}/2000 ({:.1}%)",
            gojo.hp, gojo.get_hp_percentage());
    }

    if let Some(sukuna) = manager.get_combatant("両面宿儺") {
        println!("両面宿儺: HP {}/1800 ({:.1}%)",
            sukuna.hp, sukuna.get_hp_percentage());
    }

    // 最終統計
    println!("\\n{}", manager.generate_overall_stats());
}
```

</div>
</details>

## 章末総括

第2章の練習問題、お疲れ様！所有権・借用・ライフタイムという三位一体の力を実践で確認できたな。

これらの概念を通して学んだこと：

- **所有権** - メモリ安全性の基盤
- **借用** - 効率的なデータ共有
- **ライフタイム** - 参照の生存期間管理
- **コンパイラとの対話** - エラーメッセージから学ぶ

!!! tip "五条先生の最終メッセージ"
    Rustの所有権システムは最初は複雑に感じるかもしれない。でも一度マスターすれば、他の言語では味わえない安全性と性能を手に入れられる。

    俺の無下限術式と同じで、理解すれば絶対的な力になる。諦めずに練習を続けろ。

次は第3章「反転術式編」でエラーハンドリングを学ぼう。エラーを逆手に取る技術だ。

______________________________________________________________________

*「所有権を極めれば、メモリの支配者になれる」*
