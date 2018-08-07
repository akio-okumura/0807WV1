# 8/7,Mozilla WebVRアセットの実験用レポジトリ

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
