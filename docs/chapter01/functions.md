# 関数 - 術式のモジュール化

## 関数の基本 - 術式の定義

関数は術式をパッケージ化したものだ。同じ術式を何度も使い回せるし、複雑な術式を小さな部品に分けて管理できる。

### 基本的な関数定義

```rust
fn greet() {
    println!(\"やあやあ、俺は五条悟だ\");
}

fn main() {
    greet();  // 関数の呼び出し
    greet();  // 何度でも呼び出せる
}
```

### パラメータ付き関数

```rust
fn cast_technique(name: &str, power: i32) {
    println!(\"{}を発動！威力: {}\", name, power);
}

fn main() {
    cast_technique(\"術式順転『蒼』\", 1000);
    cast_technique(\"術式反転『赤』\", 1500);
    cast_technique(\"虚式『茈』\", 3000);
}
```

!!! note "パラメータの型指定"
    関数のパラメータは必ず型を明示する必要がある。これは俺の術式と同じで、正確な定義が威力を決めるんだ。

### 戻り値のある関数

```rust
fn calculate_damage(base_power: i32, multiplier: f64) -> i32 {
    (base_power as f64 * multiplier) as i32
}

fn get_technique_name(level: i32) -> &'static str {
    if level >= 3000 {
        \"虚式『茈』\"
    } else if level >= 1500 {
        \"術式反転『赤』\"
    } else {
        \"術式順転『蒼』\"
    }
}

fn main() {
    let damage = calculate_damage(1000, 2.5);
    let technique = get_technique_name(damage);

    println!(\"ダメージ: {}\", damage);
    println!(\"推奨術式: {}\", technique);
}
```

### 早期リターン

```rust
fn check_enemy_threat(enemy_power: i32) -> String {
    if enemy_power <= 0 {
        return String::from(\"敵がいない\");  // 早期リターン
    }

    if enemy_power > 5000 {
        return String::from(\"ちょっと本気出すかな\");
    }

    String::from(\"雑魚だね\")  // 最後の式が戻り値
}

fn main() {
    let threats = [0, 1000, 6000];

    for power in threats.iter() {
        println!(\"敵の力: {} -> {}\", power, check_enemy_threat(*power));
    }
}
```

## 式と文の詳細

```rust
fn power_calculation() -> i32 {
    let base = 1000;
    let bonus = 500;

    // これは式（セミコロンなし）- 戻り値になる
    base + bonus
}

fn technique_analysis() -> (String, i32, bool) {
    let name = String::from(\"無下限呪術\");
    let power = 9999;
    let is_defensive = false;

    // タプルで複数の値を返す
    (name, power, is_defensive)
}

fn main() {
    let total_power = power_calculation();
    let (tech_name, tech_power, is_def) = technique_analysis();

    println!(\"総呪力: {}\", total_power);
    println!(\"技: {} (威力: {}, 防御: {})\", tech_name, tech_power, is_def);
}
```

## 所有権と関数

### 値の移動

```rust
fn take_ownership(technique: String) {
    println!(\"{}を受け取った\", technique);
}  // ここでtechniqueがドロップされる

fn main() {
    let my_technique = String::from(\"術式順転『蒼』\");
    take_ownership(my_technique);

    // println!(\"{}\", my_technique);  // エラー！所有権が移動した
}
```

### 参照の使用

```rust
fn analyze_technique(technique: &String) -> usize {
    technique.len()  // 参照なので所有権は移動しない
}

fn main() {
    let my_technique = String::from(\"術式順転『蒼』\");
    let length = analyze_technique(&my_technique);

    println!(\"技名: {}\", my_technique);  // まだ使える！
    println!(\"文字数: {}\", length);
}
```

### 可変参照

```rust
fn power_up(technique: &mut String) {
    technique.push_str(\" - 最大出力\");
}

fn main() {
    let mut my_technique = String::from(\"術式順転『蒼』\");
    println!(\"変更前: {}\", my_technique);

    power_up(&mut my_technique);
    println!(\"変更後: {}\", my_technique);
}
```

## 高度な関数機能

### 関数ポインタ

```rust
fn blue_technique() -> i32 { 1000 }
fn red_technique() -> i32 { 1500 }
fn hollow_purple() -> i32 { 9999 }

fn execute_technique(technique_fn: fn() -> i32) -> i32 {
    technique_fn()
}

fn main() {
    let techniques = [blue_technique, red_technique, hollow_purple];

    for (i, technique) in techniques.iter().enumerate() {
        let damage = execute_technique(*technique);
        println!(\"術式{}: ダメージ {}\", i + 1, damage);
    }
}
```

### クロージャ（無名関数）

```rust
fn main() {
    let base_power = 1000;

    // クロージャの定義
    let calculate_damage = |multiplier: f64| -> i32 {
        (base_power as f64 * multiplier) as i32
    };

    // クロージャの使用
    let normal_damage = calculate_damage(1.0);
    let critical_damage = calculate_damage(2.5);

    println!(\"通常ダメージ: {}\", normal_damage);
    println!(\"クリティカル: {}\", critical_damage);

    // より簡潔な書き方
    let quick_calc = |x| x * 2;
    println!(\"倍率計算: {}\", quick_calc(500));
}
```

### 高階関数

```rust
fn apply_multiplier<F>(power: i32, func: F) -> i32
where
    F: Fn(i32) -> i32
{
    func(power)
}

fn main() {
    let base_power = 1000;

    // 異なる変換関数を適用
    let doubled = apply_multiplier(base_power, |x| x * 2);
    let squared = apply_multiplier(base_power, |x| x * x);
    let boosted = apply_multiplier(base_power, |x| x + 500);

    println!(\"基本威力: {}\", base_power);
    println!(\"2倍: {}\", doubled);
    println!(\"2乗: {}\", squared);
    println!(\"強化: {}\", boosted);
}
```

## 実践例 - 呪術戦闘システム

```rust
// 呪術師の情報
struct Sorcerer {
    name: String,
    power: i32,
    health: i32,
}

impl Sorcerer {
    fn new(name: &str, power: i32) -> Self {
        Sorcerer {
            name: String::from(name),
            power,
            health: 100,
        }
    }
}

// 攻撃関数
fn attack(attacker: &Sorcerer, target: &mut Sorcerer, technique_name: &str) {
    let damage = calculate_technique_damage(&attacker, technique_name);
    target.health = (target.health - damage).max(0);

    println!(\"{} が {} で {} を攻撃！\",
             attacker.name, technique_name, target.name);
    println!(\"{}ダメージ！{}の残りHP: {}\",
             damage, target.name, target.health);
}

// 術式ダメージ計算
fn calculate_technique_damage(sorcerer: &Sorcerer, technique: &str) -> i32 {
    let base_damage = match technique {
        \"術式順転『蒼』\" => sorcerer.power,
        \"術式反転『赤』\" => (sorcerer.power as f64 * 1.5) as i32,
        \"虚式『茈』\" => sorcerer.power * 2,
        \"無下限呪術『紫』\" => sorcerer.power * 10,
        _ => sorcerer.power / 2,
    };

    // ランダム要素（簡易版）
    let variation = (base_damage as f64 * 0.1) as i32;
    base_damage + variation
}

// 戦闘終了判定
fn is_battle_over(participants: &[Sorcerer]) -> bool {
    participants.iter().any(|s| s.health <= 0)
}

// 生存者発表
fn announce_winner(participants: &[Sorcerer]) {
    for sorcerer in participants {
        if sorcerer.health > 0 {
            println!(\"勝者: {} (残りHP: {})\", sorcerer.name, sorcerer.health);
            return;
        }
    }
    println!(\"引き分け！\");
}

fn main() {
    let mut gojo = Sorcerer::new(\"五条悟\", 2000);
    let mut sukuna = Sorcerer::new(\"両面宿儺\", 1800);

    println!(\"=== 最強対決開始！ ===\");
    println!(\"{} (HP: {}) vs {} (HP: {})\",
             gojo.name, gojo.health, sukuna.name, sukuna.health);

    // 戦闘ループ
    let mut round = 1;
    loop {
        println!(\"\\n--- ラウンド {} ---\", round);

        // 五条の攻撃
        if gojo.health > 0 {
            let technique = if round >= 3 { \"無下限呪術『紫』\" } else { \"術式順転『蒼』\" };
            attack(&gojo, &mut sukuna, technique);
        }

        if is_battle_over(&[gojo.clone(), sukuna.clone()]) {
            break;
        }

        // 宿儺の攻撃
        if sukuna.health > 0 {
            attack(&sukuna, &mut gojo, \"解\");
        }

        if is_battle_over(&[gojo.clone(), sukuna.clone()]) {
            break;
        }

        round += 1;
        if round > 5 {  // 無限ループ防止
            break;
        }
    }

    println!(\"\\n=== 戦闘終了 ===\");
    announce_winner(&[gojo, sukuna]);
}

// 簡易的なclone実装のため
impl Clone for Sorcerer {
    fn clone(&self) -> Self {
        Sorcerer {
            name: self.name.clone(),
            power: self.power,
            health: self.health,
        }
    }
}
```

## 練習問題

<div class=\"exercise\">

### 問題1: 呪力変換関数

整数の呪力値を受け取り、以下の変換を行う関数を作成せよ：

- 入力が負の場合は0を返す
- 1000以下の場合は2倍する
- 1000を超える場合は平方根を取る（整数で返す）

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
fn convert_power(power: i32) -> i32 {
    if power < 0 {
        0
    } else if power <= 1000 {
        power * 2
    } else {
        (power as f64).sqrt() as i32
    }
}

fn main() {
    let test_powers = [-100, 500, 1000, 2500, 10000];

    for power in test_powers.iter() {
        let converted = convert_power(*power);
        println!(\"元の呪力: {} -> 変換後: {}\", power, converted);
    }
}
```

</div>
</details>

<div class=\"exercise\">

### 問題2: 術式コンボ関数

2つの術式名を受け取り、組み合わせによって新しい技名とダメージを返す関数を作成せよ。

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
fn combo_technique(tech1: &str, tech2: &str) -> (String, i32) {
    match (tech1, tech2) {
        (\"蒼\", \"赤\") | (\"赤\", \"蒼\") => (String::from(\"虚式『茈』\"), 3000),
        (\"蒼\", \"蒼\") => (String::from(\"強化『蒼』\"), 1500),
        (\"赤\", \"赤\") => (String::from(\"強化『赤』\"), 2000),
        (\"茈\", _) | (_, \"茈\") => (String::from(\"超虚式『茈』\"), 5000),
        _ => (format!(\"{} + {}\", tech1, tech2), 800),
    }
}

fn main() {
    let combinations = [
        (\"蒼\", \"赤\"),
        (\"蒼\", \"蒼\"),
        (\"赤\", \"赤\"),
        (\"茈\", \"蒼\"),
        (\"基本\", \"基本\"),
    ];

    for (tech1, tech2) in combinations.iter() {
        let (combo_name, damage) = combo_technique(tech1, tech2);
        println!(\"{} + {} = {} (ダメージ: {})\",
                 tech1, tech2, combo_name, damage);
    }
}
```

</div>
</details>

## まとめ

関数の習得は完了だ！重要なポイント：

1. **関数定義** - `fn`キーワード、パラメータ型必須
1. **戻り値** - `->` で型指定、最後の式が戻り値
1. **所有権** - 値渡し、参照渡しを適切に使い分け
1. **クロージャ** - 無名関数、環境をキャプチャ
1. **高階関数** - 関数を引数に取る関数

これで第1章「術式習得編」は完了だ。基本的な術式はマスターできたな。次は第2章で所有権システムという、Rustの真の力を学んでいこう。

______________________________________________________________________

*「関数は術式の基本。これを極めれば複雑な呪術も組み立てられる」*
