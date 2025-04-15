WinUI3 ノウハウメモ書き  
https://github.com/drumsmidi/DrumMidiEditorApp

# x:Bind

```xml
<TextBox Text="{x:Bind MusicInfo.BgmFilePath, Mode=TwoWay}" 
  TextChanged="MusicInfoBgmFilePathTextBox_TextChanged" />
```

- OneTime  
　　Page読み込み時に一度だけデータ取得

- OneWay  
　　Page読み込み時に一度だけデータ取得。  
　　データに対して変更があった際に、画面側にも反映する。  
　　但し、自動で変更してくれるわけではなく意図的に通知が必要となる。

- TwoWay  
　　OneWay の機能＋画面上で入力した値は、データに書き込みに行く。  
　　但し、Changedイベントなどでは、データに書き込みはまだ行われていない為  
　　イベント内でデータを使用する場合には注意が必要。  
　　また、ListView_SelectionChanged イベントなどはデータの書き換え後に処理される。


```csharp
public sealed partial class PageMusic : Page, INotifyPropertyChanged
{
	public event PropertyChangedEventHandler? PropertyChanged = delegate { };

	public void OnPropertyChanged( [CallerMemberName] string? aPropertyName = null )
	{
		PropertyChanged?.Invoke( this, new( aPropertyName ) );
	}

	// MusicInfoの情報に変更あった際に実行し、意図的にバインド変数に変更があったよと知らせる。
	public void ReloadMusicInfo()
	{
		OnPropertyChanged( "MusicInfo" );
	}
}
```

参考URL  
https://docs.microsoft.com/ja-jp/windows/uwp/data-binding/function-bindings


# リソース定義

![image](https://github.com/user-attachments/assets/9654f374-3428-4b22-8594-56e95477cf0c)

## 多言語化

Package.appxmanifest ファイルをコード表示で表示し、リソースを追加する。
```xml:Package.appxmanifest
  <Resources>
    <Resource Language="ja-JP"/>
    <Resource Language="en-US"/>
  </Resources>
```

[Strings] フォルダ配下に [言語フォルダ]/Resources.resw を作成する。  
![image](https://github.com/user-attachments/assets/18eb40f9-c5bc-4a3a-9ee7-0db56b432a3f)

## XAML内でのリソース参照方法

XAML内でリソースを参照する場合
```xml:XAML
  <ComboBox x:Uid="ChannelNoComboBox">
```

リソースには下記のような感じで設定
```:Resources.resw
ChannelNoComboBox.Header = ヘッダテキスト  
ChannelNoComboBox.Width = 100 
```

## C#内でのリソース参照方法

C#リソース内で参照する場合
```csharp
public static class HelperResources
{
    private static readonly ResourceLoader _Resource = new();

    public static string GetString( string aKey )
        => _Resource.GetString( aKey );

    public static string GetString( string aKey, params object [] aParams )
        => string.Format( GetString( aKey ), aParams );
}

// リソースの値を取得
HelperResources.GetString( "LabelBpm" );

// リソース内の「AAA.BBB.CCC」の項目を参照する場合は
// "."⇒"/" に置き換えて取得が必要
HelperResources.GetString( "AAA/BBB/CCC" )

// バインド変数の値を置き換えて取得する。
// リソースの設定："あいうえお：{0}, {1}"
// 取得値："あいうえお：A, B"
HelperResources.GetString( "DialogChangeKey/Content", "A", "B" )
```

# ContentDialogの最大サイズを変更する

デフォルト設定だと横幅が狭いので、拡張する場合は App.xaml内に設定を追加する
```xaml:App.xaml
<Application
    x:Class="DrumMidiEditorApp.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:controls="using:Microsoft.UI.Xaml.Controls">

    <!-- https://github.com/microsoft/microsoft-ui-xaml/issues/424 -->
    <Application.Resources>
        <controls:XamlControlsResources>
            <controls:XamlControlsResources.MergedDictionaries>
                <ResourceDictionary>
                    <x:Double x:Key="ContentDialogMaxHeight">960</x:Double>
                    <x:Double x:Key="ContentDialogMaxWidth">1280</x:Double>
                </ResourceDictionary>
            </controls:XamlControlsResources.MergedDictionaries>
        </controls:XamlControlsResources>
    </Application.Resources>
</Application>
```

確か、デフォルト設定だと横幅が狭くて、下記のような２列表示ができなかった  
![image](https://github.com/user-attachments/assets/1fe813a6-7068-4699-9efb-386dbf93c7d2)

# ファイルからプログラムを開いた際に、ファイルパスを取得する方法

## 拡張子の関連付け

Package.appxmanifest 内の宣言項目より、サポートされるファイルの種類を設定する。  
![image](https://github.com/user-attachments/assets/188f7fcb-45cc-4b0c-b276-87388090bfa4)

## プログラム実行時のファイルパス取得方法

```csharp:App.xaml.cs
public partial class App : Application
{
    protected override void OnLaunched( LaunchActivatedEventArgs aArgs )
    {
        #region 起動元ファイルのパス取得
        {
            var active_event_args = AppInstance.GetCurrent().GetActivatedEventArgs();

            if ( active_event_args.Kind == ExtendedActivationKind.File )
            {
                if ( active_event_args.Data is IFileActivatedEventArgs data )
                {
                    if ( data.Files.Count > 0 )
                    {
                        var path = data.Files [ 0 ].Path ?? string.Empty;
                    }
                }
            }
        }
        #endregion
    }
}
```
