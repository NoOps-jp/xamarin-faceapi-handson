# Module2: UIの作成

UIをXAMLで記述します

## 1. XAMLファイルの記述

UIを追加します。ソリューションで`MainPage.xaml`を開いてください。

つぎのコードを記述します。

`HowOld.Xamarin`となっている部分はご自身で追加したプロジェクト名に適宜書き換えてください。

```
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
  xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
  xmlns:local="clr-namespace:HowOld.Xamarin"
  xmlns:conv="clr-namespace:HowOld.Xamarin.
    Converters;assembly=FaceEmotionRecognition"
    x:Class="HowOld.Xamarin.MainPage">
  <StackLayout>
    <StackLayout>
      <Image x:Name="Image1" HeightRequest="240" />
      <ActivityIndicator x:Name="Indicator1" HeightRequest="20" IsRunning="False" />
    </StackLayout>
    <StackLayout VerticalOptions="EndAndExpand">
      <StackLayout Orientation="Horizontal" Padding="3">
        <Label Text="性別: "/>
        <Label x:Name="GenderLabel" Text="{Binding Path=Gender}" />
      </StackLayout>
      <StackLayout Orientation="Horizontal" Padding="3">
        <Label Text="年齢: "/>
        <Label x:Name="AgeLabel" Text="{Binding Path=Age}"/>
      </StackLayout>
      <StackLayout Orientation="Horizontal" Padding="3">
        <Label Text="感情: "/>
        <Label x:Name="EmotionLabel" Text="{Binding Path=Emotion}"/>
      </StackLayout>
      <StackLayout Orientation="Horizontal" Padding="3">
        <Label Text="笑顔: "/>
        <Label x:Name="SmileLabel"
               Text="{Binding Path=Smile}"/>
      </StackLayout>
      <StackLayout Orientation="Horizontal" Padding="3">
        <Label Text="眼鏡: "/>
        <Label x:Name="GlassesLabel" Text="{Binding Path=Glasses}"/>
      </StackLayout>
      <StackLayout Orientation="Horizontal" Padding="3">
        <Label Text="ひげ: "/>
        <Label x:Name="BeardLabel"
               Text="{Binding Path=Beard}"/>
      </StackLayout>
      <StackLayout Orientation="Horizontal" Padding="3">
        <Label Text="口ひげ: "/>
        <Label x:Name="MoustacheLabel"
               Text="{Binding Path=Moustache}"/>
      </StackLayout>
    </StackLayout>
    <Grid VerticalOptions="EndAndExpand">
      <Grid.ColumnDefinitions>
        <ColumnDefinition />
        <ColumnDefinition />
      </Grid.ColumnDefinitions>
      <Button x:Name="TakePictureButton" Grid.Column="0" Clicked="TakePictureButton_Clicked" Text="Take from camera"/>
      <Button x:Name="UploadPictureButton" Grid.Column="1" Clicked="UploadPictureButton_Clicked" Text="Pick a photo"/>
    </Grid>
  </StackLayout>
</ContentPage>
```

---
[Back](module1.md) | [Next](module3.md)