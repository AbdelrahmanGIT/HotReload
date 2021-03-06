# HotReload
Xamarin.Forms XAML hot reload, live reload, live xaml

![Sample GIF](https://github.com/AndreiMisiukevich/HotReload/blob/master/files/gf1.gif?raw=true)


## Setup
* Available on NuGet: [Xamarin.HotReload](http://www.nuget.org/packages/Xamarin.HotReload) [![NuGet](https://img.shields.io/nuget/v/Xamarin.HotReload.svg?label=NuGet)](https://www.nuget.org/packages/Xamarin.HotReload)
* Add nuget package to your Xamarin.Forms **NETSTANDARD**/**PCL** project and to platform-specific projects **iOS**, **Android** etc.
* Setup Reloader
```csharp
using Xamarin.Forms;

namespace YourNamespace
{
    public partial class App : Application
    {
        public App()
        {
#if DEBUG
            HotReloader.Current.Start();     
#endif
            this.InitComponent(InitializeComponent);
            MainPage = new NavigationPage(new MainPage());
        }
    }
}
```

* ALL XAML partial classes (ContentPage, ViewCell etc.) MUST be set up like:
```csharp
using Xamarin.Forms;

namespace YourNamespace
{
    public partial class MainPage : ContentPage
    {
        public MainPage()
        {
            this.InitComponent(InitializeComponent);
            //OR
//#if DEBUG
//            this.InitComponent();
//#else
//            InitializeComponent();
//#endif
        }
    }
}
```

* Download fresh version of **observer.exe** https://github.com/AndreiMisiukevich/HotReload/blob/master/files/observer.exe and put it in the root folder of your Xamarin.Forms NETSTANDARD/PCL project.
* Start observer.exe via terminal (for MAC) ```mono observer.exe``` or via command line (Windows) ```observer.exe```
* **IF observer.exe doesn't work**: build observer project yourself (Release mode) and find **exe** file in bin/release folder https://github.com/AndreiMisiukevich/HotReload/tree/master/Observer/Xamarin.Forms.HotReload.Observer 
* Optionaly you can set specific folder for observing files (if you didn't put observer.exe to the root folder) and specific device url for sending changes.
```mono observer.exe p=/Users/andrei/SpecificFolder/ u=http://192.168.0.3```
* Run your app and start developing with **HotReload**!

* **IMPORTANT**: make sure, that *reloader* and *observer* run on the same url. Check application output "HOTRELOADER STARTED AT {IP}" and compare it with url in terminal/cmd. Application output shows the IP of your device/emulator - observer must send it there. Also keep in mind, that your PC/Mac and device/emulator must be in the same local network.

* If you want to initialize your element after reloading (update named childs or something else), you should implement **IReloadable** interface. **OnLoaded** will be called every loading (INCLUDING FIRST! So you needn't duplicate code in constructor)

```csharp
public partial class MainPage : ContentPage, IReloadable
{
    public MainPage()
    {
        this.InitComponent(InitializeComponent);
    }

    public void OnLoaded() // Add event handlers in this method
    {
        Btn.Clicked += Handle_Clicked;
    }

    void Handle_Clicked(object sender, System.EventArgs e)
    {
        //Do work
    }
}
```

## Android Emulator
In case your `observer.exe` detects `xaml` changes but doesn't update in the emulator, you may need to forward the port to your `127.0.0.1`:

- Update your Application code to listen to the emulator `127.0.0.1`
```csharp
using Xamarin.Forms;

namespace YourNamespace
{
    public partial class App : Application
    {
        public App()
        {
#if DEBUG
            HotReloader.Current.Start("127.0.0.1",8000);
            // or
            // to listen to all possible ip addresses use
            //HotReloader.Current.Start("0.0.0.0",8000); 
#endif
            this.InitComponent(InitializeComponent);
            MainPage = new NavigationPage(new MainPage());
        }
    }
}
```
- You also need to forward the port to your pc/mac using `adb`

```
adb forward tcp:8000 tcp:8000
```

- Now you can run `observer.exe` with `u=http://127.0.0.1:8000` and all updates will be forwarded to the emultor 

## How does it work?
- Observer uses *FileSystemWatcher* for detecting all xaml files changes in specific folder and subfolders (by default it's current folder for observer.exe, but you can specify it). When observer detects that xaml file is updated, it sends http POST request with updated file to specified url (http://127.0.0.1:8000 by default).
- Reloader runs *TcpListener* at specified url (http://127.0.0.1:8000 by default). When reloader get POST request, it updates all related views.

Are there any questions? 🇧🇾 ***just ask me =)*** 🇧🇾

## License
The MIT License (MIT) see [License file](LICENSE)

## Contribution
Feel free to create issues and PRs 😃
