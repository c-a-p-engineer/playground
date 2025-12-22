# Auto Robot Survivors (P0-01 Movement AI) 仕様

## 目的
- 操作なし観戦の俯瞰2Dプロトタイプを、ブラウザで単体HTMLとして動かす。
- 移動AIを「性格ミックス(スライダー) + 安全装置付き突撃」で実装し、挙動を可視化する。

## 画面/入力
- 解像度: 720x1280 固定、カメラ固定。
- 背景: dark + simple grid。
- 入力: プレイヤー操作なし。HTML range スライダーのみ。
- 初期画面: 機体/武器種(2種)/AI性格を選択して開始。

## エンティティ
### ロボ
- 初期値: x=360, y=640, hp=100, maxHp=100, en=100, maxEn=100, speed=260。
- 視覚: 半径18のシアン円 + 方向線 + ターゲット線。

### 敵
- 種別: mob / elite / boss。
- スポーン: 画面外縁から、220ms間隔、最大55体。
- 分布: boss 3%, elite 9%, mob 88%。
- 速度: mob 160, elite 130, boss 90。
- 視覚: mob白/半径10, elite橙/半径14, boss赤/半径22。

## 移動AI
- move types: flee / far / mid / near / charge。
- スライダーは合計100に正規化しない。
- 突撃はハードゲート条件を満たさない場合は候補から除外。

### situation features
- hpRate = hp/maxHp
- enRate = en/maxEn
- nearestDist = 最近傍敵距離
- density = 半径220内の敵数
- hasElite / hasBoss
- wasHitRecently = nowMs - lastHitAtMs <= 700

### multipliers
- hpRate < 0.3 → flee 1.8, charge 0.2, near 0.7
- density > 6 → flee 1.5, near 0.7, charge 0.4
- wasHitRecently → flee 1.6, charge 0.6
- enRate > 0.7 → charge 1.3
- hasBoss → charge 1.2, far 0.9

### charge hard gate
- enRate > 0.6
- density < 4
- hasBoss or hasElite
- wasHitRecently == false
- nowMs - lastChargeAtMs >= 1800

### selection
- score = weight * multiplier
- tie-break: mid > far > flee > near > charge

### distance bands
- far 420 / mid 260 / near 140 / deadband 20

## システム
- EN: regen 18/sec, charge cost 35/sec。
- フェイク被弾: 2600ms間隔, 35%で6ダメージ。

## デバッグ
- MOVE理由・スコア・hp/en/density/nearest 等を表示。
- ターゲット線(橙) + 進行方向線(シアン)。

## 既知の制約
- 物理演算なし (Arcade Physics 非使用)。
- 攻撃/ダメージはP0デモ用の簡易処理。
