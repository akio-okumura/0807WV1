# 8/7,Mozilla WebVRアセットの実験用レポジトリ

アセットをそのまま入れて、Buildしたら**VR化出来なかった**

また、昨日VR化可能だったページを開いても**VR化出来なかった**

しかし、1度はデモシーンをそのまま開けばVR化出来たことから、

私は、原因を、**OculusGoがVRデバイスと認識されている時とされていない時がある**可能性を見出した。

1. WebVRCamera.cs のvrAcitiveが**常にFalse**になっている。

1. vrActiveは onVRChangeメソッドで変更している。

```
private void onVRChange(WebVRState state)
{
    vrActive = state == WebVRState.ENABLED;
}
```

  1. これはWebVRManagerのOnVRChangeデリゲートに含まれている。

  1. `state == WebVRState.ENABLED`は、**!stateを返す関数**

  1. つまりは、`onVRChange(ENABLED)`で`vrActive = true`となり、ステレオカメラ化出来る。

3. OnVRChangeが呼ばれるのは、WebVRManager.cs(以後、WVM)の**setVrState**

  3.1 setVrState(WebVRState.ENABLED)であればvrActiveがtrue.VR化出来る。

  3.2 setVrStateが呼ばれるのは、toggleVrStateとOnStartVRとOnEndVR。

  今回toggleVrStateは無視する。`setVrState(WebVRState.ENABLED)`になるのは、

  OnStartVRなので、次はOnStartVRを追う。

4. OnStartVR

  4.1 OnStartVRを呼んでいるのは、`webvr.js`の**OnRequestPresent**,**onUnity**

  4.2 webvr.jsに変更を加えた。`SendMessageでDebugOnjにtextを出力させる`

  (webvj.jsはテンプレートに組まれているので一旦別場所に保存してからビルド)
