# gazebo -ros control-

この文章は[gazebo ros](http://gazebosim.org/tutorials/?tut=ros_comm)を参考程度に和訳したものです．

## 用語

- state : 剛体のpose(位置)とtwist(姿勢)のこと
- properties : 質量とか摩擦とかのこと
- body : 剛体のことを指す
- link : URDFのファイルの中で使用されるのと同様の意味合いでここでも使用される
- model : jointで結ばれたbodyの集合体のこと



## gazebo_ros_api_pluginについて

`gazebo_ros_api_plugin`と呼ばれるプラグインは`gazebo_ros`パッケージとともに読み込まれるプラグインのことで，rosノードとしてのgazeboが初期化される際に読み込まれる．`gazebo_ros_api_plugin`により，ユーザーはrosのapiを用いてシミュレーション環境のproperties(質量とか，摩擦とか)を操作できるだけでなく，modelをシミュレーション中に挿入したりすることができる．このプラグインは`gzserver`にのみ読み込まれる．

## gazebo_ros_paths_pluginについて

`gazebo_ros_paths_plugin`は`gazebo_ros`パッケージの中に含まれているプラグインで，`gazebo_ros_paths_plugin`によって`gazebo`は簡単にros中のリソースを見つけることを可能にする．例えば，パッケージ名のパス解決などを行うことができる．このプラグインは`gzserver`と`gzclient`で読み込まれる．

## gazeboが送信(publish)するparameterとtopic

gazeboは下記の２つの情報を送信している

parameter : `/use_sim_time` : **Bool** 

時刻としてrosの各ノードが参照している`ROS time`に`/clock`を使用することを推奨するパラメータ. 

```
rosparam get /use_sim_time
```

```
true
```



topic : `clock` 

gazebo中の現在時刻をtopicとして送信する．

```
rostopic echo /clock
```

```
---
clock: 
  secs: 434
  nsecs: 943000000
---
```



## gazeboが購読(subscribe)しているtopic

gazeboは下記の２つのtopicを購読している．

`~/set_link_state` : `gazebo_msgs/LinkState`

このtopicを通してlinkのstate(pose/twist)を設定することができる

```
rostopic info /gazebo/set_link_state
```



`~/set_model_state` : `gazebo_msgs/ModelState`

このtopicをとおしてmodelのstate(psoe/twist)を設定することができる

```
rospopic info /gazebo/set_model_state
```

### Topicを通してモデルの位置と姿勢を設定する

これらのtopicを通してgazeboに干渉する方法では，`action (ros actionのこと)`が完了するのを待つ必要がないため，高速に処理が行える．topicを通してモデルに目標の位置と姿勢を設定するには`model state message`を`/gazebo/set_model_state`トピックに送ればよい．

ためしに，シミュレータ内にオンラインデータベースからコカコーラの缶を挿入し，topicによってロボットの位置を変更してみよう．

```
rosrun gazebo_ros spawn_model -database coke_can -sdf  -model coke_can -y 1 
```

topicの`/gazebo/set_model_state`を送ることによってコーラの缶の姿勢を変えてみよう．

```
rostopic pub -r 20 /gazebo/set_model_state gazebo_msgs/ModelState "model_name: 'coke_can'
pose:
  position:
    x: 1.0
    y: 0.0
    z: 2.0
  orientation:
    x: 0.0
    y: 0.49
    z: 0.0
    w: 0.8
twist:
  linear:
    x: 0.0
    y: 0.0
    z: 0.0
  angular:
    x: 0.0
    y: 0.0
    z: 0.0
reference_frame: 'world'" 
```

![Screenshot from 2019-01-04 21-55-10](../image/2019-01-04 21-55-10.png)

20Hzで位置を送信しているので，なんだか震えているように見える．

## gazeboが配信(publish)しているtopic

gazeboは下記の３つのtopicを配信している．

`/clock` : `rosgraph_msgs/Clock` 

このtopicはシミュレーションの現在時刻を配信する．`/use_sim_time`パラメータと一緒に使用される

`~/link_states` : `gazebo_msgs/LinkStates`

このtopicはシミュレーション中の全リンクのstate(位置と姿勢)を配信する

`~/model_states` : `gazebo_msgs/ModelStates`

このtopicはシミュレーション中の全モデルのstate(位置と姿勢)を配信する

### topicを用いてモデルとリンクのstateを取り出す

gazeboは`/gazebo/link_states/`と`/gazebo/model_states`topicを配信しており，それらはシミュレーション中に存在する物体の位置と姿勢に関する情報を持っている．ただし，それらの座標系は**world**座標系である．

```
rostopic echo -n 1 /gazebo/model_states
```

```
name: [ground_plane, coke_can]
pose: 
  - 
    position: 
      x: 0.0
      y: 0.0
      z: 0.0
    orientation: 
      x: 0.0
      y: 0.0
      z: 0.0
      w: 1.0
  - 
    position: 
      x: 1.11122948075
      y: 0.0596752875869
      z: 0.0662327169434
    orientation: 
      x: -0.23373060486
      y: 0.666667965594
      z: -0.29032521043
      w: 0.645472772619
twist: 
  - 
    linear: 
      x: 0.0
      y: 0.0
      z: 0.0
    angular: 
      x: 0.0
      y: 0.0
      z: 0.0
  - 
    linear: 
      x: 0.0
      y: 0.0
      z: 0.0
    angular: 
      x: 0.0
      y: 0.0
      z: 0.0
---
```

```
rostopic echo -n 1 /gazebo/link_states
```

```
name: ['ground_plane::link', 'coke_can::link']
pose: 
  - 
    position: 
      x: 0.0
      y: 0.0
      z: 0.0
    orientation: 
      x: 0.0
      y: 0.0
      z: 0.0
      w: 1.0
  - 
    position: 
      x: 1.11122948075
      y: 0.0596752875869
      z: 0.0662327169434
    orientation: 
      x: -0.23373060486
      y: 0.666667965594
      z: -0.29032521043
      w: 0.645472772619
twist: 
  - 
    linear: 
      x: 0.0
      y: 0.0
      z: 0.0
    angular: 
      x: 0.0
      y: 0.0
      z: 0.0
  - 
    linear: 
      x: 0.0
      y: 0.0
      z: 0.0
    angular: 
      x: 0.0
      y: 0.0
      z: 0.0
---
```

繰り返しになるが，リンクは慣性<inertial>，視覚<visual>および，衝突特性<collision>をもつ剛体として定義される．一方，モデルはリンクとジョイントの集まりとして定義される．モデルのstateはそのモデルの基底リンクのstateとなる．URDFはツリー構造を持っているので，そこから考えると，モデルの基底リンクは，そのモデルのルートリンクとなる．

## serviceを用いたシミュレーション中のモデルの挿入と削除

下記の`ros service`はユーザーにモデルのシミュレーションへの挿入と削除を動的に行うことができる．-

- `~/spawn_urdf_model` : `gazebo_msgs/SpawnModel` 

​        このサービスはURDFをシミュレーション内へ挿入する

- `~/spawn_sdf_model` : `gazebo_msgs/SpawnModel`

​        このサービスはSDFをシミュレーション内へ挿入する

- `~/delete_model` : `gazebo_msgs/DeleteModel`

​        このサービスはシミュレーション内のモデルを削除する

### モデルの生成

`spawn_model`と呼ばれるヘルパースクリプトは`gazebo_ros`パッケージで提供され，モデル挿入サービスを提供する．serviceの最も実用的なモデル挿入の方法は`roslaunch`ファイルとともに使用する方法である．詳しくは[Using roslaunch File to Spawn Models](http://gazebosim.org/tutorials/?tut=ros_roslaunch)のチュートリアルを見るとよい．そこには多数の`spawn_model`を使用してURDFやSDFをgazeboに加える方法が示されている．

ファイルからURDFをシミュレーションに挿入する．まず，xacroファイルをxmlファイルへ変換する．

```
rosrun xacro xacro `rospack find rrbot_description`/urdf/rrbot.xacro >> `rospack find rrbot_description`/urdf/rrbot.xml
```

そして，挿入する

```
rosrun gazebo_ros spawn_model -file `rospack find rrbot_description`/urdf/rrbot.xml -urdf -y 1 -model rrbot1 -robot_namespace rrbot1
```

もし，エラー中に*junk*の文字があったら，一回作成したxmlファイルを消してから，もう一度xmlを生成すること．そうでない場合は,gazeboとかroscoreとかを全部消してからもう一度おこなうこと．

ROS melodicを使用している場合はrrbotのURDFの記述が時代遅れなためコーラの缶とrrbotを同時にシミュレーター内に挿入することはできない．

### モデルの消去

モデルを消去する方法は簡単で，もしあなたがrrbotを"rrbot1"という名前で挿入している場合は下記のコマンドによってモデルを消去できる．

```
rosservice call gazebo/delete_model '{model_name: rrbot1}'
```

## serviceを通してstateとpropertiesを設定する

`ros service`をとおして物体のstate（位置・姿勢）やproperties（摩擦など）を設定できる

- `~/set_link_properties` : `gazebo_msgs/SetLinkProperties`
- `~/set_physics_properties` : `gazebo_msgs/SetPhysicsProperties`
- `~/set_model_state` : `gazebo_msgs/SetModelState`
- `~/set_model_configuration` : `gazebo_msgs/SetModelConfiguration`

​        このサービスは動的な呼び出しなしにモデルのジョイント角度を設定することができる

- `~/set_joint_properties` : `gazebo_msgs/SetJointProperties`
- `~/set_link_state` : `gazebo_msgs/SetLinkState`
- `~/set_link_state` : `gazebo_msgs/LinkState`
- `~/set_model_state` : `gazebo_msgs/ModelState`

### モデルのstateを設定する例

**ROS Melodicだとrrbotのパッケージが古すぎてrrbotとコーラの缶を同時に存在させられないのでここはパス**

### serviceを通してstateとpropertiesの値を得る

`ros service`を通して物体のstateとpropertiesの設定された値を取得できる

- `~/get_model_properties` : `gazebo_msgs/GetModelProperties`

​        このサービスはシミュレーション内のモデルのpropertiesを返す

- `~/get_model_state` : `gazebo_msgs/GetModelState`

​        このサービスはシミュレータ内のモデルのstateを返す

- `~/get_world_properties` : `gazebo_msgs/GetWorldProperties`

​        このサービスはシミュレーションワールドのpropertiesを返す

- `~/get_joint_properties` : `gazebo_msgs/GetJointProperties`

​        このサービスはシミュレータ内のジョイントのpropertiesを返す

- `~/get_link_properties` : `gazebo_msgs/GetLinkProperties` 

​        このサービスはシミュレータ内のリンクのpropertiesを返す

- `~/get_link_state` : `gazebo_msgs/GetLinkState`

​        このサービスはシミュレータ内のリンクのstateを返す

- `~/get_physics_properties` : `gazebo_msgs/GetPhysicsProperties`
  このサービスはシミュレータで使用されている物理エンジンのpropertiesを返す

### モデルのstateを得る例

```
rosservice call /gazebo/get_model_state '{model_name: coke_can}'
```

```
header: 
  seq: 1
  stamp: 
    secs: 1546622800
    nsecs: 365554047
  frame_id: ''
pose: 
  position: 
    x: -1.31615897336e-05
    y: 0.999944301203
    z: -0.00398847138329
  orientation: 
    x: -0.00789028282505
    y: 0.00174586151438
    z: 0.000124429190607
    w: 0.999967339428
twist: 
  linear: 
    x: 0.0
    y: 0.0
    z: 0.0
  angular: 
    x: 0.0
    y: 0.0
    z: 0.0
success: True
status_message: "GetModelState: got properties"
```

ただし，`ros service`を用いるこの例だと取得まで0.3秒くらいかかるので上記のtopicを利用した方法を用いた場合のほうがいいかもしれない．

### シミュレーションのワールドと物体のproperties

実行中のシミュレータのワールドに存在するモデルのリストを得ることができる

```
rosservice call /gazebo/get_world_properties 
```

```
sim_time: 1742.647
model_names: [ground_plane, coke_can]
rendering_enabled: True
success: True
status_message: "GetWorldProperties: got properties"
```

さらに詳細をモデルごとに検索することが可能

```
rosservice call /gazebo/get_model_properties '{model_name: coke_can}'
```

```
parent_model_name: ''
canonical_body_name: ''
body_names: [link]
geom_names: [collision]
joint_names: []
child_model_names: []
is_static: False
success: True
status_message: "GetModelProperties: got properties"
```

## サービスを用いた力制御

**Melodicだとロボットと缶を同時に出現させられないのでパス**

## サービスを用いてシミュレーションをコントロール

serviceを用いて一時停止とか停止解除とかを行うことができる

- `~/pause_physics` : `std_srvs/Empty`

​        シミュレータの一時停止

- `~/unpause_physics` : `std_srvs/Empty`

​        シミュレータの一時停止解除

- `~/reset_simulation` : `std_srvs/Empty`

​        シミュレータ内の時間を含めたすべてのものをリセットする

- `~/reset_world` : `std_srvs/Empty`

​        シミュレータ内のモデルを初期状態に戻す

### 例

```
roservice call /gazebo/reset_world
```

```
rosservice call /gazebo/pause_physics
```

```
rosservice call /gazebo/unpause_physics
```

## トラブルシューティング

- gazeboを立ち上げたけどロゴが表示されて止まっており，真っ暗な世界が表示されている．

以前のgzserverが立ち上がっている可能性があります．`ps -aux | grep gz`としてみてgzserverが生き残っていたら`kill`で殺しておきましょう．
