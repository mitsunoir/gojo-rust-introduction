# 非同期プログラミング - 無下限の時空間操作

## 非同期の概念 - 時空を操る力

ついに来たな、最終章「無下限呪術編」だ。これまでの全ての技術を習得した君なら、この最高位の奥義を理解できるはずだ。

非同期プログラミングは俺の無下限呪術のように、**時間と空間の概念を操作する**技術だ。複数の処理を同時に進行させ、待機時間を無にして、無限の効率を実現する。

!!! note "五条先生の解説"
    非同期処理では、一つの処理が終わるのを待つ間に、他の処理を進められる。
    例えば、ファイル読み込み中にネットワーク通信を行ったり、複数のAPIを並行して呼び出すことができる。

## 基本的な非同期概念

### Future と async/await

```rust
use std::time::Duration;

// 非同期関数の定義
async fn basic_technique() -> String {
    "基本術式完了".to_string()
}

// 時間のかかる非同期処理
async fn time_consuming_technique(power: i32) -> String {
    // 呪力チャージ時間をシミュレート
    tokio::time::sleep(Duration::from_millis(1000)).await;
    format!("{}の呪力で術式発動完了", power)
}

// エラーが発生する可能性のある非同期関数
async fn risky_technique(success_rate: f32) -> Result<String, String> {
    tokio::time::sleep(Duration::from_millis(500)).await;
    
    if success_rate > 0.7 {
        Ok("高難度術式成功！".to_string())
    } else {
        Err("術式発動失敗 - 呪力不足".to_string())
    }
}

#[tokio::main]
async fn main() {
    println!("=== 基本的な非同期処理 ===");
    
    // 基本的な非同期関数の呼び出し
    let result = basic_technique().await;
    println!("結果: {}", result);
    
    // 時間のかかる処理
    println!("呪力チャージ開始...");
    let power_result = time_consuming_technique(1500).await;
    println!("結果: {}", power_result);
    
    // エラーハンドリング
    match risky_technique(0.8).await {
        Ok(success) => println!("✓ {}", success),
        Err(error) => println!("✗ {}", error),
    }
    
    match risky_technique(0.5).await {
        Ok(success) => println!("✓ {}", success),
        Err(error) => println!("✗ {}", error),
    }
}
```

### 並行処理の基本

```rust
use std::time::{Duration, Instant};
use tokio::time::sleep;

async fn cast_blue() -> String {
    println!("術式順転『青』詠唱開始...");
    sleep(Duration::from_millis(800)).await;
    "術式順転『青』発動完了！".to_string()
}

async fn cast_red() -> String {
    println!("術式反転『赤』詠唱開始...");
    sleep(Duration::from_millis(1200)).await;
    "術式反転『赤』発動完了！".to_string()
}

async fn cast_hollow() -> String {
    println!("虚式『茈』詠唱開始...");
    sleep(Duration::from_millis(1500)).await;
    "虚式『茈』発動完了！".to_string()
}

#[tokio::main]
async fn main() {
    println!("=== 並行処理デモ ===");
    
    // 逐次処理（遅い）
    let start = Instant::now();
    
    println!("\n--- 逐次処理 ---");
    let blue_result = cast_blue().await;
    let red_result = cast_red().await;
    let hollow_result = cast_hollow().await;
    
    println!("{}", blue_result);
    println!("{}", red_result);
    println!("{}", hollow_result);
    
    let sequential_duration = start.elapsed();
    println!("逐次処理時間: {:?}", sequential_duration);
    
    // 並行処理（速い）
    let start = Instant::now();
    
    println!("\n--- 並行処理 ---");
    let (blue_result, red_result, hollow_result) = tokio::join!(
        cast_blue(),
        cast_red(),
        cast_hollow()
    );
    
    println!("{}", blue_result);
    println!("{}", red_result);
    println!("{}", hollow_result);
    
    let concurrent_duration = start.elapsed();
    println!("並行処理時間: {:?}", concurrent_duration);
    
    println!("\n効率向上: {:.1}x", 
             sequential_duration.as_millis() as f64 / concurrent_duration.as_millis() as f64);
}
```

## 高度な並行制御

### select! マクロによる競合処理

```rust
use tokio::time::{sleep, Duration};
use tokio::select;

async fn gojo_technique() -> String {
    sleep(Duration::from_millis(800)).await;
    "五条の無下限術式発動！".to_string()
}

async fn sukuna_technique() -> String {
    sleep(Duration::from_millis(1200)).await;
    "宿儺の斬撃発動！".to_string()
}

async fn timeout_guard(duration: Duration) -> String {
    sleep(duration).await;
    "タイムアウト - 術式発動時間切れ".to_string()
}

#[tokio::main]
async fn main() {
    println!("=== select! マクロデモ ===");
    
    // 最初に完了した処理が勝利
    let result = select! {
        gojo_result = gojo_technique() => {
            format!("勝者: {}", gojo_result)
        },
        sukuna_result = sukuna_technique() => {
            format!("勝者: {}", sukuna_result)
        },
        timeout_result = timeout_guard(Duration::from_millis(1000)) => {
            format!("結果: {}", timeout_result)
        }
    };
    
    println!("{}", result);
    
    // 条件付きselect
    let mut counter = 0;
    
    loop {
        select! {
            _ = sleep(Duration::from_millis(200)) => {
                counter += 1;
                println!("カウンター: {}", counter);
                
                if counter >= 5 {
                    println!("カウンター完了");
                    break;
                }
            },
            _ = sleep(Duration::from_millis(1500)), if counter < 3 => {
                println!("早期終了条件発動");
                break;
            }
        }
    }
}
```

### Mutex とチャンネルによる状態管理

```rust
use std::sync::Arc;
use tokio::sync::{Mutex, mpsc};
use tokio::time::{sleep, Duration};
use std::collections::HashMap;

// 共有状態を持つ呪術師管理システム
#[derive(Debug, Clone)]
struct Sorcerer {
    name: String,
    power: i32,
    is_fighting: bool,
}

impl Sorcerer {
    fn new(name: &str, power: i32) -> Self {
        Sorcerer {
            name: name.to_string(),
            power,
            is_fighting: false,
        }
    }
    
    fn start_fighting(&mut self) {
        self.is_fighting = true;
    }
    
    fn stop_fighting(&mut self) {
        self.is_fighting = false;
    }
    
    fn gain_experience(&mut self, exp: i32) {
        self.power += exp;
    }
}

// 戦闘システム
struct BattleSystem {
    sorcerers: Arc<Mutex<HashMap<String, Sorcerer>>>,
}

impl BattleSystem {
    fn new() -> Self {
        BattleSystem {
            sorcerers: Arc::new(Mutex::new(HashMap::new())),
        }
    }
    
    async fn add_sorcerer(&self, name: &str, power: i32) {
        let mut sorcerers = self.sorcerers.lock().await;
        sorcerers.insert(name.to_string(), Sorcerer::new(name, power));
        println!("{}が戦場に参戦（呪力: {}）", name, power);
    }
    
    async fn start_battle(&self, name: &str) -> Result<(), String> {
        let mut sorcerers = self.sorcerers.lock().await;
        
        match sorcerers.get_mut(name) {
            Some(sorcerer) => {
                if sorcerer.is_fighting {
                    Err(format!("{}は既に戦闘中です", name))
                } else {
                    sorcerer.start_fighting();
                    println!("{}が戦闘を開始", name);
                    Ok(())
                }
            },
            None => Err(format!("{}が見つかりません", name)),
        }
    }
    
    async fn end_battle(&self, name: &str, exp_gained: i32) -> Result<(), String> {
        let mut sorcerers = self.sorcerers.lock().await;
        
        match sorcerers.get_mut(name) {
            Some(sorcerer) => {
                sorcerer.stop_fighting();
                sorcerer.gain_experience(exp_gained);
                println!("{}が戦闘終了（経験値+{}, 現在呪力: {}）", 
                        name, exp_gained, sorcerer.power);
                Ok(())
            },
            None => Err(format!("{}が見つかりません", name)),
        }
    }
    
    async fn get_status(&self, name: &str) -> Option<Sorcerer> {
        let sorcerers = self.sorcerers.lock().await;
        sorcerers.get(name).cloned()
    }
    
    async fn list_all(&self) -> Vec<Sorcerer> {
        let sorcerers = self.sorcerers.lock().await;
        sorcerers.values().cloned().collect()
    }
}

// 戦闘シミュレーション
async fn simulate_battle(
    system: Arc<BattleSystem>, 
    name: String, 
    duration: Duration,
    sender: mpsc::Sender<String>
) {
    // 戦闘開始
    if let Err(e) = system.start_battle(&name).await {
        let _ = sender.send(format!("戦闘開始エラー: {}", e)).await;
        return;
    }
    
    // 戦闘時間
    sleep(duration).await;
    
    // 戦闘終了と経験値獲得
    let exp = (duration.as_millis() / 10) as i32;
    if let Err(e) = system.end_battle(&name, exp).await {
        let _ = sender.send(format!("戦闘終了エラー: {}", e)).await;
        return;
    }
    
    let _ = sender.send(format!("{}の戦闘完了", name)).await;
}

#[tokio::main]
async fn main() {
    println!("=== 並行戦闘管理システム ===");
    
    let system = Arc::new(BattleSystem::new());
    
    // 呪術師を追加
    system.add_sorcerer("五条悟", 3000).await;
    system.add_sorcerer("虎杖悠仁", 1200).await;
    system.add_sorcerer("伏黒恵", 1000).await;
    system.add_sorcerer("釘崎野薔薇", 900).await;
    
    // チャンネルの作成
    let (tx, mut rx) = mpsc::channel::<String>(32);
    
    // 並行戦闘の開始
    let battles = vec![
        (system.clone(), "五条悟".to_string(), Duration::from_millis(1000), tx.clone()),
        (system.clone(), "虎杖悠仁".to_string(), Duration::from_millis(1500), tx.clone()),
        (system.clone(), "伏黒恵".to_string(), Duration::from_millis(800), tx.clone()),
        (system.clone(), "釘崎野薔薇".to_string(), Duration::from_millis(1200), tx.clone()),
    ];
    
    // 全ての戦闘を並行実行
    for (sys, name, duration, sender) in battles {
        tokio::spawn(simulate_battle(sys, name, duration, sender));
    }
    
    // チャンネルを閉じる
    drop(tx);
    
    // 戦闘結果を受信
    while let Some(message) = rx.recv().await {
        println!("📨 {}", message);
    }
    
    // 最終状態を表示
    println!("\n=== 最終戦闘結果 ===");
    let all_sorcerers = system.list_all().await;
    for sorcerer in all_sorcerers {
        println!("{}: 呪力{} (戦闘中: {})", 
                sorcerer.name, sorcerer.power, sorcerer.is_fighting);
    }
}
```

## 実践例 - 非同期Web APIクライアント

```rust
use std::collections::HashMap;
use std::time::Duration;
use tokio::time::sleep;
use serde::{Deserialize, Serialize};

// モックAPIレスポンス
#[derive(Debug, Serialize, Deserialize, Clone)]
struct CurseData {
    id: u32,
    name: String,
    grade: String,
    power_level: i32,
    location: String,
}

#[derive(Debug, Serialize, Deserialize)]
struct ApiResponse<T> {
    success: bool,
    data: Option<T>,
    error: Option<String>,
}

// 非同期APIクライアント
struct CurseApiClient {
    base_url: String,
    timeout: Duration,
}

impl CurseApiClient {
    fn new(base_url: &str) -> Self {
        CurseApiClient {
            base_url: base_url.to_string(),
            timeout: Duration::from_secs(5),
        }
    }
    
    // モックAPIコール - 単一の呪霊データ取得
    async fn get_curse(&self, id: u32) -> Result<CurseData, String> {
        println!("🔍 呪霊ID{}のデータを取得中...", id);
        
        // ネットワーク遅延をシミュレート
        sleep(Duration::from_millis(300 + (id * 50) as u64)).await;
        
        // モックデータ
        let mock_data = match id {
            1 => CurseData {
                id: 1,
                name: "特級呪霊『リカ』".to_string(),
                grade: "特級".to_string(),
                power_level: 2500,
                location: "東京都立呪術高等専門学校".to_string(),
            },
            2 => CurseData {
                id: 2,
                name: "花御".to_string(),
                grade: "特級".to_string(),
                power_level: 2200,
                location: "渋谷".to_string(),
            },
            3 => CurseData {
                id: 3,
                name: "漏瑚".to_string(),
                grade: "特級".to_string(),
                power_level: 2000,
                location: "富士山".to_string(),
            },
            _ => return Err(format!("呪霊ID{}は存在しません", id)),
        };
        
        Ok(mock_data)
    }
    
    // 複数の呪霊データを並行取得
    async fn get_multiple_curses(&self, ids: Vec<u32>) -> HashMap<u32, Result<CurseData, String>> {
        let mut results = HashMap::new();
        let futures: Vec<_> = ids.iter().map(|&id| async move {
            (id, self.get_curse(id).await)
        }).collect();
        
        // 全ての非同期処理を並行実行
        let completed = futures::future::join_all(futures).await;
        
        for (id, result) in completed {
            results.insert(id, result);
        }
        
        results
    }
    
    // タイムアウト付きAPI呼び出し
    async fn get_curse_with_timeout(&self, id: u32) -> Result<CurseData, String> {
        match tokio::time::timeout(self.timeout, self.get_curse(id)).await {
            Ok(result) => result,
            Err(_) => Err(format!("呪霊ID{}の取得がタイムアウトしました", id)),
        }
    }
    
    // 呪霊の脅威レベル分析
    async fn analyze_threat_level(&self, ids: Vec<u32>) -> Result<String, String> {
        let results = self.get_multiple_curses(ids).await;
        
        let mut total_power = 0;
        let mut curse_count = 0;
        let mut special_grade_count = 0;
        let mut locations = Vec::new();
        
        for (id, result) in results {
            match result {
                Ok(curse) => {
                    total_power += curse.power_level;
                    curse_count += 1;
                    
                    if curse.grade == "特級" {
                        special_grade_count += 1;
                    }
                    
                    locations.push(curse.location);
                    println!("✓ {}: {} (呪力: {})", id, curse.name, curse.power_level);
                },
                Err(e) => {
                    println!("✗ ID {}: {}", id, e);
                }
            }
        }
        
        if curse_count == 0 {
            return Err("有効な呪霊データが取得できませんでした".to_string());
        }
        
        let average_power = total_power / curse_count;
        let threat_level = match average_power {
            0..=500 => "低",
            501..=1500 => "中",
            1501..=2000 => "高",
            _ => "極高",
        };
        
        Ok(format!(
            "=== 脅威レベル分析結果 ===\n\
            分析対象: {}体\n\
            特級呪霊: {}体\n\
            平均呪力: {}\n\
            脅威レベル: {}\n\
            出現地域: {:?}",
            curse_count, special_grade_count, average_power, threat_level, locations
        ))
    }
}

// ストリーミング処理
async fn stream_curse_monitoring(client: &CurseApiClient) {
    println!("\n=== リアルタイム呪霊監視開始 ===");
    
    let curse_ids = vec![1, 2, 3];
    
    for round in 1..=3 {
        println!("\n--- 監視ラウンド {} ---", round);
        
        let futures: Vec<_> = curse_ids.iter().map(|&id| async move {
            let start = std::time::Instant::now();
            let result = client.get_curse_with_timeout(id).await;
            let duration = start.elapsed();
            (id, result, duration)
        }).collect();
        
        // 結果をストリーミング形式で表示
        for (id, result, duration) in futures::future::join_all(futures).await {
            match result {
                Ok(curse) => {
                    println!("📊 [{}ms] {}: {} (脅威度: {})", 
                            duration.as_millis(), curse.name, curse.location, curse.power_level);
                },
                Err(e) => {
                    println!("❌ [{}ms] ID {}: {}", duration.as_millis(), id, e);
                }
            }
        }
        
        // 次のラウンドまで待機
        if round < 3 {
            sleep(Duration::from_millis(1000)).await;
        }
    }
}

#[tokio::main]
async fn main() {
    println!("=== 非同期呪霊情報管理システム ===");
    
    let client = CurseApiClient::new("https://api.jujutsu-kaisen.com");
    
    // 単一の呪霊データ取得
    println!("\n--- 単一データ取得 ---");
    match client.get_curse(1).await {
        Ok(curse) => println!("取得成功: {:?}", curse),
        Err(e) => println!("取得失敗: {}", e),
    }
    
    // 複数データの並行取得
    println!("\n--- 複数データ並行取得 ---");
    let start = std::time::Instant::now();
    let results = client.get_multiple_curses(vec![1, 2, 3, 4]).await;
    let duration = start.elapsed();
    
    println!("取得時間: {:?}", duration);
    for (id, result) in results {
        match result {
            Ok(curse) => println!("✓ ID {}: {}", id, curse.name),
            Err(e) => println!("✗ ID {}: {}", id, e),
        }
    }
    
    // 脅威レベル分析
    println!("\n--- 脅威レベル分析 ---");
    match client.analyze_threat_level(vec![1, 2, 3]).await {
        Ok(analysis) => println!("{}", analysis),
        Err(e) => println!("分析エラー: {}", e),
    }
    
    // ストリーミング監視
    stream_curse_monitoring(&client).await;
}

// 外部クレート依存の例（Cargo.tomlに追加が必要）
/*
[dependencies]
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
futures = "0.3"
*/
```

## エラーハンドリングと回復戦略

```rust
use std::time::Duration;
use tokio::time::{sleep, timeout};

#[derive(Debug)]
enum TechniqueError {
    PowerInsufficient(i32, i32), // (required, available)
    ConnectionLost,
    Timeout,
    SystemOverload,
}

impl std::fmt::Display for TechniqueError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            TechniqueError::PowerInsufficient(req, avail) => {
                write!(f, "呪力不足: 必要{}, 利用可能{}", req, avail)
            },
            TechniqueError::ConnectionLost => write!(f, "呪力の流れが途絶"),
            TechniqueError::Timeout => write!(f, "術式発動タイムアウト"),
            TechniqueError::SystemOverload => write!(f, "術式システム過負荷"),
        }
    }
}

impl std::error::Error for TechniqueError {}

// 回復可能な非同期処理
async fn unstable_technique(power: i32, attempt: u32) -> Result<String, TechniqueError> {
    // 遅延をシミュレート
    let delay = Duration::from_millis(500 + (attempt * 200) as u64);
    sleep(delay).await;
    
    // 失敗パターンをシミュレート
    match attempt {
        1 => Err(TechniqueError::PowerInsufficient(1500, power)),
        2 => Err(TechniqueError::ConnectionLost),
        3 => Err(TechniqueError::SystemOverload),
        _ => Ok(format!("術式成功！威力: {}", power * attempt as i32)),
    }
}

// リトライ戦略付き非同期関数
async fn reliable_technique_with_retry(
    power: i32, 
    max_retries: u32,
    timeout_duration: Duration
) -> Result<String, TechniqueError> {
    for attempt in 1..=max_retries {
        println!("術式試行 {}/{}", attempt, max_retries);
        
        let result = timeout(timeout_duration, unstable_technique(power, attempt)).await;
        
        match result {
            Ok(Ok(success)) => {
                println!("✓ 試行{}で成功: {}", attempt, success);
                return Ok(success);
            },
            Ok(Err(e)) => {
                println!("✗ 試行{}で失敗: {}", attempt, e);
                
                // エラーの種類に応じて待機時間を調整
                let wait_time = match e {
                    TechniqueError::PowerInsufficient(_, _) => Duration::from_millis(100),
                    TechniqueError::ConnectionLost => Duration::from_millis(500),
                    TechniqueError::SystemOverload => Duration::from_millis(1000),
                    TechniqueError::Timeout => Duration::from_millis(200),
                };
                
                if attempt < max_retries {
                    println!("{}ms後に再試行...", wait_time.as_millis());
                    sleep(wait_time).await;
                } else {
                    return Err(e);
                }
            },
            Err(_) => {
                println!("✗ 試行{}でタイムアウト", attempt);
                if attempt == max_retries {
                    return Err(TechniqueError::Timeout);
                }
                sleep(Duration::from_millis(300)).await;
            }
        }
    }
    
    Err(TechniqueError::Timeout)
}

// サーキットブレーカーパターン
#[derive(Debug)]
struct CircuitBreaker {
    failure_count: u32,
    failure_threshold: u32,
    recovery_timeout: Duration,
    last_failure_time: Option<std::time::Instant>,
    state: CircuitState,
}

#[derive(Debug, PartialEq)]
enum CircuitState {
    Closed,    // 正常状態
    Open,      // 故障状態
    HalfOpen,  // 回復試行状態
}

impl CircuitBreaker {
    fn new(failure_threshold: u32, recovery_timeout: Duration) -> Self {
        CircuitBreaker {
            failure_count: 0,
            failure_threshold,
            recovery_timeout,
            last_failure_time: None,
            state: CircuitState::Closed,
        }
    }
    
    async fn call<F, T, E>(&mut self, operation: F) -> Result<T, String>
    where
        F: std::future::Future<Output = Result<T, E>>,
        E: std::fmt::Display,
    {
        match self.state {
            CircuitState::Open => {
                if let Some(last_failure) = self.last_failure_time {
                    if last_failure.elapsed() >= self.recovery_timeout {
                        println!("サーキットブレーカー: 回復試行状態に移行");
                        self.state = CircuitState::HalfOpen;
                    } else {
                        return Err("サーキットブレーカー開放中 - 処理拒否".to_string());
                    }
                }
            },
            _ => {}
        }
        
        match operation.await {
            Ok(result) => {
                // 成功時の処理
                if self.state == CircuitState::HalfOpen {
                    println!("サーキットブレーカー: 正常状態に復帰");
                    self.state = CircuitState::Closed;
                    self.failure_count = 0;
                }
                Ok(result)
            },
            Err(e) => {
                // 失敗時の処理
                self.failure_count += 1;
                self.last_failure_time = Some(std::time::Instant::now());
                
                if self.failure_count >= self.failure_threshold {
                    println!("サーキットブレーカー: 故障状態に移行");
                    self.state = CircuitState::Open;
                }
                
                Err(format!("処理失敗: {}", e))
            }
        }
    }
}

#[tokio::main]
async fn main() {
    println!("=== 高度なエラーハンドリング ===");
    
    // リトライ戦略のテスト
    println!("\n--- リトライ戦略テスト ---");
    match reliable_technique_with_retry(
        1000, 
        4, 
        Duration::from_secs(2)
    ).await {
        Ok(result) => println!("最終成功: {}", result),
        Err(e) => println!("最終失敗: {}", e),
    }
    
    // サーキットブレーカーのテスト
    println!("\n--- サーキットブレーカーテスト ---");
    let mut breaker = CircuitBreaker::new(2, Duration::from_millis(1000));
    
    // 故障を発生させる
    for i in 1..=5 {
        let result = breaker.call(async {
            if i <= 3 {
                Err("模擬故障")
            } else {
                Ok(format!("成功 {}", i))
            }
        }).await;
        
        println!("呼び出し {}: {:?} (状態: {:?})", i, result, breaker.state);
        sleep(Duration::from_millis(500)).await;
    }
    
    // 回復を待つ
    println!("\n回復タイムアウトを待機中...");
    sleep(Duration::from_millis(600)).await;
    
    // 回復後のテスト
    for i in 6..=7 {
        let result = breaker.call(async {
            Ok(format!("回復後成功 {}", i))
        }).await;
        
        println!("回復テスト {}: {:?} (状態: {:?})", i, result, breaker.state);
    }
}
```

## まとめ

非同期プログラミングの基礎をマスターできたか？重要なポイント：

1. **async/await** - 非同期関数の定義と実行
2. **並行処理** - join!とselect!による効率的な処理
3. **共有状態** - MutexとChannelによる安全な状態管理
4. **エラーハンドリング** - タイムアウトとリトライ戦略
5. **実践パターン** - サーキットブレーカーやストリーミング処理

これで時空間を操る力を手に入れた。俺の無下限術式のように、時間の制約を超越し、複数の処理を同時に制御できるようになったな。

次はマクロについて学ぼう。コード自体を生成する、究極の抽象化技術だ。

---

*「非同期を極めれば、時間さえも思いのままにできる」*