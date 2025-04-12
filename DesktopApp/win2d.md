Win2D ノウハウメモ書き  
https://github.com/drumsmidi/DrumMidiEditorApp

- CanvasControl  
　静的な描画に適している。

- CanvasAnimatedControl  
　WinUI3 では実装されていない。（2022/07時点）

- CanvasSwapChainPanel  
　DirectXのプログラムしているとよく出てくる SwapChain

# CanvasControl

XAML
```xaml
<UserControl 
    xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
    Unloaded="UserControl_Unloaded">

    <Grid>
        <canvas:CanvasControl Draw="EditerCanvas_Draw" />
    </Grid>
</UserControl>
```

イベント処理
```csharp
private void UserControl_Unloaded( object sender, RoutedEventArgs args )
{
	// Win2D アンロード
	_EqualizerCanvas.RemoveFromVisualTree();
	_EqualizerCanvas = null;
}

private void EditerCanvas_Draw( CanvasControl sender, CanvasDrawEventArgs args )
{
}
```
# CanvasSwapChainPanel

CanvasControl から CanvasSwapChainPanel への変更は簡単  
Drawイベントがないので、非同期の描画ループ処理など、描画プロセスの実装が必要

XAML
```xaml
<UserControl
    x:Class="DrumMidiPlayerApp.pView.pPlayer.UserControlPanel"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
    Unloaded="UserControl_Unloaded"
    mc:Ignorable="d">
    
    <Grid>
        <canvas:CanvasSwapChainPanel x:Name = "_Canvas" />
    </Grid>
</UserControl>
```

C#ソース
```csharp
public UserControlPanel()
{
    InitializeComponent();

    // スワップチェイン作成
    _Canvas.SwapChain = new CanvasSwapChain
        (
            new CanvasDevice(),
            Config.Panel.ResolutionScreenWidth,
            Config.Panel.ResolutionScreenHeight,
            Config.Window.DefaultDpi,
            DirectXPixelFormat.R8G8B8A8UIntNormalized,
            2,
            CanvasAlphaMode.Premultiplied
        );

    // 描画タスク開始
    DrawTaskStart();
}

public void DrawTaskStart()
{
    // 粒度の細かいシステムではなく、タスクが長時間実行され、
    // 少量の大きなコンポーネントを含む粒度の粗い操作とすることを指定します。
    // これは、TaskScheduler に対し、オーバーサブスクリプションを許可してもよいことを示します。
    // オーバーサブスクリプションを使用すると、使用可能なハードウェア スレッドよりも多くのスレッドを作成できます。
    // これは、タスクの処理に追加のスレッドが必要になる可能性があるというヒントをタスク スケジューラに提供し、
    // 他のスレッドまたはローカル スレッド プール キューの作業項目の進行をスケジューラがブロックするのを防ぎます。
    _IdleTask = Task.Factory.StartNew( DrawTaskAsync, TaskCreationOptions.LongRunning );
//  _IdleTask = Task.Run( () => DrawTaskAsync() );
}

public async Task DrawTaskAsync()
{
    while ( !_FlagIdleTaskStop )
    {
        // 描画処理
        // コマンドリストを定義して描画しないと描画数が多いい場合に画面のちらつきが発生する
        using var cl = new CanvasCommandList( _Canvas.SwapChain );

        using var drawSessionA = _Canvas.SwapChain.CreateDrawingSession( Config.Panel.SheetColor.Color );
        using var drawSessionB = cl.CreateDrawingSession();

        var args = new CanvasDrawEventArgs( drawSessionB );

        _CurrentScreen.OnDraw( args );

        using var blur = new ScaleEffect
        { 
            Source  = cl,
            Scale   = Config.Panel.ScreenMagnification,
        };

        if ( blur != null )
        {
            drawSessionA.DrawImage( blur );
        }

        _Canvas.SwapChain.Present();

        await Task.Delay( 1 );
    }
}

private void UserControl_Unloaded( object aSender, RoutedEventArgs aArgs )
{
    // Win2D アンロード
    _Canvas.RemoveFromVisualTree();
    _Canvas = null;
}
```

## 画面ハードコピー

オフスクリーンに書き込んで、CanvasBitmap を作成後  
Bitmapへバッファコピーする。

```csharp
/// <summary>
/// 画面ハードコピー用オフスクリーン
/// </summary>
private CanvasRenderTarget? _Offscreen = null;

public void GetFrameStart()
{
    // オフスクリーン作成
    _Offscreen = new CanvasRenderTarget
        (
            CanvasDevice.GetSharedDevice(),
            DrawSet.ResolutionScreenWidth,
            DrawSet.ResolutionScreenHeight,
            Config.Window.DefaultDpi
        );
}

public CanvasBitmap? GetFrame( double aFrameTime )
{
    if ( _Offscreen == null )
    {
        return null;
    }

    using var cl = new CanvasCommandList( _Offscreen );

    //// 描画処理
    using var drawSessionA = _Offscreen.CreateDrawingSession();
    using var drawSessionB = cl.CreateDrawingSession();

    var args = new CanvasDrawEventArgs( drawSessionB );

    _ = ( _PlayerSurface?.OnDraw( args ) );

    drawSessionA.Clear( DrawSet.SheetColor.Color );

    using var blur = GetEffectImage( cl );
    if ( blur != null )
    {
        drawSessionA.DrawImage( blur );
    }

    // Bitmap作成
    return CanvasBitmap.CreateFromBytes
        (
            drawSessionA,
            _Offscreen.GetPixelBytes( 0, 0, (int)_Offscreen.SizeInPixels.Width, (int)_Offscreen.SizeInPixels.Height ),
            (int)_Offscreen.SizeInPixels.Width,
            (int)_Offscreen.SizeInPixels.Height,
            DirectXPixelFormat.R8G8B8A8UIntNormalized,
            Config.Window.DefaultDpi,
            CanvasAlphaMode.Premultiplied
        );
}

public void GetFrameEnd()
{
    _Offscreen?.Dispose();
    _Offscreen = null;
}

// CanvasBitmap形式で取得
using var frame = ControlAccess.UCPlayerPanel?.GetFrame( time );

// Bitmapへ変換
var buffer = frame.GetPixelBytes();

var bmpData = bmp.LockBits
(
    new( 0, 0, bmp.Width, bmp.Height ),
    ImageLockMode.WriteOnly,
    bmp.PixelFormat
);

Marshal.Copy( buffer, 0, bmpData.Scan0, buffer.Length );

bmp.UnlockBits( bmpData );
```
