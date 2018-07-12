# Module3: 写真判定機能の開発

## 1. Face APIのレスポンスをデシリアライズするクラス追加
Face APIからのレスポンスをデシリアライズするクラス（`FaceDetection`クラス）を追加します。

共通プロジェクトを右クリックし、**追加** -> **新しい項目**を選択してください。

つぎに、**Visual C#アイテム** -> **クラス**を選択し、`FaceDetection.cs`を入力して**追加**をクリックします。

`FaceDetection.cs`につぎのコードを記述します。`namespace`はご自身のプロジェクト名のまま変更しないでください。

```
namespace HowOld.Xamarin
{
    public class FaceDetection
    {
        public string Emotion { get; set; }
        public double Smile { get; set; }
        public string Glasses { get; set; }
        public string Gender { get; set; }
        public double Age { get; set; }
        public double Beard { get; set; }
        public double Moustache { get; set; }
    }
}
```

## 2. APIを使用するロジックの追加

`MainPage.xaml.cs`を開いてください。

まず`using`を追加してください。

```
using Microsoft.ProjectOxford.Face;
using Plugin.Media.Abstractions;
using Plugin.Media;
```

更につぎのコードを記述します。

```
public partial class MainPage : ContentPage
{
    private readonly IFaceServiceClient _faceServiceClient;

    public MainPage()
    {
        InitializeComponent();
        // 書き換えが必要です
        _faceServiceClient = new FaceServiceClient([取得したAPIキーを記述], [Endpoint URL]);
    }

    private async Task<FaceDetection> DetectFaceAsync(MediaFile inputFile)
    {
        try
        {
            var faces = await _faceServiceClient.DetectAsync(inputFile.GetStream(), false, false, (FaceAttributeType[])Enum.GetValues(typeof(FaceAttributeType)));

            if (faces.Length == 0) {
                throw new Exception("顔を認識できませんでした。別の写真を試してください。");
            }

            var faceAttributes = faces[0]?.FaceAttributes;
            var faceDetection = new FaceDetection();
            faceDetection.Age = faceAttributes.Age;
            faceDetection.Emotion = faceAttributes.Emotion.ToRankedList().FirstOrDefault().Key;
            faceDetection.Glasses = faceAttributes.Glasses.ToString();
            faceDetection.Smile = faceAttributes.Smile;
            faceDetection.Gender = faceAttributes.Gender;
            faceDetection.Moustache = faceAttributes.FacialHair.Moustache;
            faceDetection.Beard = faceAttributes.FacialHair.Beard;

            return faceDetection;
        }
        catch(Exception ex)
        {
            await DisplayAlert("Error", ex.Message, "OK");
            return null;
        }
    }
}
```

[取得したAPIキーを記述]と書いてある部分には[module0](module0.md)で取得したFace APIのAPIキーを記述してください。

[Endpoint URL]と書いてある部分にはAPIのエンドポイントのURLを記述してください。東アジアにサービスをデプロイした場合は

`https://eastasia.api.cognitive.microsoft.com/face/v1.0`

になります。

## 3. 撮影した写真を判定するロジック追加

同`MainPage.xaml.cs`ファイルにつぎのコードを記述してください。

```
private async void TakePictureButton_Clicked(object sender, EventArgs e)
{
    await CrossMedia.Current.Initialize();
    if (!CrossMedia.Current.IsCameraAvailable || !CrossMedia.Current.
      IsTakePhotoSupported)
    {
        await DisplayAlert("カメラが使用できません", "この端末はカメラの使用をサポートしていません", "OK");
        return;
    }
    var file = await CrossMedia.Current.TakePhotoAsync(new StoreCameraMediaOptions
    {
        SaveToAlbum = true,
        Name = "test.jpg"
    });
    if (file == null)
        return;
    Indicator1.IsRunning = true;
    Image1.Source = ImageSource.FromStream(() => file.GetStream());
    FaceDetection theData = await DetectFaceAsync(file);
    BindingContext = theData;
    Indicator1.IsRunning = false;
}
```

## 4. 選択した写真を判定するロジック追加

同`MainPage.xaml.cs`ファイルにつぎのコードを記述してください。

```
private async void UploadPictureButton_Clicked(object sender, EventArgs e)
{
    if (!CrossMedia.Current.IsPickPhotoSupported)
    {
        await DisplayAlert("アップロードできません", "この端末は写真の選択をサポートしていません", "OK");
        return;
    }
    var file = await CrossMedia.Current.PickPhotoAsync();
    if (file == null)
        return;
    Indicator1.IsRunning = true;
    Image1.Source = ImageSource.FromStream(() => file.GetStream());
    FaceDetection theData = await DetectFaceAsync(file);
    BindingContext = theData;
    Indicator1.IsRunning = false;
}
```


## 5. パーミッションの設定

アプリで写真関連の機能を使用するためにパーミッションを設定します。

### Android

Androidプロジェクトを右クリックし、**プロパティ**をクリックしてください。

Androidマニフェストでつぎの3つのアクセスをチェックしてください。

* CAMERA
* READ_EXTERNAL_STORAGE
* WRITE_EXTERNAL_STORAGE

また、直接**Property**フォルダ内の`AndroidManifest.xml`を編集します。

<application>タグ内につぎのタグを記述してください。

```
<provider android:name="android.support.v4.content.FileProvider"
          android:authorities="${applicationId}.fileprovider"
          android:exported="false"
          android:grantUriPermissions="true">
	  <meta-data android:name="android.support.FILE_PROVIDER_PATHS"
                     android:resource="@xml/file_paths"></meta-data>
</provider>
```

更に、**Resources**フォルダの中に**xml**フォルダを作成し、`file_paths.xml`という名前でファイルを作成してください。

`file_paths.xml`の中身はつぎのように記述してください。

```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
  <external-files-path name="my_images" path="Pictures" />
  <external-files-path name="my_movies" path="Movies" />
</paths>
```

`MainActivity.cs`を開いてください。

まず`using`を追加してください。

```
using Plugin.CurrentActivity;
using Plugin.Permissions;
```

つぎのコードを追加してください。

```
public override void OnRequestPermissionsResult(int requestCode, string[] permissions, [GeneratedEnum] Permission[] grantResults)
{
    base.OnRequestPermissionsResult(requestCode, permissions, grantResults);
    PermissionsImplementation.Current.OnRequestPermissionsResult(requestCode, permissions, grantResults);
}
```

`OnCreate`メソッドの`base.OnCreate(bundle);`の直後につぎのコードを追加してください。

```
CrossCurrentActivity.Current.Init(this, bundle);
```

`AssemblyInfo.cs`を開き、つぎのコードを追加してください。

```
[assembly: UsesFeature("android.hardware.camera", Required = false)]
[assembly: UsesFeature("android.hardware.camera.autofocus", Required = false)]
```

### iOS

iOSプロジェクトの`info.plist`をエディタで開き、`<dict>`タグ内につぎの`key`と`string`を記述してください。

```
<key>NSCameraUsageDescription</key>
<string>This app needs access to the camera to take photos.</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>This app needs access to photos.</string>
<key>NSPhotoLibraryAddUsageDescription</key>
<string>This app needs access to the photo gallery.</string>
```

---
[Back](module2.md) | [Next](module4.md)