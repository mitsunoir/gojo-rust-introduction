# 制御構文 - 術式の流れを操る

## 条件分岐 - 状況に応じた術式選択

呪術師は状況に応じて最適な術式を選ぶ必要がある。Rustでも同じだ。条件によってプログラムの流れを制御できる。

### if式 - 基本的な判断

```rust
fn main() {
    let enemy_power = 1500;
    let my_power = 2000;

    if my_power > enemy_power {
        println!(\"楽勝だね\");
    } else if my_power == enemy_power {
        println!(\"互角か...面白い\");
    } else {
        println!(\"ちょっと本気出すかな\");
    }
}
```

!!! note "五条先生のポイント"
    Rustの`if`は**式**だから値を返せる。これは俺の術式選択みたいに、条件によって結果が決まるんだ。

### if式で値を返す

```rust
fn main() {
    let enemy_type = \"特級呪霊\";

    let technique = if enemy_type == \"特級呪霊\" {
        \"術式順転『蒼』\"
    } else if enemy_type == \"偽夏油\" {
        \"術式反転『赤』\"
    } else {
        \"基本術式\"
    };

    println!(\"使用する技: {}\", technique);

    // 数値での例
    let power_level = 1000;
    let damage = if power_level > 500 { power_level * 2 } else { power_level };

    println!(\"ダメージ: {}\", damage);
}
```

## match式 - パターンマッチング

これが本当の力だ。`match`式は俺の領域展開みたいに、あらゆる可能性を完璧にカバーできる。

### 基本的なmatch

```rust
fn main() {
    let technique_type = \"蒼\";

    match technique_type {
        \"蒼\" => println!(\"術式順転『蒼』 - 引力操作\"),
        \"赤\" => println!(\"術式反転『赤』 - 斥力操作\"),
        \"茈\" => println!(\"虚式『茈』 - 仮想質量\"),
        \"紫\" => println!(\"無下限呪術『紫』 - 最強\"),
        _ => println!(\"未知の術式\"),  // その他全て
    }
}
```

### 値を返すmatch

```rust
fn calculate_damage(technique: &str) -> i32 {
    match technique {
        \"蒼\" => 1000,
        \"赤\" => 1500,
        \"茈\" => 3000,
        \"紫\" => 9999,
        _ => 100,
    }
}

fn main() {
    let techniques = [\"蒼\", \"赤\", \"茈\", \"紫\"];

    for tech in techniques.iter() {
        let damage = calculate_damage(tech);
        println!(\"{}: ダメージ {}\", tech, damage);
    }
}
```

### 複数のパターン

```rust
fn main() {
    let enemy_grade = 2;

    let strategy = match enemy_grade {
        1 | 2 => \"基本術式で十分\",
        3 => \"少し本気を出そう\",
        4 => \"特級か...面白い\",
        0 => \"特級の中でも格が違う\",
        _ => \"未知の等級\",
    };

    println!(\"戦略: {}\", strategy);
}
```

### 範囲でのマッチ

```rust
fn main() {
    let power_level = 1500;

    let comment = match power_level {
        0..=500 => \"雑魚だね\",
        501..=1000 => \"まあまあかな\",
        1001..=2000 => \"そこそこやるじゃない\",
        2001..=5000 => \"お、なかなか\",
        5001.. => \"ほほう、面白い\",
    };

    println!(\"評価: {}\", comment);
}
```

## ループ - 術式の反復

### loop - 無限ループ

```rust
fn main() {
    let mut curse_count = 5;

    loop {
        if curse_count == 0 {
            println!(\"全ての呪霊を祓った！\");
            break;  // ループから抜ける
        }

        println!(\"呪霊を祓った。残り: {}\", curse_count);
        curse_count -= 1;
    }
}
```

### loopで値を返す

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;  // 値を返してループ終了
        }
    };

    println!(\"結果: {}\", result);  // 20
}
```

### while - 条件付きループ

```rust
fn main() {
    let mut enemy_hp = 1000;
    let damage = 150;

    while enemy_hp > 0 {
        enemy_hp -= damage;
        println!(\"攻撃！残りHP: {}\", enemy_hp.max(0));
    }

    println!(\"敵を倒した！\");
}
```

### for - イテレータループ

```rust
fn main() {
    // 範囲でのループ
    println!(\"カウントダウン:\");
    for i in (1..=5).rev() {
        println!(\"{}\", i);
    }
    println!(\"術式発動！\");

    // 配列のループ
    let techniques = [\"蒼\", \"赤\", \"茈\"];
    for (index, technique) in techniques.iter().enumerate() {
        println!(\"技{}: {}\", index + 1, technique);
    }

    // ベクターのループ
    let mut enemies = vec![\"呪霊A\", \"呪霊B\", \"呪霊C\"];
    for enemy in enemies.iter_mut() {
        *enemy = \"撃破済み\";
    }
    println!(\"敵の状態: {:?}\", enemies);
}
```

## ループ制御 - break と continue

```rust
fn main() {
    println!(\"=== break の例 ===\");
    for i in 1..10 {
        if i == 5 {
            println!(\"{}で中断\", i);
            break;
        }
        println!(\"数値: {}\", i);
    }

    println!(\"\\n=== continue の例 ===\");
    for i in 1..=10 {
        if i % 2 == 0 {
            continue;  // 偶数はスキップ
        }
        println!(\"奇数: {}\", i);
    }
}
```

### ラベル付きループ

```rust
fn main() {
    'outer: for x in 1..=3 {
        'inner: for y in 1..=3 {
            if x == 2 && y == 2 {
                println!(\"({}, {}) で外側ループを抜ける\", x, y);
                break 'outer;  // 外側のループを抜ける
            }
            println!(\"({}, {})\", x, y);
        }
    }
    println!(\"完了\");
}
```

## 実践例 - 呪霊討伐シミュレーター

```rust
fn main() {
    let mut enemies = vec![
        (\"1級呪霊\", 500),
        (\"特級呪霊\", 1500),
        (\"特級呪霊(偽夏油)\", 3000),
    ];

    let techniques = [
        (\"蒼\", 600),
        (\"赤\", 800),
        (\"茈\", 2000),
        (\"紫\", 9999),
    ];

    for (enemy_name, enemy_hp) in enemies.iter() {
        println!(\"\\n{} (HP: {}) が現れた！\", enemy_name, enemy_hp);

        let selected_technique = match *enemy_hp {
            0..=500 => &techniques[0],      // 蒼
            501..=1000 => &techniques[1],   // 赤
            1001..=2000 => &techniques[2],  // 茈
            _ => &techniques[3],            // 紫
        };

        let (tech_name, damage) = selected_technique;

        println!(\"『{}』を使用！\", tech_name);

        if damage >= enemy_hp {
            println!(\"{}ダメージ！{} を撃破！\", damage, enemy_name);
        } else {
            println!(\"{}ダメージだが、まだ生きている...\", damage);
        }
    }

    println!(\"\\n全ての敵を倒した！最強だから当然だね。\");
}
```

## 練習問題

<div class=\"exercise\">

### 問題1: 呪力判定システム

呪力値に応じて等級を判定するプログラムを`match`式で作成せよ。

- 0-100: 4級
- 101-500: 3級
- 501-1000: 2級
- 1001-2000: 1級
- 2001以上: 特級

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
fn judge_grade(power: i32) -> &'static str {
    match power {
        0..=100 => \"4級\",
        101..=500 => \"3級\",
        501..=1000 => \"2級\",
        1001..=2000 => \"1級\",
        2001.. => \"特級\",
    }
}

fn main() {
    let power_levels = [50, 300, 800, 1500, 3000];

    for power in power_levels.iter() {
        let grade = judge_grade(*power);
        println!(\"呪力{}: {} 呪術師\", power, grade);
    }
}
```

</div>
</details>

<div class=\"exercise\">

### 問題2: FizzBuzz呪術版

1から30まで数えて、3の倍数で「呪", 5の倍数で「術", 両方の倍数で「呪術"と表示するプログラムを作成せよ。

</div>

<details>
<summary>解答を見る</summary>
<div class=\"solution\">

```rust
fn main() {
    for i in 1..=30 {
        match (i % 3, i % 5) {
            (0, 0) => println!(\"呪術\"),
            (0, _) => println!(\"呪\"),
            (_, 0) => println!(\"術\"),
            _ => println!(\"{}\", i),
        }
    }
}
```

</div>
</details>

## まとめ

制御構文の習得は完了だ。重要なポイント：

1. **if式** - 条件に応じた分岐、値も返せる
1. **match式** - パターンマッチング、全パターン網羅必須
1. **loop** - 無限ループ、値を返せる
1. **while** - 条件付きループ
1. **for** - イテレータループ、最も使用頻度が高い
1. **break/continue** - ループ制御

次は関数について学ぼう。関数は術式をモジュール化する重要な概念だ。

______________________________________________________________________

*「制御の流れを極めれば、どんな複雑な術式も組み立てられる」*
