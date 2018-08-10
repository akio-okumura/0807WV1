# 8/7,Mozilla WebVRアセットの実験用レポジトリ
## 結論

**webvr.js**から**WebVRCameraSet**へのSendMessageが出来ていないことで、

VR化を制御するboolまで変更が届いていなかった。

なので、Unityシーン内にwebvr.jsとWebVRCameraSetを橋渡しするオブジェクトを配置。

一旦そのオブジェクトをwebvr.jsで呼び出し、そこからWebVRCameraSetの関数に触るようにした。

具体的には、webvr.js内の`gameInstance.SendMessage('WebVRCameraSet', 'OnStartVR');`

呼び出しを、`gameInstance.SendMessage('transmit', 'Transmit_OnStartVR');`

と変更し、**transmit**オブジェクト内に**OnStartVR**を呼び出す関数、

```
public void Transmit_OnStartVR() {
  OnStartVR();
}
```

を定義する。これで橋渡しが完成する。

## この解決法の問題点

本来であれば橋渡しをせず直接WebVRCameraSetに触るのでHMDのポジショントラッキングにラグが殆ど生まれない。

が今回はブリッジした為映像にカクつきが出てしまっている。

この問題は一旦スルーでいいかと考えている。


# 実験経緯

アセットをそのまま入れて、Buildしたら**VR化出来なかった**

また、昨日VR化可能だったページを開いても**VR化出来なかった**

しかし、1度はデモシーンをそのまま開けばVR化出来たことから、

私は、原因を、**OculusGoがVRデバイスと認識されている時とされていない時がある**可能性を見出した。


- WebVRCamera.cs のvrAcitiveが**常にFalse**になっている。

- vrActiveは onVRChangeメソッドで変更している。

```

private void onVRChange(WebVRState state)
{
    vrActive = state == WebVRState.ENABLED;
}

```

- これはWebVRManagerのOnVRChangeデリゲートに含まれている。

- `state == WebVRState.ENABLED`は、**!stateを返す関数**

- つまりは、`onVRChange(ENABLED)`で`vrActive = true`となり、ステレオカメラ化出来る。

- OnVRChangeが呼ばれるのは、WebVRManager.cs(以後、WVM)の**setVrState**

  - setVrState(WebVRState.ENABLED)であればvrActiveがtrue.VR化出来る。

  - setVrStateが呼ばれるのは、toggleVrStateとOnStartVRとOnEndVR。

  今回toggleVrStateは無視する。`setVrState(WebVRState.ENABLED)`になるのは、

  OnStartVRなので、次はOnStartVRを追う。

- OnStartVR

  - OnStartVRを呼んでいるのは、`webvr.js`の**OnRequestPresent**,**onUnity**

  - webvr.jsに変更を加えた。`SendMessageでDebugOnjにtextを出力させる`

  (webvj.jsはテンプレートに組まれているので一旦別場所に保存してからビルド)

  - access by onUnity function と出たので、onUnity関数のしかも

  `gameInstance.SendMessage('WebVRCameraSet', 'OnStartVR')`の手前まで届いている。

  **そのままOnStartVRが呼ばれているはず、おかしい**

  - またコンソールの中身を出力と、vrActiveの状態を出力してみる.

    - 1度目は「StartedVR」

    - 1回Escapeして2度目は「Entered VR mode」

  - おかしい、**OnStartVRは呼ばれている**

# webvr.js 挙動まとめ

- 初めてVRモードに入る時は、**onUnity()**

- 二度目以降VRモードに入る時は、**onRequestPresent()**

- 87行目、`gameInstance.SendMessage('WebVRCameraSet', 'OnStartVR');`が上手く行っていない気がする。

- 8/7,23:00コミットwebvr.jsから、SendMessageをしっかり出来るか

  もしかしたら、親コンポーネントにSendMessageは効かない可能性あり
