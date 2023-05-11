<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　Oculus Go向けにUnityプロジェクトのビルドを実施したらときにエラー（AndroidSDKToolsException）が発生したのでその解決方法をメモ。

## 発生したエラー

<div class="sm-code">
```
/Users/admin/unity/OculusGoSample/Temp/StagingArea/AndroidManifest-main.xml:4:16-57 Error:
	Attribute application@theme value=(@style/UnityThemeSelector) from AndroidManifest-main.xml:4:16-57
	is also present at AndroidManifest.xml:3:83-147 value=(@android:style/Theme.Black.NoTitleBar.Fullscreen).
	Suggestion: add 'tools:replace="android:theme"' to <application> element at AndroidManifest-main.xml:4:3-12:17 to override.

UnityEditor.HostView:OnGUI()
```
```
AndroidSDKToolsException: Unable to merge android manifests. See the Console for more details. 
UnityEditor.Android.AndroidSDKTools.DetectErrorsAndWarnings (System.String logMessages, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.RunCommandInternal (System.String javaExe, System.String sdkToolsDir, System.String[] sdkToolCommand, Int32 memoryMB, System.String workingdir, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.RunCommandSafe (System.String javaExe, System.String sdkToolsDir, System.String[] sdkToolCommand, Int32 memoryMB, System.String workingdir, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.RunCommand (System.String[] sdkToolCommand, Int32 memoryMB, System.String workingdir, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.RunCommand (System.String[] sdkToolCommand, System.String workingdir, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.RunCommand (System.String[] sdkToolCommand, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.MergeManifests (System.String target, System.String mainManifest, System.String[] libraryManifests, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit)
UnityEditor.Android.PostProcessor.Tasks.GenerateManifest.MergeManifests (UnityEditor.Android.PostProcessor.PostProcessorContext context, System.String targetManifest, System.String mainManifest)
UnityEditor.Android.PostProcessor.Tasks.GenerateManifest.Execute (UnityEditor.Android.PostProcessor.PostProcessorContext context)
UnityEditor.Android.PostProcessor.PostProcessRunner.RunAllTasks (UnityEditor.Android.PostProcessor.PostProcessorContext context)
UnityEditor.HostView:OnGUI()
```
```
UnityEditor.BuildPlayerWindow+BuildMethodException: Build failed with errors.
  at UnityEditor.BuildPlayerWindow+DefaultBuildMethods.BuildPlayer (BuildPlayerOptions options) [0x001bf] in /Users/builduser/buildslave/unity/build/Editor/Mono/BuildPlayerWindowBuildMethods.cs:163 
  at UnityEditor.BuildPlayerWindow.CallBuildMethods (Boolean askForBuildLocation, BuildOptions defaultBuildOptions) [0x00050] in /Users/builduser/buildslave/unity/build/Editor/Mono/BuildPlayerWindowBuildMethods.cs:83 
UnityEditor.HostView:OnGUI()
```
</div>

## 解決方法

　Android SDK Toolsのバージョンがよくないらしい。  

１. 下のサイトから「**tools_r25.2.5-macosx.zip**」をダウンロード

[https://androidsdkoffline.blogspot.com/p/android-sdk-tools.html:title]

２. SDKパス配下のtoolsディレクトリ(`/Users/admin/Library/Android/sdk/tools`)を上記手順でダウンロードしたフォルダに置き換える。

SDKパス確認は"External Tools"から確認できます。
![](https://i.imgur.com/BlzL4Og.png)

## 参考

[https://answers.unity.com/questions/1355171/androidmanifest-mainxml-merging-error.html:embed:cite]

