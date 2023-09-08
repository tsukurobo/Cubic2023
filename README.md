**目次**

- [Cubic2023](#cubic2023)
  - [概要](#概要)
    - [コメント](#コメント)
- [Cubic2023\_DCMotorDriver](#cubic2023_dcmotordriver)
  - [概要](#概要-1)
    - [connector](#connector)
    - [pull down](#pull-down)
    - [A3921](#a3921)
    - [current sensor](#current-sensor)
    - [Filter](#filter)
    - [DeadTime](#deadtime)
    - [Protection Circuit](#protection-circuit)
    - [for motor connection](#for-motor-connection)
    - [Bootstrap Chargepump](#bootstrap-chargepump)
    - [Indication Circuit](#indication-circuit)
    - [V5 Circuit](#v5-circuit)
    - [Decoupling Condenser](#decoupling-condenser)
    - [Bridge](#bridge)
    - [pull up](#pull-up)
    - [error lamp](#error-lamp)
    - [PS](#ps)
    - [BD63150](#bd63150)
    - [Protection Circuit](#protection-circuit-1)
    - [Indication Circuit](#indication-circuit-1)
    - [VREF](#vref)
    - [for solenoid valve](#for-solenoid-valve)
- [Cubic2023\_RP2040](#cubic2023_rp2040)
  - [概要](#概要-2)
    - [RP2040](#rp2040)
    - [bypass condenser](#bypass-condenser)
    - [surge protection](#surge-protection)
    - [connector](#connector-1)
    - [indicator](#indicator)
    - [RESET switch](#reset-switch)
    - [microUSB](#microusb)
    - [QSPI NOR flash](#qspi-nor-flash)
    - [crystal](#crystal)



# Cubic2023

## 概要
つくばろぼっとサークルではNHK学生ロボコンに向けて統合型基板を製作しており、mainがNHK学生ロボコン2023において使用したバージョンです。
回路図の説明やブロック分けなどが出来ておらず、分かりにくい状態です。※修正中

### コメント
修正中です。8/10程度を目途に進めています。

# Cubic2023_DCMotorDriver
## 概要
RZ-735クラスのDCモータを2基、RS-555クラスのDCモータを1基、24Vで駆動することが出来る基板です。以下ではRZ-735を想定したチャネルをメインチャネル、他方をサブチャネルと呼称します。

メインチャネルではICにA3921を採用しています。サブチャネルではICにBD63150を採用しています。データシートを参考に作成しました。データシートに載っている内容は説明を省略します。

サブチャネルはハーフブリッジ×2と見なすことで電磁弁を制御することも出来ます。

https://www.allegromicro.com/ja-jp/products/motor-drivers/brush-dc-motor-drivers/a3921

https://www.rohm.co.jp/products/motor-actuator-drivers/dc-brush-motor/bd63150afm-product

### connector
電流センサのアナログ値伝送用に4pin、信号・降圧電源用に14pin、VDDとGNDPWRに各14pinでMotherBoardと接続します。GNDは信号用と電源用で分離しています。

### pull down
MotherBoardからの信号がない場合の誤動作防止のためにプルダウン抵抗を挿入しています。

### A3921
Cissの大きいMOSFETを容易に駆動できる便利なICです。SR,PWMLにhighを入力、pwmhにPWM、phaseにhigh/lowを入力することで、SMB方式でモータを駆動できます。

### current sensor
メインチャネルを電流制御するために搭載した電流センサICです。コスト面からACS712を採用しましたが、動作を確認したところ、ノイズの影響で大きかったため、現状電流を用いた制御は実装していません。

### Filter
MotherBoardに大容量の電解コンデンサを搭載していますが、ピンヘッダを経由するため補助として中容量の電解コンデンサを搭載しています。

### DeadTime
$$ 50+\frac{7200}{1.2+(200/Rdead[kΩ])} $$
datasheet上の数式に従いdeadtimeが2300nsになるように抵抗値を設定しました。

### Protection Circuit
RCスナバ回路と双方向ツェナーダイオードによる保護回路を搭載しています。RCスナバ回路の値は以下の式に基づいて決定しました。回路を解析して搭載したわけではなく、簡易的な計算から求めているため効果は薄いと思います。

$$ Rsnb = \frac{Vin}{Iin} = \frac{24[V]}{2.4[A]} = 10[Ω] $$
$$ Csnb = \frac{Prsnb}{f×{Vin}^2} = \frac{0.1[W]}{60k[Hz]×{24[V]}^2} = 2.89n[F] $$

$$ Iin:想定電流, Prsnb：スナバ抵抗の定格電力, f:MD動作周波数$$
※想定電流は実際より小さめ、MD動作周波数は実際より大きめの値で計算されてます。設計初期の名残です。

モーターの逆起電力からMDを保護するために降伏電圧が28Vの双方向ツェナーダイオード（TVS）を採用しています。

**注：現行基板では、逆起電力から保護しきれていないことが原因と思われるA3921の破壊が確認されています。**

### for motor connection
モーターに接続する端子です。

### Bootstrap Chargepump
DCバイアスを考慮してコンデンサを配置しました。

### Indication Circuit
モーターに接続している箇所にLEDを対面配置することで回転方向を表示しています。プログラム開発時に便利でした。ローサイドMOSFETのゲート端子にデバッグ用のLEDを配置しています。ICが壊れた場合に表示されなくなるのでデバッグに活用できました。

|  LEDの色  |  用途  |
| ---- | ---- |
|  緑  |  常時点灯  |
|  青  |  電源ON＆正常時点灯  |
|  赤  |  異常時点灯  |
|  黄  |  状態表示  |
|  白  |  状態表示  |

### V5 Circuit
A3921に搭載されている5Vレギュレータを利用してA3921搭載のエラーLEDを配置しています。また、VDSTHに送る電圧を分圧で生成しています。

### Decoupling Condenser
ブートストラップコンデンサの容量が大きく、A3921の挙動が不安定になることがあったので、10uFのコンデンサを搭載しました。

### Bridge
フルブリッジ回路です。前述のRCスナバ回路を搭載しています。MOSFETは**RJ1L12BGN**を採用しています。corestaffで購入すると比較的安価に購入可能です。ON抵抗は小さいですが、ゲート容量は大きいので気を付けてください。新規設計する場合は**RS6L120BG**がおすすめです。ゲート抵抗には27Ωを採用しました。この抵抗値にした理由は勘です。

簡易的ではありますが以下の式で計算可能です。
$$ \frac{Vg}{Imax} < Rg < \frac{Tdead}{3×Ciss}$$
$$ Rg:ゲート抵抗,Vg:ゲート駆動電圧,Imax:ICがゲートに流せる最大電流,Tdead:デッドタイム,Ciss:MOSFET入力容量　$$
※条件によっては解なしになるかと思います。

### pull up
MDへの入力信号をプルアップしています。

### error lamp
BD63150のエラー表示用のLEDです。

### PS
PowerSaveに繋いでる端子です。プルアップしています。

### BD63150
電圧制御・電流制限に対応したモータードライバICです。RFNには100mΩのシャント抵抗を搭載しています。3Aで電流制限をする計算でしたが、想定より厳しめに作動していました。恐らくGNDを信号とパワーで分離した弊害だと思われます。

### Protection Circuit
メインチャネルと同様の構成です。

### Indication Circuit
メインチャネルと同様の構成です。

### VREF
電流の制限値をICに入力するピンです。3Aで制限するように設定したのですが、前述の通り思うように動きませんでした。

### for solenoid valve
電磁弁を駆動できるように付加した回路です。セーフティとしてリセッタブルヒューズを載せています。また還流ダイオードも載せています。

# Cubic2023_RP2040
## 概要
Cubicでは以下の理由から組み込みマイコンにRP2040を採用しました。
- GPIOとPWMが多い
- cortex-M0（計算速度がそこそこ速い）
- 安価
- 書き込みが容易

システム内で3つ使い、容易に交換可能とするためにマイコンボードとして最小構成の基板を作成しました。

以下の公式リファレンスを参考に作成しました。公式リファレンスに載っている内容は説明を省略します。
https://datasheets.raspberrypi.com/rp2040/hardware-design-with-rp2040.pdf

### RP2040
RP2040と周辺の配線です。
### bypass condenser
マイコンの電力消費が多く動作が不安定になることがあったため、100uFのセラコンを追加しています。
### surge protection
システム内でモータドライバと絶縁せずにRP2040を用いている箇所があるため、サージ保護としてツェナーダイオードを電源に並列に繋いでいます。
### connector
基板サイズの制約の中でGPIOをフルに使う必要があり、1.27mmピッチのピンヘッダを採用しました。配線の最適化と差し込みの一意性を担保するために基板の3辺に沿うように配置しています。
### indicator
基板内通信にSPIを利用しているのですが、SPI通信の状態を外部から容易に観測するためにslave側のSSに該当する箇所にLEDを付けています。以下は目視のLED状態とSSに注意したSPI通信の状況です。
|  LED  |  状況  |
| ---- | ---- |
|  点灯  |  通信異常  |
|  僅かに点灯  |  正常  |
|  点滅  |  通信速度低  |
|  消灯  |  オフライン  |

### RESET switch
RP2040をリセットできます。接続先がMotherBoardの場合は、接続先もリセットされます。

### microUSB
RP2040にプログラムを書き込むmicroUSBです。基本的に動作中の電源は接続先の基板から受け取るため、RP2040に書き込む以外は使用しない想定です。

### QSPI NOR flash
プログラムが書き込まれるメモリです。

### crystal
RP2040動作のためのcrystalです。

