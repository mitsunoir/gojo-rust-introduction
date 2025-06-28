# マクロ - コード生成の究極奥義

## マクロの概念 - 現実改変の力

ついに到達したな、最高位の奥義「マクロ」だ。これは俺の「無量空処」を超える、**現実そのものを書き換える**力だ。

マクロはコンパイル時にコードを生成し、プログラマーの意図を無限に拡張する。まさに現実改変、コードという現実を思いのままに操る究極の抽象化技術だ。

!!! note "五条先生の解説"
    マクロは「コードを書くコード」。コンパイル時に実行され、新しいコードを生成する。
    関数とは違い、構文レベルでの変更が可能で、Rustの表現力を大幅に拡張できる。

## 宣言マクロ - macro_rules!

### 基本的なマクロ定義

```rust
// 最もシンプルなマクロ
macro_rules! technique_announce {
    () => {
        println!("術式発動準備完了！");
    };
}

// パラメータを受け取るマクロ
macro_rules! cast_technique {
    ($name:expr) => {
        println!("{}を発動！", $name);
    };
    ($name:expr, $power:expr) => {
        println!("{}を発動！威力: {}", $name, $power);
    };
}

// より複雑なパターンマッチング
macro_rules! sorcerer_info {
    ($name:expr) => {
        format!("呪術師: {}", $name)
    };
    ($name:expr, grade => $grade:expr) => {
        format!("呪術師: {} (等級: {})", $name, $grade)
    };
    ($name:expr, power => $power:expr) => {
        format!("呪術師: {} (呪力: {})", $name, $power)
    };
    ($name:expr, grade => $grade:expr, power => $power:expr) => {
        format!("呪術師: {} (等級: {}, 呪力: {})", $name, $grade, $power)
    };
}

fn main() {
    // 基本的な使用
    technique_announce!();

    // パラメータ付き
    cast_technique!("無下限術式");
    cast_technique!("黒閃", 1500);

    // 複雑なパターン
    println!("{}", sorcerer_info!("五条悟"));
    println!("{}", sorcerer_info!("虎杖悠仁", grade => "1級"));
    println!("{}", sorcerer_info!("伏黒恵", power => 1000));
    println!("{}", sorcerer_info!("釘崎野薔薇", grade => "2級", power => 900));
}
```

### 繰り返しパターン

```rust
// 可変長引数を受け取るマクロ
macro_rules! create_technique_list {
    ($($technique:expr),*) => {
        {
            let mut techniques = Vec::new();
            $(
                techniques.push($technique.to_string());
            )*
            techniques
        }
    };
}

// より複雑な繰り返し
macro_rules! define_sorcerers {
    ($(($name:expr, $power:expr)),*) => {
        {
            use std::collections::HashMap;
            let mut sorcerers = HashMap::new();
            $(
                sorcerers.insert($name.to_string(), $power);
            )*
            sorcerers
        }
    };
}

// ネストした繰り返し
macro_rules! technique_combinations {
    ($($sorcerer:ident: [$($technique:expr),*]),*) => {
        {
            use std::collections::HashMap;
            let mut combinations = HashMap::new();
            $(
                let mut techniques = Vec::new();
                $(
                    techniques.push($technique.to_string());
                )*
                combinations.insert(stringify!($sorcerer).to_string(), techniques);
            )*
            combinations
        }
    };
}

fn main() {
    // 術式リストの作成
    let techniques = create_technique_list![
        "無下限術式",
        "黒閃",
        "十種影法術",
        "共鳴"
    ];
    println!("術式一覧: {:?}", techniques);

    // 呪術師の定義
    let sorcerers = define_sorcerers![
        ("五条悟", 3000),
        ("虎杖悠仁", 1200),
        ("伏黒恵", 1000),
        ("釘崎野薔薇", 900)
    ];
    println!("呪術師と呪力: {:?}", sorcerers);

    // 複雑な組み合わせ
    let combinations = technique_combinations![
        gojo: ["無下限術式", "領域展開"],
        yuji: ["黒閃", "発散"],
        megumi: ["十種影法術", "布瑠部由良由良"],
        nobara: ["共鳴", "簪"]
    ];
    println!("術式の組み合わせ: {:#?}", combinations);
}
```

### 高度なマクロパターン

```rust
// 構造体自動生成マクロ
macro_rules! define_sorcerer_struct {
    ($name:ident {
        $(
            $field_name:ident: $field_type:ty $(= $default:expr)?
        ),*
    }) => {
        #[derive(Debug, Clone)]
        struct $name {
            $(
                $field_name: $field_type,
            )*
        }

        impl $name {
            fn new() -> Self {
                $name {
                    $(
                        $field_name: define_sorcerer_struct!(@default $field_type $(, $default)?),
                    )*
                }
            }

            // Getter methods
            $(
                paste::paste! {
                    fn [<get_ $field_name>](&self) -> &$field_type {
                        &self.$field_name
                    }

                    fn [<set_ $field_name>](&mut self, value: $field_type) {
                        self.$field_name = value;
                    }
                }
            )*
        }
    };

    // デフォルト値のヘルパー
    (@default $type:ty, $default:expr) => { $default };
    (@default String) => { String::new() };
    (@default i32) => { 0 };
    (@default bool) => { false };
    (@default Vec<$inner:ty>) => { Vec::new() };
}

// トレイト実装自動生成マクロ
macro_rules! impl_display_for_sorcerer {
    ($struct_name:ident, $format:expr) => {
        impl std::fmt::Display for $struct_name {
            fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
                write!(f, $format, self)
            }
        }
    };
}

// DSL（ドメイン固有言語）マクロ
macro_rules! battle_scenario {
    (
        fighters: [$($fighter:ident: $power:expr),*],
        rounds: $rounds:expr,
        $(
            round $round_num:expr => {
                $($action:ident($actor:ident $(, $target:ident)? $(, $value:expr)?));*
            }
        )*
    ) => {
        {
            use std::collections::HashMap;

            // 戦闘者の初期化
            let mut fighters = HashMap::new();
            $(
                fighters.insert(stringify!($fighter).to_string(), $power);
            )*

            println!("=== 戦闘シナリオ開始 ===");
            println!("参戦者: {:?}", fighters);

            // ラウンド実行
            $(
                println!("\n--- ラウンド {} ---", $round_num);
                $(
                    battle_scenario!(@action $action, fighters, $actor $(, $target)? $(, $value)?);
                )*
            )*

            println!("\n=== 戦闘結果 ===");
            for (name, power) in &fighters {
                println!("{}: 残り呪力 {}", name, power);
            }

            fighters
        }
    };

    // アクションの実行
    (@action attack, $fighters:ident, $actor:ident, $target:ident, $damage:expr) => {
        let actor_name = stringify!($actor);
        let target_name = stringify!($target);

        if let Some(target_power) = $fighters.get_mut(target_name) {
            *target_power = (*target_power - $damage).max(0);
            println!("{}が{}を攻撃！{}ダメージ", actor_name, target_name, $damage);
        }
    };

    (@action heal, $fighters:ident, $actor:ident, $amount:expr) => {
        let actor_name = stringify!($actor);

        if let Some(power) = $fighters.get_mut(actor_name) {
            *power += $amount;
            println!("{}が回復！+{}", actor_name, $amount);
        }
    };

    (@action special, $fighters:ident, $actor:ident, $target:ident, $damage:expr) => {
        let actor_name = stringify!($actor);
        let target_name = stringify!($target);

        if let Some(target_power) = $fighters.get_mut(target_name) {
            let actual_damage = $damage * 2; // 特殊攻撃は2倍
            *target_power = (*target_power - actual_damage).max(0);
            println!("{}が{}に特殊攻撃！{}ダメージ（2倍効果）", actor_name, target_name, actual_damage);
        }
    };
}

// 使用例（paste クレートが必要: cargo add paste）
/*
define_sorcerer_struct!(Sorcerer {
    name: String = "無名".to_string(),
    power: i32 = 1000,
    grade: String = "4級".to_string(),
    techniques: Vec<String>,
    is_active: bool = true
});

impl_display_for_sorcerer!(Sorcerer, "呪術師: {} (呪力: {})", self.name, self.power);
*/

fn main() {
    println!("=== 高度なマクロデモ ===");

    // 戦闘シナリオの実行
    let result = battle_scenario! {
        fighters: [gojo: 3000, sukuna: 2800, yuji: 1200],
        rounds: 3,

        round 1 => {
            attack(gojo, sukuna, 500);
            attack(sukuna, gojo, 600);
            heal(yuji, 200)
        }

        round 2 => {
            special(gojo, sukuna, 400);
            attack(sukuna, yuji, 300);
            attack(yuji, sukuna, 250)
        }

        round 3 => {
            special(sukuna, gojo, 450);
            special(gojo, sukuna, 500);
            heal(gojo, 100)
        }
    };

    // 勝者の判定
    let winner = result.iter().max_by_key(|(_, power)| *power);
    if let Some((name, power)) = winner {
        println!("\n🏆 勝者: {} (残り呪力: {})", name, power);
    }
}
```

## 手続きマクロ - proc_macro

### 属性マクロ

```rust
// 以下は手続きマクロの例（実際には別クレートが必要）

/*
// lib.rs または proc_macro クレート内
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

// derive マクロの例
#[proc_macro_derive(SorcererInfo)]
pub fn sorcerer_info_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = input.ident;

    let expanded = quote! {
        impl #name {
            fn sorcerer_type(&self) -> &'static str {
                stringify!(#name)
            }

            fn power_level(&self) -> String {
                format!("{}の呪力レベル", stringify!(#name))
            }
        }
    };

    TokenStream::from(expanded)
}

// 属性マクロの例
#[proc_macro_attribute]
pub fn technique_log(args: TokenStream, input: TokenStream) -> TokenStream {
    let input_fn = parse_macro_input!(input as syn::ItemFn);
    let fn_name = &input_fn.sig.ident;

    let expanded = quote! {
        #input_fn

        // ログ付きのラッパー関数を生成
        paste::paste! {
            pub fn [<logged_ #fn_name>]() {
                println!("術式 {} の実行開始", stringify!(#fn_name));
                #fn_name();
                println!("術式 {} の実行完了", stringify!(#fn_name));
            }
        }
    };

    TokenStream::from(expanded)
}
*/

// 使用例（実際には上記のマクロがクレートとして定義されている必要がある）
/*
#[derive(SorcererInfo)]
struct Gojo {
    name: String,
    power: i32,
}

#[technique_log]
fn limitless_technique() {
    println!("無下限術式発動！");
}
*/

// マクロ展開のシミュレーション例
macro_rules! simulate_proc_macro {
    (derive SorcererInfo for $struct_name:ident) => {
        impl $struct_name {
            fn sorcerer_type(&self) -> &'static str {
                stringify!($struct_name)
            }

            fn power_level(&self) -> String {
                format!("{}の呪力レベル", stringify!($struct_name))
            }
        }
    };
}

#[derive(Debug)]
struct MockSorcerer {
    name: String,
    power: i32,
}

simulate_proc_macro!(derive SorcererInfo for MockSorcerer);

fn main() {
    let sorcerer = MockSorcerer {
        name: "五条悟".to_string(),
        power: 3000,
    };

    println!("種類: {}", sorcerer.sorcerer_type());
    println!("レベル: {}", sorcerer.power_level());
}
```

## 実践例 - テスト生成マクロ

```rust
// テスト自動生成マクロ
macro_rules! generate_power_tests {
    (
        $(
            $test_name:ident: $sorcerer:expr, $expected_min:expr, $expected_max:expr
        ),*
    ) => {
        $(
            #[test]
            fn $test_name() {
                let sorcerer = $sorcerer;
                let power = sorcerer.calculate_power();
                assert!(
                    power >= $expected_min && power <= $expected_max,
                    "{}の呪力{}が期待範囲[{}, {}]外です",
                    sorcerer.name, power, $expected_min, $expected_max
                );
                println!("✓ {}: 呪力{} (範囲: {}-{})",
                        sorcerer.name, power, $expected_min, $expected_max);
            }
        )*
    };
}

// ベンチマーク生成マクロ
macro_rules! generate_technique_benchmarks {
    (
        $(
            $bench_name:ident: $technique:expr
        ),*
    ) => {
        $(
            fn $bench_name() {
                use std::time::Instant;

                let iterations = 1000;
                let start = Instant::now();

                for _ in 0..iterations {
                    let _ = $technique();
                }

                let duration = start.elapsed();
                let avg_time = duration.as_nanos() / iterations;

                println!("ベンチマーク {}: 平均実行時間 {}ns",
                        stringify!($bench_name), avg_time);
            }
        )*

        pub fn run_all_benchmarks() {
            println!("=== 術式ベンチマーク実行 ===");
            $(
                $bench_name();
            )*
        }
    };
}

// テスト対象の構造体
#[derive(Debug, Clone)]
struct TestSorcerer {
    name: String,
    base_power: i32,
    multiplier: f32,
}

impl TestSorcerer {
    fn new(name: &str, base_power: i32, multiplier: f32) -> Self {
        TestSorcerer {
            name: name.to_string(),
            base_power,
            multiplier,
        }
    }

    fn calculate_power(&self) -> i32 {
        (self.base_power as f32 * self.multiplier) as i32
    }
}

// テスト用の術式関数
fn blue_technique() -> i32 {
    std::thread::sleep(std::time::Duration::from_nanos(100));
    1000
}

fn red_technique() -> i32 {
    std::thread::sleep(std::time::Duration::from_nanos(150));
    1500
}

fn hollow_technique() -> i32 {
    std::thread::sleep(std::time::Duration::from_nanos(200));
    2000
}

// テストの生成
generate_power_tests! {
    test_gojo_power: TestSorcerer::new("五条悟", 2500, 1.2), 2800, 3200,
    test_yuji_power: TestSorcerer::new("虎杖悠仁", 1000, 1.3), 1200, 1400,
    test_megumi_power: TestSorcerer::new("伏黒恵", 900, 1.1), 950, 1050,
    test_nobara_power: TestSorcerer::new("釘崎野薔薇", 800, 1.2), 900, 1000
}

// ベンチマークの生成
generate_technique_benchmarks! {
    bench_blue: blue_technique,
    bench_red: red_technique,
    bench_hollow: hollow_technique
}

fn main() {
    println!("=== マクロ生成テスト実行 ===");

    // 手動でテストを実行（通常は cargo test で実行）
    test_gojo_power();
    test_yuji_power();
    test_megumi_power();
    test_nobara_power();

    println!();

    // ベンチマーク実行
    run_all_benchmarks();
}
```

## 高度なマクロテクニック

### 条件コンパイル

```rust
// 条件付きコンパイルマクロ
macro_rules! debug_technique {
    ($($arg:tt)*) => {
        #[cfg(debug_assertions)]
        {
            println!("[DEBUG] {}", format!($($arg)*));
        }
    };
}

macro_rules! feature_dependent_code {
    (async: $async_code:block, sync: $sync_code:block) => {
        #[cfg(feature = "async")]
        $async_code

        #[cfg(not(feature = "async"))]
        $sync_code
    };
}

// プラットフォーム依存マクロ
macro_rules! platform_specific {
    () => {
        #[cfg(target_os = "windows")]
        {
            println!("Windows版呪術システム起動");
            "windows_technique"
        }

        #[cfg(target_os = "linux")]
        {
            println!("Linux版呪術システム起動");
            "linux_technique"
        }

        #[cfg(target_os = "macos")]
        {
            println!("macOS版呪術システム起動");
            "macos_technique"
        }
    };
}

fn main() {
    debug_technique!("五条悟の術式を{}で実行", "デバッグモード");

    let platform_technique = platform_specific!();
    println!("プラットフォーム固有術式: {}", platform_technique);

    feature_dependent_code! {
        async: {
            println!("非同期機能が有効です");
        },
        sync: {
            println!("同期機能のみ利用可能です");
        }
    }
}
```

### マクロの再帰と高度なパターン

```rust
// 再帰マクロの例
macro_rules! recursive_technique_chain {
    // 終了条件
    ($technique:expr) => {
        println!("最終術式: {}", $technique);
    };

    // 再帰ケース
    ($first:expr, $($rest:expr),+) => {
        println!("術式実行: {}", $first);
        recursive_technique_chain!($($rest),+);
    };
}

// カウンター付き再帰マクロ
macro_rules! counted_execution {
    (@step $count:expr, $max:expr, $technique:expr) => {
        if $count <= $max {
            println!("実行 {}/{}: {}", $count, $max, $technique);
        }
    };

    ($technique:expr, $times:expr) => {
        counted_execution!(@generate 1, $times, $technique);
    };

    (@generate $current:expr, $max:expr, $technique:expr) => {
        counted_execution!(@step $current, $max, $technique);
        counted_execution!(@increment $current, $max, $technique);
    };

    (@increment $current:expr, $max:expr, $technique:expr) => {
        if $current < $max {
            counted_execution!(@generate $current + 1, $max, $technique);
        }
    };
}

// コンパイル時計算マクロ
macro_rules! compile_time_fibonacci {
    (0) => { 0 };
    (1) => { 1 };
    ($n:expr) => {
        compile_time_fibonacci!($n - 1) + compile_time_fibonacci!($n - 2)
    };
}

// 型レベル計算マクロ
macro_rules! generate_power_levels {
    ($base:expr; $($multiplier:expr),*) => {
        {
            let mut levels = Vec::new();
            let base = $base;
            $(
                levels.push(base * $multiplier);
            )*
            levels
        }
    };
}

fn main() {
    println!("=== 高度なマクロテクニック ===");

    // 再帰チェーン
    println!("\n--- 術式チェーン ---");
    recursive_technique_chain!(
        "無下限術式",
        "術式順転『蒼』",
        "術式反転『赫』",
        "虚式『茈』"
    );

    // 呪力レベル生成
    println!("\n--- 呪力レベル計算 ---");
    let power_levels = generate_power_levels!(1000; 1, 2, 3, 5, 8);
    println!("呪力レベル: {:?}", power_levels);

    // コンパイル時フィボナッチ（小さい値のみ）
    println!("\n--- コンパイル時計算 ---");
    const FIB_5: i32 = compile_time_fibonacci!(5);
    println!("フィボナッチ数列の5番目: {}", FIB_5);
}
```

## マクロのデバッグと最適化

```rust
// マクロ展開の可視化
macro_rules! debug_expand {
    ($($input:tt)*) => {
        {
            println!("マクロ入力: {}", stringify!($($input)*));
            $($input)*
        }
    };
}

// エラーメッセージの改善
macro_rules! safe_division {
    ($a:expr, $b:expr) => {
        {
            if $b == 0 {
                compile_error!("ゼロ除算は禁止されています");
            }
            $a / $b
        }
    };
}

// パフォーマンス計測マクロ
macro_rules! measure_time {
    ($operation:expr) => {
        {
            let start = std::time::Instant::now();
            let result = $operation;
            let duration = start.elapsed();
            println!("実行時間: {:?}", duration);
            result
        }
    };

    ($name:expr, $operation:expr) => {
        {
            let start = std::time::Instant::now();
            let result = $operation;
            let duration = start.elapsed();
            println!("{} 実行時間: {:?}", $name, duration);
            result
        }
    };
}

// 条件付きコンパイルによる最適化
macro_rules! optimized_technique {
    ($technique:expr) => {
        #[cfg(feature = "optimized")]
        {
            // 最適化版
            println!("⚡ 最適化版術式実行: {}", $technique);
            $technique.to_uppercase()
        }

        #[cfg(not(feature = "optimized"))]
        {
            // 通常版
            println!("🔧 通常版術式実行: {}", $technique);
            $technique.to_string()
        }
    };
}

fn expensive_operation() -> i32 {
    std::thread::sleep(std::time::Duration::from_millis(100));
    42
}

fn main() {
    println!("=== マクロのデバッグと最適化 ===");

    // デバッグ展開
    let result = debug_expand! {
        5 + 3 * 2
    };
    println!("計算結果: {}", result);

    // 時間計測
    measure_time!("高コスト術式", expensive_operation());

    // 最適化の例
    let optimized_result = optimized_technique!("無下限術式");
    println!("最適化結果: {}", optimized_result);

    // 安全な除算（コンパイル時エラーチェック）
    // safe_division!(10, 0); // これはコンパイルエラーになる
    let safe_result = safe_division!(10, 2);
    println!("安全な除算: {}", safe_result);
}
```

## まとめ

マクロの力をマスターできたか？重要なポイント：

1. **宣言マクロ** - `macro_rules!`による構文レベルの抽象化
1. **手続きマクロ** - コンパイル時のコード生成
1. **パターンマッチング** - 柔軟な入力処理
1. **再帰と条件分岐** - 複雑なロジックの実装
1. **コンパイル時最適化** - 実行時コストゼロの抽象化

これで現実改変の力を手に入れた。俺の「無量空処」を超えて、**コードという現実そのものを思いのままに操れる**ようになったな。

マクロは究極の抽象化技術だ。適切に使えば、コードの表現力を無限に拡張し、開発効率を飛躍的に向上させる。しかし、強大な力には責任が伴う。濫用は混乱を招くから注意しろ。

次は最強への道を学ぼう。これまでの全ての技術を統合した、真の最強プログラマーになるための最終章だ。

______________________________________________________________________

*「マクロを極めれば、コードの神となれる」*
