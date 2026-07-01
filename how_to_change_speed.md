# Zeus Car 速度調整マニュアル（App / リモコン）

SunFounder Zeus Car の走行アプリ（SunFounder Controller）と IR リモコンについて、**移動速度**と**旋回速度**を、それぞれ独立して調整するための手順書です。

このマニュアルでは、次の 4 つを別々に設定できるようにします。

| 調整対象 | 操作方法 | 速度の性質 |
|---|---|---|
| 移動（前後左右）速度 | App | ジョイスティックの倒し量に応じた可変。その**上限**を設定 |
| 移動（前後左右）速度 | リモコン | ボタンを押すたびに一定（固定値） |
| 旋回速度 | App | 方位ジョイスティックによる旋回。その**上限**を設定 |
| 旋回速度 | リモコン | ドリフト旋回などの旋回。その**上限**を設定 |

> **前提**：Zeus Car は「Arduino UNO（走行制御）」と「ESP32-CAM（カメラ・WiFi）」の 2 つの頭脳で動きます。速度に関わるパラメータはすべて **UNO 側のプログラム**にあり、書き換えは PC の「Arduino IDE」というソフトで行います。ESP32-CAM 側は今回いっさい触りません。

---

# 第1部　準備と書き込みに必要な手順

Arduino をまったく触ったことがない方でも進められるよう、順番どおりに書いています。このマニュアルでは、**すでに速度が調整済みのコードを SharePoint から入手して書き込む方法**を基本とします。速度を自分好みに変えたい場合のみ、書き込みの前に「第2部：速度を適合させる」を行います（任意）。

## ステップ0：全体の流れ

```
[1] PCにArduino IDEを入れる
        ↓
[2] 必要なライブラリを入れる
        ↓
[3] 調整済みプログラムを入手する
        ↓
   （必要な人だけ）第2部：速度を適合させる（Zeus_Car.ino の数値変更）
        ↓
[4] Zeus Carに書き込む
        ↓
[5] 動作確認
```

> 配布される調整済みコードには、速度・旋回の設定がすでに入っています。標準の速度で問題なければ第2部は不要です。自分で微調整したい場合のみ第2部を行います。

## ステップ1：準備するもの

**もの**
- Windows または Mac の PC
- Zeus Car 本体（組み立て済み）
- Zeus Car 付属の **青い USB ケーブル**

**ソフト**
- Arduino IDE（無料）

## ステップ2：Arduino IDE を入れる

1. ブラウザで「Arduino IDE ダウンロード」と検索し、公式サイト（arduino.cc）から **Arduino IDE 2.x** をダウンロードします。
2. インストーラーを実行し、画面の指示に従ってインストールします。
3. インストールが終わったら Arduino IDE を起動します。

> Zeus Car は「Arduino Uno」というボードを使います。Arduino IDE には Uno 用の設定が最初から入っているので、追加のボード設定は基本的に不要です。

> 📖 **公式の画像付き手順**（Windows / macOS / Linux 別）：[Arduino IDE 2.0のダウンロードとインストール](https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/install_arduino_ide.html) / [Arduino IDEの紹介（画面の見方）](https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/introduce_ide.html)

## ステップ3：必要なライブラリを入れる

Zeus Car のプログラムは、3 つの追加ライブラリを使います。

1. Arduino IDE 左側のアイコンから **ライブラリマネージャ**（本が積まれたアイコン）を開きます。
2. 検索欄に以下を 1 つずつ入力し、それぞれ **インストール** を押します。
   - `IRLremote`
   - `SoftPWM`
   - `ArduinoJson`

> 📖 **公式の画像付き手順**：[必要なライブラリをインストールする](https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/add_libraries.html)

## ステップ4：調整済みプログラムを入手する

すでに速度が調整されたコードを受け取ります。**パラメータ編集は不要**で、そのまま書き込みに進めます。入手方法は次の 2 つのどちらでも構いません。

### 方法1：SharePoint（SPO）から ZIP をダウンロードする

1. ブラウザで共有リンク **`<SPO共有リンク>`** から ZIP ファイルを **ダウンロード** します。
2. ダウンロードした ZIP を解凍します。
3. 解凍したフォルダ内の `Zeus_Car` フォルダにある **`Zeus_Car.ino`** をダブルクリックして Arduino IDE で開きます（同じフォルダ内の他のファイルも自動で一緒に開きます）。

### 方法2：GitHub リポジトリから git clone する

Git を使える方は、以下のリポジトリから直接クローンして入手することもできます。

```
git clone https://github.com/amane513/zeus-car
```

- Git を入れていない場合は、リポジトリのページ（`https://github.com/amane513/zeus-car`）を開き、緑色の **「Code」ボタン → 「Download ZIP」** からダウンロードして解凍しても構いません（内容は clone と同じです）。
- クローン（または解凍）したフォルダ内の `Zeus_Car` フォルダにある **`Zeus_Car.ino`** をダブルクリックして Arduino IDE で開きます（同じフォルダ内の他のファイルも自動で一緒に開きます）。

---

どちらの方法でも、**→ そのまま書き込む場合は第3部（書き込み）へ進みます。速度を自分で微調整したい場合のみ、次の第2部へ進んでください。**

> 公式コードを一から自分で入手して、速度・旋回の独立化まで自力で実装したい上級者は、Appendix G を参照してください（配布コードを使う場合は不要です）。

---

# 第2部　速度を適合させたい場合（任意・`Zeus_Car.ino` のみ編集）

**この第2部は、配布された速度で問題ない場合は実施不要です。** 自分の環境や好みに合わせて速度を微調整（適合）したい場合のみ行ってください。標準のままでよければ、第2部は飛ばして第3部（書き込み）に進んでください。

配布コードには、**旋回を App／リモコンで独立して設定できるようにする改造がすでに入っています**。そのため、適合の際に編集するファイルは **`Zeus_Car.ino` の 1 つだけ**です。以下の 4 つのマクロの数値を書き換えるだけで、4 つの速度をすべて調整できます。

数値を大きくすると速く、小さくすると遅くなります。値の範囲は原則 **0〜100** です。

## 2-1. 移動速度（並進）を調整する

### リモコンの移動速度

`Zeus_Car.ino` を開き、`IR_REMOTE_POWER` の数値を変更します。

```cpp
// 変更前
#define IR_REMOTE_POWER 80

// 変更後（例：ゆっくりめ）
#define IR_REMOTE_POWER 55
```

### App の移動速度（最大速度の上限）

App はジョイスティックの倒し量で速度が変わります。その**最大値の上限**を `APP_REMOTE_POWER` で決めます。`Zeus_Car.ino` の数値を変更します。

```cpp
// 変更前
#define APP_REMOTE_POWER 70

// 変更後（例：ゆっくりめ）
#define APP_REMOTE_POWER 55
```

これで App はジョイスティックの操作感を保ったまま、最大速度だけが `APP_REMOTE_POWER` に抑えられます。

## 2-2. 旋回速度を調整する

旋回速度も、App とリモコンそれぞれのマクロの数値を変えるだけです（独立して設定できるようにする改造は配布コードに済んでいます）。

### リモコンの旋回速度

```cpp
// 変更前
#define IR_ROTATE_LIMIT 100

// 変更後（例）
#define IR_ROTATE_LIMIT 80
```

### App の旋回速度

```cpp
// 変更前
#define APP_ROTATE_LIMIT 60

// 変更後（例：ゆっくりめ）
#define APP_ROTATE_LIMIT 45
```

## 2-3. パラメータ早見表

**適合時に触るのは、すべて `Zeus_Car.ino` の 4 つの数字**だけです。

| 調整したいもの | パラメータ名 | ファイル | 推奨初期値 | 備考 |
|---|---|---|---|---|
| リモコンの移動速度 | `IR_REMOTE_POWER` | `Zeus_Car.ino` | 55〜80 | 押している間の一定速度 |
| App の移動速度（上限） | `APP_REMOTE_POWER` | `Zeus_Car.ino` | 70 | ジョイスティック最大時の速度 |
| リモコンの旋回速度 | `IR_ROTATE_LIMIT` | `Zeus_Car.ino` | 100 | 0〜100 |
| App の旋回速度 | `APP_ROTATE_LIMIT` | `Zeus_Car.ino` | 60 | 0〜100 |

> **値の目安**：移動速度を 40 より小さくすると発進しづらくなることがあります。小さめにしたいときは実機で確認しながら少しずつ下げてください。旋回速度（`APP_ROTATE_LIMIT`）を小さくしすぎると直進の安定性に影響することがあります（詳細は Appendix C）。

編集が終わったら、**`Zeus_Car.ino` を保存**して第3部へ進みます。

---

# 第3部　Zeus Car への書き込み

**順番が重要**なので、そのとおりに行ってください。

## 3-1. PC と接続する

1. Zeus Car の電源を入れます。
2. 青い USB ケーブルで Zeus Car（Arduino Uno 基板）と PC をつなぎます。

## 3-2. ★重要：ESP32-CAM を外す

**書き込みの前に、必ず ESP32-CAM モジュールを基板から抜いてください。**
ESP32-CAM と Arduino は同じ通信線（RX/TX）を共有しているため、挿したままだと書き込みが失敗します。

## 3-3. ボードとポートを選ぶ

1. 上部の **「Select Board」** を押します。
2. ドロップダウン一番下の 「Select other board and port…」 をクリックします。
3. ダイアログが開きます。左側の BOARDS の検索欄に Uno と入力し、「Arduino Uno」 を選びます。
4. 接続先（ポート）の一覧から Zeus Car のポートを選びます。Windows では `COM3` のような表示、Mac では `/dev/cu.usbserial…` のような表示になります。どれか分からない場合は、USB を抜き差しして新しく現れる項目が Zeus Car です。
5. ボードとポートの両方を選ぶと OK が押せるようになるので、OK を押します。

## 3-4. 書き込む

1. Arduino IDE 左上の **「✓（Veryfy）」ボタン**を押します。
2. Arduino IDE 左上の **「→（Upload）」ボタン**を押します。
3. 自動的にコンパイル（変換）→ 書き込みが行われます。下部に **「Done uploading」** と出れば成功です。

> **うまくいかないとき**：`avrdude ... not in sync` のようなエラーが出たら、ほぼ ESP32-CAM の抜き忘れです。ESP32-CAM を外してから、もう一度書き込んでください。

## 3-5. 反映して動かす

1. ESP32-CAM を基板に挿し直します。
2. 基板の **モードスイッチを「Run」側**に切り替えます。
3. **Reset ボタン**を押して再起動します。

これで、変更した速度が反映された状態で Zeus Car が起動します。

## 3-6. 動作確認

- **リモコン**：方向ボタンで、設定した速度で動くか確認します。
- **App**：SunFounder Controller アプリを起動し、`Zeus_Car`（パスワード `12345678`）に接続。ジョイスティックの移動速度、方位ジョイスティックの旋回速度を確認します。

思ったより速い／遅い場合は、第2部の該当パラメータを再調整し、第3部の書き込みをやり直してください。

---

# Appendix（背景知識）

ここからは「なぜそうするのか」を理解したい人向けの補足です。設定だけなら読まなくても構いません。

## A. Zeus Car の構成

Zeus Car は 2 つのマイコンで動いています。

- **Arduino UNO**：モーター、コンパス、各種センサを制御する「走行の頭脳」。今回調整する速度パラメータはすべてこちら側にあります。
- **ESP32-CAM**：カメラ映像と WiFi（App 通信・FPV）を担当。今回は変更しません。

App からの操作もリモコンからの操作も、最終的には UNO 側のプログラムがモーター出力を決めています。

## B. なぜ書き込み時に ESP32-CAM を外すのか

ESP32-CAM と Arduino UNO は、データを送受信する信号線（RX / TX）を物理的に共有しています。書き込み中は PC から UNO へこの線を使ってプログラムが送られますが、ESP32-CAM が挿さっていると信号がぶつかり、書き込みが妨げられます。そのため、書き込みの間だけ ESP32-CAM を外す必要があります。

## C. 速度制御の仕組み

**移動（並進）**
入力される速度値（0〜100）は、最終的に `car_control.cpp` の `carSetMotors()` 内でモーターの PWM 出力へ変換されます。

```cpp
newPower[i] = map(abs(power[i]), 0, 100, MOTOR_POWER_MIN, 255);
```

- **App** は、この入力値がジョイスティックの倒し量そのもの（可変）。だから「入力に上限をかける（`map(..., 0, APP_REMOTE_POWER)`）」ことで最大速度を制限します。
- **リモコン** は、キーを押すたびに `remotePower = IR_REMOTE_POWER;` と固定値が入ります。だから `IR_REMOTE_POWER` 自体が速度そのものです。

**旋回**
App の方位ジョイスティックも、リモコンのドリフト旋回も、`car_control.cpp` の `carMoveFieldCentric()` にある「目標方位に向ける計算（PID 制御）」を通ります。この計算結果 `rot`（旋回量）は、元々 `±100` に制限されていました。

```cpp
rot += max(-100, min(100, offset));   // ← この ±100 が旋回速度の上限
```

この上限値こそが「旋回の勢い」を決めています。配布コード（および Appendix G の改造）では、この固定の `±100` を **引数 `rotLimit` に置き換え**、App とリモコンで別々の上限を渡せるようにしています。これにより、共通だった旋回速度を操作方法ごとに独立させています。

> **⚠️ `APP_ROTATE_LIMIT`／`IR_ROTATE_LIMIT` は「意図的な旋回」だけでなく、直進中の姿勢保持にも効きます**
>
> この `rot`（旋回量）は、方向ジョイスティックやドリフトで「向きを変えたいとき」だけに出るものではありません。`carMoveFieldCentric()` は毎ループ、コンパスで測った現在方位と目標方位のズレ（`error`）を PID で補正しており、**まっすぐ移動している間も、車体が横に流れないよう小さな `rot` を出し続けています**。
>
> つまり `rotLimit` は「その瞬間に許されるヨー（旋回）補正の最大権限」であり、意図的な旋回速度と、直進中の姿勢保持の強さの**両方の上限**になっています。そのため `APP_ROTATE_LIMIT` を小さくしすぎると、旋回がゆっくりになる代わりに、**速く移動したときに車体が向きを保ちにくくなる（まっすぐ走りにくくなる）**ことがあります。旋回をゆっくりにしたい場合は、実機で直進の安定性も確認しながら値を決めてください。

なお、旋回全体をなめらかにしたいだけであれば、`car_control.cpp` の PID ゲイン `KP`（既定 0.8）や `KD`（既定 20.0）を下げる方法もあります。停止時の細かな振動が気になる場合にも有効です。

## D. リモコンが「固定速度」である理由

IR リモコンは物理ボタンで、ジョイスティックのような「どれだけ倒したか」という連続値を送れません。ボタンは押した／押さないの 2 択です。そのためプログラム側では、キーが押されるたびに一律 `IR_REMOTE_POWER`（既定 80）を速度として設定しています。結果として、どの方向ボタンを押しても常に同じ速度になります（＝最大の 100 ではなく、80 固定）。速度を変えたいときは `IR_REMOTE_POWER` の数値を変更します。

## E. App のウィジェットとコード内リージョンの対応

`REGION_K` や `REGION_Q` は、SunFounder Controller アプリ上の各ウィジェット（操作パネル）に対応しています。公式の [APP Control Plus](https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/ar_app_control_plus.html) の配置に沿って、今回の調整に関係するものを整理すると次のとおりです。

| リージョン | アプリ上のウィジェット | 役割 | 本マニュアルとの関係 |
|---|---|---|---|
| `REGION_K` | Move in All Directions | 全方向移動ジョイスティック | **App の移動速度**（`APP_REMOTE_POWER` で上限を設定） |
| `REGION_Q` | Control the Direction | 方向（車体の向き）ジョイスティック | **App の旋回**（`APP_ROTATE_LIMIT` で速度を設定） |
| `REGION_J` | Drift Enable | ドリフトの ON/OFF スイッチ | 旋回の挙動に影響 |
| `REGION_G` | （原点リセット） | 車体の基準方位をリセット | 旋回の基準に影響 |

つまり `REGION_K` に上限をかける編集は「移動ジョイスティック」の最大速度を、`REGION_Q` から入る旋回に `APP_ROTATE_LIMIT` を効かせる編集は「方向ジョイスティック」の旋回速度を、それぞれ制御していることになります。

## F. 参考：その他モードの速度（今回は未使用）

App／リモコン以外にも、Zeus Car には自律モードがあり、それぞれ速度マクロを持っています（`Zeus_Car.ino`）。今回の用途では触りませんが、参考までに。

| マクロ | 既定 | 用途 |
|---|---|---|
| `LINE_TRACK_POWER` | 100 | ライントレース |
| `OBSTACLE_AVOID_POWER` | 90 | 障害物回避 |
| `OBSTACLE_FOLLOW_POWER` | 90 | 追従 |
| `OBSTACLE_FOLLOW_ROTATE_POWER` | 50 | 追従時の旋回 |
| `SPEECH_REMOTE_POWER` | 60 | 音声操作 |

## G. 上級者向け：公式コードを一から入手して自分で編集する（旧・方法A）

第1部では、速度・旋回の設定がすでに入った**調整済みコード（SPO 配布）**を使いました。ここでは、SunFounder **公式コード**を入手して、速度パラメータと「旋回の独立化」を**自分で一から実装する**方法を説明します。配布コードを使う場合は、このセクションは不要です。

この方法では、`Zeus_Car.ino` に加えて `car_control.h` と `car_control.cpp` も編集します。作業後は、日常の速度調整は第2部と同じく `Zeus_Car.ino` の 4 つの数字だけで行えるようになります。

### G-1. 公式コードを入手する

1. ブラウザで SunFounder 公式リポジトリ `https://github.com/sunfounder/zeus-car` を開きます。
2. 緑色の **「Code」ボタン → 「Download ZIP」** を押して ZIP をダウンロードします。
3. ダウンロードした ZIP を解凍します。
4. 解凍したフォルダ内の `Zeus_Car` フォルダにある **`Zeus_Car.ino`** をダブルクリックして Arduino IDE で開きます（同じフォルダ内の他のファイルも自動で一緒に開きます）。

> 📖 **公式の画像付き手順**：[コードのダウンロード](https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/download_code.html)

### G-2. 移動速度のパラメータを用意する

**リモコンの移動速度**：`Zeus_Car.ino` の `IR_REMOTE_POWER` の数値を変更します（既に存在します）。

```cpp
// 変更前
#define IR_REMOTE_POWER 80
// 変更後（例）
#define IR_REMOTE_POWER 55
```

**App の移動速度（上限）**：

**(a) 上限用のマクロを追加**：`Zeus_Car.ino` の `IR_REMOTE_POWER` などが並んでいる箇所に、次の 1 行を追加します。

```cpp
#define APP_REMOTE_POWER 70   // Appの最大速度（0〜100）
```

**(b) ジョイスティック値に上限をかける**：`Zeus_Car.ino` の `onReceive()` 内にある次の箇所を書き換えます（元々コメントアウトされた行が用意されています）。

```cpp
// 変更前
  uint8_t power = aiCam.getJoystick(REGION_K, JOYSTICK_RADIUS);
  // power = map(power, 0, 100, 0, CAR_DEFAULT_POWER);

// 変更後
  uint8_t power = aiCam.getJoystick(REGION_K, JOYSTICK_RADIUS);
  power = map(power, 0, 100, 0, APP_REMOTE_POWER);
```

### G-3. 旋回速度を App／リモコンで独立させる改造

旋回は App もリモコンも、内部の「方位を保つ計算（PID）」を共通で通ります。そのままでは別々にできないため、**旋回の上限を操作方法ごとに切り替えられるよう、一度だけプログラムを少し改造**します。編集するファイルは 3 つです。落ち着いて上から順に置き換えてください。

**改造1：`car_control.h` — 関数に「旋回上限」の引数を追加**

```cpp
// 変更前
void carMoveFieldCentric(int16_t angle, int8_t power, int16_t heading=0, bool drift=false, bool angFlag=false);

// 変更後
void carMoveFieldCentric(int16_t angle, int8_t power, int16_t heading=0, bool drift=false, bool angFlag=false, uint8_t rotLimit=100);
```

**改造2：`car_control.cpp` — 関数の中身を上限に対応させる**

(a) 関数の先頭行（引数を受け取れるようにする）：

```cpp
// 変更前
void carMoveFieldCentric(int16_t angle, int8_t power, int16_t heading, bool drift, bool angFlag) {

// 変更後
void carMoveFieldCentric(int16_t angle, int8_t power, int16_t heading, bool drift, bool angFlag, uint8_t rotLimit) {
```

(b) 旋回量を制限している行（`±100` 固定を、引数の上限に置き換える）：

```cpp
// 変更前
    rot += max(-100, min(100, offset));

// 変更後
    rot += max(-(int16_t)rotLimit, min((int16_t)rotLimit, offset));
```

**改造3：`Zeus_Car.ino` — 旋回上限のマクロを追加し、App／リモコンに割り当てる**

(a) 上限マクロを追加（`IR_REMOTE_POWER` などの近くに）：

```cpp
#define IR_ROTATE_LIMIT  100   // リモコンの旋回速度（0〜100）
#define APP_ROTATE_LIMIT 60    // Appの旋回速度（0〜100）
```

(b) `modeHandler()` 内の呼び出しに上限を渡す：

```cpp
// 変更前
  case MODE_REMOTE_CONTROL:
    rgbWrite(MODE_REMOTE_CONTROL_COLOR);
    carMoveFieldCentric(remoteAngle, remotePower, remoteHeading, remoteDriftEnable);
    lastRemotePower = remotePower;
    break;
  case MODE_APP_CONTROL:
    rgbWrite(MODE_APP_CONTROL_COLOR);
    carMoveFieldCentric(remoteAngle, remotePower, remoteHeading, appRemoteDriftEnable);
    lastRemotePower = remotePower;
    break;

// 変更後
  case MODE_REMOTE_CONTROL:
    rgbWrite(MODE_REMOTE_CONTROL_COLOR);
    carMoveFieldCentric(remoteAngle, remotePower, remoteHeading, remoteDriftEnable, false, IR_ROTATE_LIMIT);
    lastRemotePower = remotePower;
    break;
  case MODE_APP_CONTROL:
    rgbWrite(MODE_APP_CONTROL_COLOR);
    carMoveFieldCentric(remoteAngle, remotePower, remoteHeading, appRemoteDriftEnable, false, APP_ROTATE_LIMIT);
    lastRemotePower = remotePower;
    break;
```

### G-4. 保存して書き込む

編集が終わったら、**すべてのファイルを保存**して第3部（書き込み）へ進みます。以降の日常的な速度調整は、第2部と同じく `Zeus_Car.ino` の 4 つの数字（`IR_REMOTE_POWER` / `APP_REMOTE_POWER` / `IR_ROTATE_LIMIT` / `APP_ROTATE_LIMIT`）を変えるだけです。

---

## 参考リンク

**公式ドキュメント（プログラミング）**
- Arduino IDE でのプログラミング（日本語・入口）：`https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/programming_mode.html`
- コードのダウンロード：`https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/download_code.html`
- Arduino IDE 2.0 のダウンロードとインストール：`https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/install_arduino_ide.html`
- 必要なライブラリをインストールする：`https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/add_libraries.html`

**今回の調整に関係する解説ページ**
- 8. 移動 - フィールドセントリック制御システム（旋回の仕組み）：`https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/ar_move_field_centric.html`
- 17. APP 制御 / 18. APP Control Plus（アプリ操作とウィジェット）：`https://docs.sunfounder.com/projects/zeus-car/ja/latest/get_started/ar_app_control_plus.html`

**コード・その他**
- SunFounder Zeus Car 公式コード：`https://github.com/sunfounder/zeus-car`
- SunFounder Zeus Car 公式ドキュメント（トップ）：`https://docs.sunfounder.com/projects/zeus-car/ja/latest/index.html`
- 配布用（調整済み）コード（SharePoint 共有リンク）：`<SPO共有リンク>`
- 配布用（調整済み）コード（GitHub リポジトリ）：`https://github.com/amane513/zeus-car`
