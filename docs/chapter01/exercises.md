# 第1章 練習問題 - 基本術式の試練

## 術式習得の確認

ここまでの学習で基本的な術式は身についたはずだ。でも本当にマスターできているか？実際に手を動かして確認してみよう。

俺の教え子たちも、頭で理解しただけじゃダメだ。実戦で使えるようになって初めて「習得した」と言える。

!!! tip "五条先生からのアドバイス"
    練習問題は段階的に難しくなっている。最初は簡単でも、最後の方はちょっと手強いぞ。
    でも心配するな。俺が直々に解答も用意してある。

## 初級編 - 基本術式

### 問題1: 変数と演算

五条先生の基本呪力は1500、レベルアップで300増加する。レベル5まで上がった時の最終呪力を計算して表示するプログラムを書け。

```rust
fn main() {
    // ここにコードを書く
}
```

<details>
<summary>解答を見る</summary>

```rust
fn main() {
    let base_power = 1500;
    let level_bonus = 300;
    let max_level = 5;

    let final_power = base_power + (level_bonus * max_level);

    println!("五条先生の基本呪力: {}", base_power);
    println!("レベル{}到達時の呪力: {}", max_level, final_power);

    // より詳細な表示
    for level in 1..=max_level {
        let current_power = base_power + (level_bonus * level);
        println!("レベル{}: 呪力 {}", level, current_power);
    }
}
```

</details>

### 問題2: 文字列操作

「術式順転」という文字列に「『蒼』」を追加して「術式順転『蒼』」にし、さらに「発動！」を追加するプログラムを書け。

```rust
fn main() {
    let base_technique = "術式順転";
    // ここにコードを書く
}
```

<details>
<summary>解答を見る</summary>

```rust
fn main() {
    let base_technique = "術式順転";

    // String型に変換して操作
    let mut full_technique = String::from(base_technique);
    full_technique.push_str("『蒼』");
    full_technique.push_str("発動！");

    println!("完成した呪文: {}", full_technique);

    // format!マクロを使う方法
    let formatted = format!("{}『蒼』発動！", base_technique);
    println!("format!使用: {}", formatted);
}
```

</details>

## 中級編 - 制御構文の応用

### 問題3: 呪術師等級判定システム

呪力値に応じて呪術師の等級を判定し、さらに特別な称号も付与するシステムを作成せよ。

**判定基準:**

- 0-500: 4級呪術師
- 501-1000: 3級呪術師
- 1001-1500: 2級呪術師
- 1501-2500: 1級呪術師
- 2501-4000: 特級呪術師
- 4001以上: 特級呪術師（最強候補）

**特別称号:**

- 偶数の呪力: 「安定型」
- 3の倍数: 「天才型」
- 5の倍数: 「努力型」
- 1000の倍数: 「規格外」

```rust
fn main() {
    let test_powers = [300, 750, 1200, 2000, 3000, 5000];

    for power in test_powers.iter() {
        // ここに判定ロジックを書く
    }
}
```

<details>
<summary>解答を見る</summary>

```rust
fn judge_grade(power: i32) -> String {
    let grade = match power {
        0..=500 => "4級呪術師",
        501..=1000 => "3級呪術師",
        1001..=1500 => "2級呪術師",
        1501..=2500 => "1級呪術師",
        2501..=4000 => "特級呪術師",
        _ => "特級呪術師（最強候補）",
    };

    let mut titles = Vec::new();

    if power % 2 == 0 {
        titles.push("安定型");
    }
    if power % 3 == 0 {
        titles.push("天才型");
    }
    if power % 5 == 0 {
        titles.push("努力型");
    }
    if power % 1000 == 0 {
        titles.push("規格外");
    }

    if titles.is_empty() {
        format!("{}", grade)
    } else {
        format!("{} [{}]", grade, titles.join(", "))
    }
}

fn main() {
    let test_powers = [300, 750, 1200, 2000, 3000, 5000];

    println!("=== 呪術師等級判定システム ===");
    for power in test_powers.iter() {
        let result = judge_grade(*power);
        println!("呪力{}: {}", power, result);
    }
}
```

</details>

### 問題4: 術式コンビネーション

以下の術式を組み合わせて新しい技を作成するシステムを実装せよ。

基本術式: 蒼(1000), 赫(1200), 黒(800), 白(600)

**組み合わせルール:**

- 蒼 + 赫 = 茈 (威力3000)
- 黒 + 白 = 陰陽 (威力1800)
- 同じ術式2つ = 強化版 (威力1.5倍)
- その他 = 基本コンボ (威力合計 × 1.2)

```rust
fn main() {
    let combinations = [
        ("蒼", "赫"),
        ("黒", "白"),
        ("蒼", "蒼"),
        ("蒼", "黒"),
    ];

    // ここに実装
}
```

<details>
<summary>解答を見る</summary>

```rust
fn get_base_power(technique: &str) -> i32 {
    match technique {
        "蒼" => 1000,
        "赫" => 1200,
        "黒" => 800,
        "白" => 600,
        _ => 0,
    }
}

fn combine_techniques(tech1: &str, tech2: &str) -> (String, i32) {
    match (tech1, tech2) {
        ("蒼", "赫") | ("赫", "蒼") => {
            (String::from("虚式『茈』"), 3000)
        },
        ("黒", "白") | ("白", "黒") => {
            (String::from("陰陽術"), 1800)
        },
        (a, b) if a == b => {
            let base = get_base_power(a);
            let enhanced = (base as f64 * 1.5) as i32;
            (format!("強化『{}』", a), enhanced)
        },
        (a, b) => {
            let total = get_base_power(a) + get_base_power(b);
            let combo_power = (total as f64 * 1.2) as i32;
            (format!("{}×{}コンボ", a, b), combo_power)
        }
    }
}

fn main() {
    let combinations = [
        ("蒼", "赫"),
        ("黒", "白"),
        ("蒼", "蒼"),
        ("蒼", "黒"),
        ("赫", "白"),
    ];

    println!("=== 術式コンビネーションシステム ===");

    for (tech1, tech2) in combinations.iter() {
        let (combo_name, power) = combine_techniques(tech1, tech2);
        println!("{} + {} → {} (威力: {})",
                 tech1, tech2, combo_name, power);
    }
}
```

</details>

## 上級編 - 関数とデータ構造

### 問題5: 呪霊討伐シミュレーター

呪霊の群れと戦うシミュレーターを作成せよ。以下の仕様を満たすこと：

**呪霊の種類:**

- 下級呪霊: HP 100, 経験値 10
- 中級呪霊: HP 300, 経験値 30
- 上級呪霊: HP 500, 経験値 50
- 特級呪霊: HP 1000, 経験値 100

**呪術師の能力:**

- 初期HP: 1000, 初期呪力: 1500
- レベルアップ: 経験値100で呪力+200
- 攻撃パターン: 基本攻撃(呪力×0.8), 必殺技(呪力×1.5, 3回に1回)

```rust
// 構造体の定義例
struct Curse {
    name: String,
    hp: i32,
    exp_value: i32,
}

struct Sorcerer {
    hp: i32,
    power: i32,
    exp: i32,
    level: i32,
}

fn main() {
    // ここに実装
}
```

<details>
<summary>解答を見る</summary>

```rust
#[derive(Clone)]
struct Curse {
    name: String,
    hp: i32,
    max_hp: i32,
    exp_value: i32,
}

struct Sorcerer {
    hp: i32,
    max_hp: i32,
    power: i32,
    exp: i32,
    level: i32,
    attack_count: i32,
}

impl Curse {
    fn new(curse_type: &str) -> Self {
        let (hp, exp) = match curse_type {
            "下級" => (100, 10),
            "中級" => (300, 30),
            "上級" => (500, 50),
            "特級" => (1000, 100),
            _ => (50, 5),
        };

        Curse {
            name: format!("{}呪霊", curse_type),
            hp,
            max_hp: hp,
            exp_value: exp,
        }
    }

    fn is_alive(&self) -> bool {
        self.hp > 0
    }

    fn take_damage(&mut self, damage: i32) {
        self.hp = (self.hp - damage).max(0);
    }
}

impl Sorcerer {
    fn new() -> Self {
        Sorcerer {
            hp: 1000,
            max_hp: 1000,
            power: 1500,
            exp: 0,
            level: 1,
            attack_count: 0,
        }
    }

    fn attack(&mut self, target: &mut Curse) -> i32 {
        self.attack_count += 1;

        let damage = if self.attack_count % 3 == 0 {
            println!("必殺技発動！");
            (self.power as f64 * 1.5) as i32
        } else {
            (self.power as f64 * 0.8) as i32
        };

        target.take_damage(damage);
        damage
    }

    fn gain_exp(&mut self, exp: i32) {
        self.exp += exp;

        // レベルアップ判定
        let new_level = (self.exp / 100) + 1;
        if new_level > self.level {
            let level_ups = new_level - self.level;
            self.level = new_level;
            self.power += level_ups * 200;
            println!("レベルアップ！ Lv.{} 呪力: {}", self.level, self.power);
        }
    }
}

fn battle(sorcerer: &mut Sorcerer, curse: &mut Curse) -> bool {
    println!("\\n{} との戦闘開始！", curse.name);

    while curse.is_alive() && sorcerer.hp > 0 {
        // 呪術師の攻撃
        let damage = sorcerer.attack(curse);
        println!("{} に {} ダメージ！ (残りHP: {})",
                 curse.name, damage, curse.hp);

        if !curse.is_alive() {
            println!("{} を撃破！", curse.name);
            sorcerer.gain_exp(curse.exp_value);
            return true;
        }

        // 呪霊の反撃（簡易版）
        let curse_damage = curse.max_hp / 10;
        sorcerer.hp = (sorcerer.hp - curse_damage).max(0);
        if curse_damage > 0 {
            println!("{} の反撃！ {} ダメージ (残りHP: {})",
                     curse.name, curse_damage, sorcerer.hp);
        }
    }

    false
}

fn main() {
    let mut gojo = Sorcerer::new();

    let curse_types = ["下級", "下級", "中級", "上級", "特級"];
    let mut defeated_count = 0;

    println!("=== 呪霊討伐シミュレーター ===");
    println!("呪術師ステータス: HP {} 呪力 {} レベル {}",
             gojo.hp, gojo.power, gojo.level);

    for curse_type in curse_types.iter() {
        if gojo.hp <= 0 {
            break;
        }

        let mut curse = Curse::new(curse_type);

        if battle(&mut gojo, &mut curse) {
            defeated_count += 1;
        }
    }

    println!("\\n=== 戦闘結果 ===");
    println!("撃破数: {}", defeated_count);
    println!("最終ステータス: HP {} 呪力 {} レベル {} 経験値 {}",
             gojo.hp, gojo.power, gojo.level, gojo.exp);

    if gojo.hp > 0 {
        println!("完全勝利！さすが最強だ。");
    } else {
        println!("まだまだ修行が足りないな。");
    }
}
```

</details>

### 問題6: 術式データベース

術式の情報を管理するシステムを作成せよ。以下の機能を実装すること：

- 術式の追加
- 名前による検索
- 威力でのソート
- 統計情報の表示

```rust
struct Technique {
    name: String,
    power: i32,
    element: String,
    user: String,
}

fn main() {
    // ここに実装
}
```

<details>
<summary>解答を見る</summary>

```rust
#[derive(Debug, Clone)]
struct Technique {
    name: String,
    power: i32,
    element: String,
    user: String,
}

impl Technique {
    fn new(name: &str, power: i32, element: &str, user: &str) -> Self {
        Technique {
            name: String::from(name),
            power,
            element: String::from(element),
            user: String::from(user),
        }
    }
}

struct TechniqueDatabase {
    techniques: Vec<Technique>,
}

impl TechniqueDatabase {
    fn new() -> Self {
        TechniqueDatabase {
            techniques: Vec::new(),
        }
    }

    fn add_technique(&mut self, technique: Technique) {
        self.techniques.push(technique);
        println!("術式 '{}' を追加しました", self.techniques.last().unwrap().name);
    }

    fn search_by_name(&self, name: &str) -> Vec<&Technique> {
        self.techniques.iter()
            .filter(|tech| tech.name.contains(name))
            .collect()
    }

    fn search_by_user(&self, user: &str) -> Vec<&Technique> {
        self.techniques.iter()
            .filter(|tech| tech.user == user)
            .collect()
    }

    fn sort_by_power(&self) -> Vec<&Technique> {
        let mut sorted: Vec<&Technique> = self.techniques.iter().collect();
        sorted.sort_by(|a, b| b.power.cmp(&a.power));
        sorted
    }

    fn show_statistics(&self) {
        if self.techniques.is_empty() {
            println!("術式データがありません");
            return;
        }

        let total_techniques = self.techniques.len();
        let total_power: i32 = self.techniques.iter().map(|t| t.power).sum();
        let average_power = total_power / total_techniques as i32;
        let max_power = self.techniques.iter().map(|t| t.power).max().unwrap();
        let min_power = self.techniques.iter().map(|t| t.power).min().unwrap();

        println!("\\n=== 術式データベース統計 ===");
        println!("総術式数: {}", total_techniques);
        println!("平均威力: {}", average_power);
        println!("最大威力: {}", max_power);
        println!("最小威力: {}", min_power);

        // 属性別集計
        let mut elements: std::collections::HashMap<String, i32> = std::collections::HashMap::new();
        for tech in &self.techniques {
            *elements.entry(tech.element.clone()).or_insert(0) += 1;
        }

        println!("\\n属性別統計:");
        for (element, count) in elements {
            println!("{}: {} 個", element, count);
        }
    }

    fn list_all(&self) {
        println!("\\n=== 全術式一覧 ===");
        for (i, tech) in self.techniques.iter().enumerate() {
            println!("{}. {} - 威力:{} 属性:{} 使用者:{}",
                     i + 1, tech.name, tech.power, tech.element, tech.user);
        }
    }
}

fn main() {
    let mut db = TechniqueDatabase::new();

    // サンプルデータの追加
    db.add_technique(Technique::new("術式順転『蒼』", 1000, "無下限", "五条悟"));
    db.add_technique(Technique::new("術式反転『赫』", 1500, "無下限", "五条悟"));
    db.add_technique(Technique::new("虚式『茈』", 3000, "無下限", "五条悟"));
    db.add_technique(Technique::new("解", 2000, "斬撃", "両面宿儺"));
    db.add_technique(Technique::new("捌", 2500, "斬撃", "両面宿儺"));
    db.add_technique(Technique::new("十種影法術", 800, "式神", "伏黒恵"));
    db.add_technique(Technique::new("釘崎野薔薇", 600, "呪具", "釘崎野薔薇"));

    // 全一覧表示
    db.list_all();

    // 統計情報
    db.show_statistics();

    // 検索テスト
    println!("\\n=== 検索テスト ===");
    let blue_results = db.search_by_name("蒼");
    println!("'蒼'を含む術式:");
    for tech in blue_results {
        println!("  {}", tech.name);
    }

    let gojo_techniques = db.search_by_user("五条悟");
    println!("\\n五条悟の術式:");
    for tech in gojo_techniques {
        println!("  {} (威力: {})", tech.name, tech.power);
    }

    // 威力順ソート
    println!("\\n=== 威力順ランキング ===");
    let sorted = db.sort_by_power();
    for (i, tech) in sorted.iter().take(5).enumerate() {
        println!("{}位: {} - 威力 {} ({})",
                 i + 1, tech.name, tech.power, tech.user);
    }
}
```

</details>

## 総合問題

### 問題7: 呪術高専入学試験シミュレーター

最後の問題だ。これまで学んだ全ての要素を使って、呪術高専の入学試験をシミュレートするプログラムを作成せよ。

**要件:**

- 受験者は名前、初期呪力、得意術式を持つ
- 3つの試験科目: 実技、筆記、面接
- 科目ごとに異なる評価方法
- 最終的な合否判定と講評

自由に設計して実装してみよう！

<details>
<summary>解答例を見る</summary>

```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
struct Student {
    name: String,
    base_power: i32,
    specialty: String,
    scores: HashMap<String, i32>,
    total_score: i32,
    passed: bool,
}

impl Student {
    fn new(name: &str, base_power: i32, specialty: &str) -> Self {
        Student {
            name: String::from(name),
            base_power,
            specialty: String::from(specialty),
            scores: HashMap::new(),
            total_score: 0,
            passed: false,
        }
    }

    fn add_score(&mut self, subject: &str, score: i32) {
        self.scores.insert(String::from(subject), score);
    }

    fn calculate_total(&mut self) {
        self.total_score = self.scores.values().sum();
    }
}

struct ExamSystem {
    students: Vec<Student>,
    pass_threshold: i32,
}

impl ExamSystem {
    fn new(pass_threshold: i32) -> Self {
        ExamSystem {
            students: Vec::new(),
            pass_threshold,
        }
    }

    fn register_student(&mut self, student: Student) {
        println!("{} が受験登録しました", student.name);
        self.students.push(student);
    }

    fn practical_exam(&mut self) {
        println!("\\n=== 実技試験開始 ===");

        for student in &mut self.students {
            let base_score = match student.specialty.as_str() {
                "攻撃術式" => 85,
                "防御術式" => 75,
                "補助術式" => 70,
                "特殊術式" => 80,
                _ => 60,
            };

            // 呪力による補正
            let power_bonus = (student.base_power / 100).min(20);
            let final_score = (base_score + power_bonus).min(100);

            student.add_score("実技", final_score);
            println!("{}: {}点 (得意: {}, 呪力補正: +{})",
                     student.name, final_score, student.specialty, power_bonus);
        }
    }

    fn written_exam(&mut self) {
        println!("\\n=== 筆記試験開始 ===");

        // 簡易的なランダム要素（実際はもっと複雑）
        let base_scores = [95, 85, 78, 92, 88, 76, 90, 82];

        for (i, student) in self.students.iter_mut().enumerate() {
            let score = base_scores[i % base_scores.len()];
            student.add_score("筆記", score);
            println!("{}: {}点", student.name, score);
        }
    }

    fn interview_exam(&mut self) {
        println!("\\n=== 面接試験開始 ===");

        for student in &mut self.students {
            let base_score = 70;

            // 特殊術式は面接で高評価
            let specialty_bonus = match student.specialty.as_str() {
                "特殊術式" => 15,
                "攻撃術式" => 5,
                "防御術式" => 10,
                "補助術式" => 12,
                _ => 0,
            };

            let final_score = (base_score + specialty_bonus).min(100);
            student.add_score("面接", final_score);
            println!("{}: {}点 (専門性評価: +{})",
                     student.name, final_score, specialty_bonus);
        }
    }

    fn judge_results(&mut self) {
        println!("\\n=== 最終判定 ===");

        for student in &mut self.students {
            student.calculate_total();
            student.passed = student.total_score >= self.pass_threshold;

            let status = if student.passed { "合格" } else { "不合格" };

            println!("{}: {}点 - {}",
                     student.name, student.total_score, status);

            // 詳細スコア
            println!("  実技: {}点, 筆記: {}点, 面接: {}点",
                     student.scores.get("実技").unwrap_or(&0),
                     student.scores.get("筆記").unwrap_or(&0),
                     student.scores.get("面接").unwrap_or(&0));
        }
    }

    fn show_statistics(&self) {
        println!("\\n=== 試験統計 ===");

        let total_students = self.students.len();
        let passed_students = self.students.iter().filter(|s| s.passed).count();
        let pass_rate = (passed_students as f64 / total_students as f64) * 100.0;

        println!("受験者数: {}", total_students);
        println!("合格者数: {}", passed_students);
        println!("合格率: {:.1}%", pass_rate);

        if let Some(top_student) = self.students.iter().max_by_key(|s| s.total_score) {
            println!("首席: {} ({}点)", top_student.name, top_student.total_score);
        }
    }

    fn gojo_comment(&self) {
        println!("\\n=== 五条先生の講評 ===");

        let passed_count = self.students.iter().filter(|s| s.passed).count();

        match passed_count {
            0 => println!("全員不合格か...まだまだ修行が足りないな。"),
            1..=2 => println!("少数精鋭だね。質の高い学生が入学してくる。"),
            3..=5 => println!("良い結果だ。期待できる新入生たちだね。"),
            _ => println!("大豊作だ！次世代の最強候補がたくさんいる。"),
        }

        // 高得点者へのコメント
        for student in &self.students {
            if student.total_score >= 270 {
                println!("{}: 素晴らしい！君には最強の素質がある。", student.name);
            } else if student.total_score >= 240 {
                println!("{}: とても良い成績だ。しっかり伸ばしていこう。", student.name);
            } else if student.passed {
                println!("{}: 合格おめでとう。これからが本番だぞ。", student.name);
            }
        }
    }
}

fn main() {
    println!("=== 東京呪術高等専門学校 入学試験 ===");

    let mut exam = ExamSystem::new(210);  // 合格ライン210点

    // 受験生登録
    exam.register_student(Student::new("田中呪介", 800, "攻撃術式"));
    exam.register_student(Student::new("佐藤霊子", 1200, "防御術式"));
    exam.register_student(Student::new("鈴木式神", 600, "補助術式"));
    exam.register_student(Student::new("高橋異能", 1500, "特殊術式"));
    exam.register_student(Student::new("伊藤呪力", 900, "攻撃術式"));

    // 試験実施
    exam.practical_exam();
    exam.written_exam();
    exam.interview_exam();

    // 結果発表
    exam.judge_results();
    exam.show_statistics();
    exam.gojo_comment();

    println!("\\n試験終了。新たな呪術師たちの門出を祝おう！");
}
```

</details>

## 章末総括

お疲れ様！第1章の練習問題は全てクリアできたか？

これらの問題を通して、以下の重要な概念を実践できたはずだ：

- **変数と型** - 適切なデータ管理
- **制御構文** - 条件分岐とループの活用
- **関数** - コードの再利用と構造化
- **データ構造** - 構造体とベクターの活用
- **パターンマッチング** - matchの強力さ

!!! tip "五条先生の最終アドバイス"
    プログラミングは暗記じゃない。理解して、手を動かして、実際に作ることが大切だ。

    この練習問題で躓いたところがあれば、恥ずかしがらずに基本に戻って復習しろ。
    俺の教え子たちも、最初は簡単なことから始めて、今では立派な呪術師になっている。

次は第2章「呪力操作編」で所有権システムを学ぶ。これがRustの真の力の源泉だ。

基礎をしっかり固めたからこそ、次のレベルに進める。準備はいいか？

______________________________________________________________________

*「基礎ができたからこそ、次のステージが見えてくる」*
