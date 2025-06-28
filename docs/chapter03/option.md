# 第3章：反転術式編 - Option型

## 反転術式の概念 - エラーを力に変える

さあ、いよいよ反転術式の習得だ。俺の術式反転『赤』が引力を斥力に変えるように、Rustでは失敗や不在を安全に扱う技術がある。

従来の言語では、値が存在しない場合に`null`や`undefined`を使っていた。でもこれは呪いのようなもので、予期しないクラッシュを引き起こす。Rustは違う。**Option型**で安全に「値があるかもしれない」状況を表現する。

!!! note "五条先生の解説"
    Option型は「値があるかもしれないし、ないかもしれない」状況を型安全に表現する。
    `Some(T)`で値がある場合、`None`で値がない場合を表す。nullポインタ例外は起こりえない。

## Option型の基本

### Option型の定義

```rust
// Rustの標準ライブラリで定義されている
enum Option<T> {
    Some(T),  // 値がある
    None,     // 値がない
}
```

### 基本的な使用法

```rust
fn main() {
    // 値がある場合
    let technique_name: Option<String> = Some(String::from(\"術式順転『蒼』\"));

    // 値がない場合
    let unknown_technique: Option<String> = None;

    // Option型の値を表示
    println!(\"{:?}\", technique_name);  // Some(\"術式順転『蒼』\")
    println!(\"{:?}\", unknown_technique);  // None
}
```

## Option型の作成

### 直接作成

```rust
fn main() {
    let some_power = Some(1500);
    let no_power: Option<i32> = None;

    let technique = Some(\"無下限呪術\");
    let empty_technique: Option<&str> = None;

    println!(\"呪力: {:?}\", some_power);
    println!(\"技: {:?}\", technique);
}
```

### 関数から返す

```rust
fn find_technique_power(technique_name: &str) -> Option<i32> {
    match technique_name {
        \"蒼\" => Some(1000),
        \"赤\" => Some(1500),
        \"茈\" => Some(3000),
        \"紫\" => Some(9999),
        _ => None,  // 未知の技
    }
}

fn get_strongest_student() -> Option<String> {
    // 条件によって値を返すかNoneを返す
    let has_students = true;

    if has_students {
        Some(String::from(\"虎杖悠仁\"))
    } else {
        None
    }
}

fn main() {
    let blue_power = find_technique_power(\"蒼\");
    let unknown_power = find_technique_power(\"未知の技\");

    println!(\"蒼の威力: {:?}\", blue_power);      // Some(1000)
    println!(\"未知技の威力: {:?}\", unknown_power); // None

    let strongest = get_strongest_student();
    println!(\"最強の学生: {:?}\", strongest);
}
```

## Option型の処理方法

### match式による処理

```rust
fn process_technique_power(power: Option<i32>) {
    match power {
        Some(p) => println!(\"威力: {}\", p),
        None => println!(\"技が見つかりません\"),
    }
}

fn get_power_level(power: Option<i32>) -> String {
    match power {
        Some(p) if p >= 3000 => \"超強力\".to_string(),
        Some(p) if p >= 1500 => \"強力\".to_string(),
        Some(p) if p >= 1000 => \"普通\".to_string(),
        Some(p) => format!(\"弱い ({})\", p),
        None => \"不明\".to_string(),
    }
}

fn main() {
    let techniques = [
        (\"蒼\", Some(1000)),
        (\"赤\", Some(1500)),
        (\"茈\", Some(3000)),
        (\"未知\", None),
    ];

    for (name, power) in techniques.iter() {
        println!(\"{}: {}\", name, get_power_level(*power));
        process_technique_power(*power);
        println!();
    }
}
```

### if let を使った処理

```rust
fn main() {
    let technique_power = Some(1500);

    // if let で値がある場合のみ処理
    if let Some(power) = technique_power {
        println!(\"技の威力: {}\", power);

        if power > 1000 {
            println!(\"強力な技です！\");
        }
    } else {
        println!(\"技が見つかりません\");
    }

    // より複雑な例
    let student_grade = Some(85);

    if let Some(grade) = student_grade {
        let evaluation = match grade {
            90..=100 => \"優秀\",
            80..=89 => \"良好\",
            70..=79 => \"普通\",
            _ => \"要努力\",
        };
        println!(\"成績: {}点 - {}\", grade, evaluation);
    }
}
```

## Option型のメソッド

### unwrap系メソッド

```rust
fn main() {
    let some_power = Some(1500);
    let no_power: Option<i32> = None;

    // unwrap() - 値を取り出すが、Noneの場合はパニック
    let power = some_power.unwrap();
    println!(\"威力: {}\", power);

    // no_power.unwrap(); // パニックする！

    // unwrap_or() - Noneの場合はデフォルト値を使用
    let safe_power = no_power.unwrap_or(0);
    println!(\"安全な威力: {}\", safe_power);

    // unwrap_or_else() - Noneの場合はクロージャを実行
    let calculated_power = no_power.unwrap_or_else(|| {
        println!(\"デフォルト値を計算中...\");
        500
    });
    println!(\"計算された威力: {}\", calculated_power);
}
```

!!! warning "unwrap()の注意点"
    `unwrap()`はNoneの場合にプログラムがクラッシュする。本番コードでは避けるべきだ。
    でも学習やプロトタイピングでは便利なので、適切に使い分けろ。

### expect() メソッド

```rust
fn main() {
    let technique = Some(\"術式順転『蒼』\");
    let empty_technique: Option<&str> = None;

    // expect() - unwrap()と同じだが、エラーメッセージを指定できる
    let tech_name = technique.expect(\"技名が必要です\");
    println!(\"技: {}\", tech_name);

    // エラーメッセージ付きでパニック
    // let empty_name = empty_technique.expect(\"技名が見つかりません！\");
}
```

### is_some() と is_none()

```rust
fn check_technique_availability(technique: Option<&str>) {
    if technique.is_some() {
        println!(\"技が利用可能です: {:?}\", technique);
    } else {
        println!(\"技が利用できません\");
    }

    // ガード節として使用
    if technique.is_none() {
        println!(\"技がないので早期リターン\");
        return;
    }

    println!(\"技の処理を続行...\");
}

fn main() {
    let available_technique = Some(\"術式順転『蒼』\");
    let unavailable_technique: Option<&str> = None;

    check_technique_availability(available_technique);
    check_technique_availability(unavailable_technique);
}
```

### map() メソッド - 関数型の力

```rust
fn main() {
    let powers = vec![Some(1000), None, Some(1500), Some(2000)];

    // map() - Someの場合のみ関数を適用
    let doubled_powers: Vec<Option<i32>> = powers.iter()
        .map(|power| power.map(|p| p * 2))
        .collect();

    println!(\"元の威力: {:?}\", powers);
    println!(\"2倍の威力: {:?}\", doubled_powers);

    // より複雑な変換
    let power_descriptions: Vec<Option<String>> = powers.iter()
        .map(|power| power.map(|p| {
            if p >= 2000 {
                format!(\"超強力 ({})\", p)
            } else if p >= 1500 {
                format!(\"強力 ({})\", p)
            } else {
                format!(\"普通 ({})\", p)
            }
        }))
        .collect();

    println!(\"威力説明: {:?}\", power_descriptions);
}
```

### and_then() メソッド - モナド的合成

```rust
fn get_technique_power(name: &str) -> Option<i32> {
    match name {
        \"蒼\" => Some(1000),
        \"赤\" => Some(1500),
        \"茈\" => Some(3000),
        _ => None,
    }
}

fn calculate_combo_power(power: i32) -> Option<i32> {
    if power >= 1000 {
        Some(power * 2)
    } else {
        None  // 弱すぎてコンボにならない
    }
}

fn main() {
    let technique_name = \"蒼\";

    // and_then() でOptionを返す関数をチェーン
    let combo_power = get_technique_power(technique_name)
        .and_then(calculate_combo_power);

    println!(\"{} のコンボ威力: {:?}\", technique_name, combo_power);

    // より複雑なチェーン
    let result = Some(\"茈\")
        .and_then(get_technique_power)
        .and_then(calculate_combo_power)
        .map(|power| format!(\"最終威力: {}\", power));

    println!(\"チェーン結果: {:?}\", result);
}
```

### filter() メソッド

```rust
fn main() {
    let powers = vec![Some(500), Some(1500), Some(3000), None];

    // filter() - 条件を満たす場合のみSomeを保持
    let strong_powers: Vec<Option<i32>> = powers.iter()
        .map(|power| power.filter(|&&p| p >= 1000))
        .collect();

    println!(\"元の威力: {:?}\", powers);
    println!(\"強力な技のみ: {:?}\", strong_powers);

    // 実用例：有効な技のみフィルタ
    let technique_levels = vec![
        Some(95),   // 優秀
        Some(65),   // 要努力
        Some(88),   // 良好
        None,       // 未評価
    ];

    let excellent_techniques: Vec<Option<i32>> = technique_levels.iter()
        .map(|level| level.filter(|&&l| l >= 90))
        .collect();

    println!(\"優秀な技: {:?}\", excellent_techniques);
}
```

## 実践例 - 呪術師データベース

```rust
#[derive(Debug, Clone)]
struct Sorcerer {
    name: String,
    grade: String,
    power: i32,
    techniques: Vec<String>,
}

impl Sorcerer {
    fn new(name: &str, grade: &str, power: i32) -> Self {
        Sorcerer {
            name: String::from(name),
            grade: String::from(grade),
            power,
            techniques: Vec::new(),
        }
    }

    fn add_technique(&mut self, technique: String) {
        self.techniques.push(technique);
    }

    fn get_strongest_technique(&self) -> Option<&String> {
        // 最初の技を最強とする簡易実装
        self.techniques.first()
    }

    fn get_power_level(&self) -> &str {
        match self.power {
            3000.. => \"最強級\",
            2000..=2999 => \"特級\",
            1500..=1999 => \"1級\",
            1000..=1499 => \"2級\",
            500..=999 => \"3級\",
            _ => \"4級\",
        }
    }
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

    // 名前で検索 - Option<T>を返す
    fn find_by_name(&self, name: &str) -> Option<&Sorcerer> {
        self.sorcerers.iter()
            .find(|sorcerer| sorcerer.name == name)
    }

    // 等級で検索 - 最初の1人のみ
    fn find_first_by_grade(&self, grade: &str) -> Option<&Sorcerer> {
        self.sorcerers.iter()
            .find(|sorcerer| sorcerer.grade == grade)
    }

    // 最強の呪術師を取得
    fn get_strongest(&self) -> Option<&Sorcerer> {
        self.sorcerers.iter()
            .max_by_key(|sorcerer| sorcerer.power)
    }

    // 特定の威力以上の呪術師を取得
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

    // 呪術師の詳細情報を取得
    fn get_sorcerer_details(&self, name: &str) -> Option<String> {
        self.find_by_name(name)
            .map(|sorcerer| {
                let strongest_tech = sorcerer.get_strongest_technique()
                    .map(|tech| tech.as_str())
                    .unwrap_or(\"なし\");

                format!(
                    \"名前: {}\\n等級: {}\\n呪力: {} ({})\\n技数: {}\\n最強技: {}\",
                    sorcerer.name,
                    sorcerer.grade,
                    sorcerer.power,
                    sorcerer.get_power_level(),
                    sorcerer.techniques.len(),
                    strongest_tech
                )
            })
    }

    // 呪術師同士の実力比較
    fn compare_sorcerers(&self, name1: &str, name2: &str) -> Option<String> {
        let sorcerer1 = self.find_by_name(name1)?;
        let sorcerer2 = self.find_by_name(name2)?;

        let comparison = if sorcerer1.power > sorcerer2.power {
            format!(\"{} の方が強い (呪力差: {})\",
                sorcerer1.name, sorcerer1.power - sorcerer2.power)
        } else if sorcerer2.power > sorcerer1.power {
            format!(\"{} の方が強い (呪力差: {})\",
                sorcerer2.name, sorcerer2.power - sorcerer1.power)
        } else {
            \"互角の実力\".to_string()
        };

        Some(comparison)
    }
}

fn main() {
    let mut db = SorcererDatabase::new();

    // データの登録
    let mut gojo = Sorcerer::new(\"五条悟\", \"特級\", 3000);
    gojo.add_technique(\"無下限呪術\".to_string());
    gojo.add_technique(\"術式順転『蒼』\".to_string());
    gojo.add_technique(\"術式反転『赤』\".to_string());

    let mut sukuna = Sorcerer::new(\"両面宿儺\", \"特級\", 2800);
    sukuna.add_technique(\"解\".to_string());
    sukuna.add_technique(\"捌\".to_string());

    let mut yuji = Sorcerer::new(\"虎杖悠仁\", \"1級\", 1200);
    yuji.add_technique(\"黒閃\".to_string());

    db.add_sorcerer(gojo);
    db.add_sorcerer(sukuna);
    db.add_sorcerer(yuji);

    println!(\"=== 呪術師データベース検索 ===\");

    // 名前検索
    if let Some(details) = db.get_sorcerer_details(\"五条悟\") {
        println!(\"\\n五条悟の詳細:\\n{}\", details);
    }

    // 存在しない呪術師
    match db.find_by_name(\"存在しない呪術師\") {
        Some(sorcerer) => println!(\"見つかりました: {}\", sorcerer.name),
        None => println!(\"\\n'存在しない呪術師' は見つかりませんでした\"),
    }

    // 最強の呪術師
    if let Some(strongest) = db.get_strongest() {
        println!(\"\\n最強の呪術師: {} (呪力: {})\",
                 strongest.name, strongest.power);
    }

    // 実力比較
    if let Some(comparison) = db.compare_sorcerers(\"五条悟\", \"両面宿儺\") {
        println!(\"\\n実力比較: {}\", comparison);
    }

    // 高威力呪術師検索
    match db.find_by_min_power(2000) {
        Some(powerful_sorcerers) => {
            println!(\"\\n呪力2000以上の呪術師:\");
            for sorcerer in powerful_sorcerers {
                println!(\"  {} - 呪力: {}\", sorcerer.name, sorcerer.power);
            }
        },
        None => println!(\"\\n呪力2000以上の呪術師はいません\"),
    }
}
```

## Option型の連鎖操作

```rust
// 複雑な検索とデータ変換
fn process_sorcerer_data(db: &SorcererDatabase, name: &str) -> Option<String> {
    db.find_by_name(name)                    // Option<&Sorcerer>
        .filter(|s| s.power >= 1000)        // 十分強い場合のみ
        .and_then(|s| s.get_strongest_technique())  // 最強技を取得
        .map(|tech| format!(\"{}の奥義: {}\", name, tech))  // メッセージ作成
}

fn calculate_total_power(db: &SorcererDatabase, names: &[&str]) -> Option<i32> {
    let mut total = 0;

    for name in names {
        match db.find_by_name(name) {
            Some(sorcerer) => total += sorcerer.power,
            None => return None,  // 1人でも見つからなければNone
        }
    }

    Some(total)
}

fn main() {
    let mut db = SorcererDatabase::new();

    // データ省略...

    // 連鎖操作のテスト
    let result = process_sorcerer_data(&db, \"五条悟\");
    println!(\"処理結果: {:?}\", result);

    // 合計呪力計算
    let team = [\"五条悟\", \"虎杖悠仁\"];
    let total_power = calculate_total_power(&db, &team);
    println!(\"チーム総呪力: {:?}\", total_power);
}
```

## まとめ

Option型の基本はマスターできたか？重要なポイント：

1. **型安全性** - nullポインタ例外が起こりえない
1. **明示的な処理** - 値がない可能性を型で表現
1. **関数型メソッド** - map、and_then、filterなど
1. **パターンマッチング** - matchとif letで安全に処理
1. **チェーン操作** - 複数の操作を安全に連鎖

これは俺の反転術式と同じで、一見ネガティブな「値がない」状況を、型安全という強力な力に変える技術だ。

次はResult型について学ぼう。エラーハンドリングの真の力を見せてやる。

______________________________________________________________________

*「Optionを極めれば、不在すら味方にできる」*
