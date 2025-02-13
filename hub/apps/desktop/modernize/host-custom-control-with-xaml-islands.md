---
description: This article demonstrates how to host a custom UWP control in a WPF app by using XAML Islands.
title: Host a custom UWP control in a WPF app using XAML Islands
ms.date: 08/20/2019
ms.topic: article
keywords: windows 10, uwp, windows forms, wpf, xaml islands, custom controls, user controls, host controls
ms.author: mcleans
author: mcleanbyron
ms.localizationpriority: medium
ms.custom: 19H1
---

# Host a custom UWP control in a WPF app using XAML Islands

This article demonstrates how to use the [WindowsXamlHost](https://docs.microsoft.com/windows/communitytoolkit/controls/wpf-winforms/windowsxamlhost) control in the Windows Community Toolkit to host a custom UWP control in a WPF app that targets .NET Core 3. The custom control contains several first-party UWP controls from the Windows SDK and binds a property in one of the UWP controls to a string in the WPF app. This article also demonstrates how to also host a first-party UWP control from the [WinUI library](https://docs.microsoft.com/uwp/toolkits/winui/).

Although this article demonstrates how to do this in a WPF app, the process is similar for a Windows Forms app. For an overview about hosting UWP controls in WPF and Windows Forms apps, see [this article](xaml-islands.md#wpf-and-windows-forms-applications).

## Overview

To host a custom UWP control in a WPF app, you'll need the following components. This article provides instructions for creating each of these components.

* **The project and source code for your WPF app**. Using the [WindowsXamlHost](https://docs.microsoft.com/windows/communitytoolkit/controls/wpf-winforms/windowsxamlhost) control to host custom UWP controls is supported only in WPF and Windows Forms apps that target .NET Core 3. This scenario is not supported in apps that target the .NET Framework.

* **The custom UWP control**. You'll need the source code for the custom UWP control you want to host so you can compile it with your app. Typically, the custom control is defined in a UWP class library project that you reference in the same solution as your WPF (or Windows Forms) project.

* **A UWP app project that defines a XamlApplication object**. Your WPF (or Windows Forms) project must have access to an instance of the `Microsoft.Toolkit.Win32.UI.XamlHost.XamlApplication` class provided by the Windows Community Toolkit. This object acts as a root metadata provider for loading metadata for custom UWP XAML types in assemblies in the current directory of your application. The recommended way to do this is to add a **Blank App (Universal Windows)** project to the same solution as your WPF (or Windows Forms) project and revise the default `App` class in this project.
  > [!NOTE]
  > Your solution can contain only one project that defines a `XamlApplication` object. All custom UWP controls in your app share the same `XamlApplication` object. The project that defines the `XamlApplication` object must include references to all other UWP libraries and projects that are used host UWP controls in the XAML Island.

## Create a WPF project

Before getting started, follow these instructions to create a WPF project and configure it to host XAML Islands. If you have an existing WPF project, you can adapt these steps and code examples for your project.

> [!NOTE]
> If you have an existing project that targets the .NET Framework, you'll need to migrate your project to .NET Core 3. For more information, see [this blog series](https://devblogs.microsoft.com/dotnet/migrating-a-sample-wpf-app-to-net-core-3-part-1/).

1. If you haven't done so already, install the latest version of the [.NET Core 3 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0).

2. In Visual Studio 2019, create a new **WPF App (.NET Core)** project.

3. Make sure [package references](https://docs.microsoft.com/nuget/consume-packages/package-references-in-project-files) are enabled:

    1. In Visual Studio, click **Tools -> NuGet Package Manager -> Package Manager Settings**.
    2. Make sure **PackageReference** is selected for **Default package management format**.

4. Right-click your WPF project in **Solution Explorer** and choose **Manage NuGet Packages**.

5. In the **NuGet Package Manager** window, make sure that **Include prerelease** is selected.

6. Select the **Browse** tab, search for the [Microsoft.Toolkit.Wpf.UI.XamlHost](https://www.nuget.org/packages/Microsoft.Toolkit.Wpf.UI.XamlHost) package (version v6.0.0-preview7 or later), and install the package. This package provides everything you need to use the **WindowsXamlHost** control to host a UWP control, including other related NuGet packages.
    > [!NOTE]
    > Windows Forms apps must use the [Microsoft.Toolkit.Forms.UI.XamlHost](https://www.nuget.org/packages/Microsoft.Toolkit.Forms.UI.XamlHost) package (version v6.0.0-preview7 or later).

7. Configure your solution to target a specific platform such as x86 or x64. Custom UWP controls are not supported in projects that target **Any CPU**.

    1. In **Solution Explorer**, right-click the solution node and select **Properties** -> **Configuration Properties** -> **Configuration Manager**. 
    2. Under **Active solution platform**, select **New**. 
    3. In the **New Solution Platform** dialog, select **x64** or **x86** and press **OK**. 
    4. Close the open dialog boxes.

## Create a XamlApplication object in a UWP app project

Next, add a UWP app project to the same solution as your WPF project. You will revise the default `App` class in this project to derive from the `Microsoft.Toolkit.Win32.UI.XamlHost.XamlApplication` class provided by the Windows Community Toolkit. The **WindowsXamlHost** object in your WPF app needs this `XamlApplication` object to host custom UWP controls.

1. In **Solution Explorer**, right-click the solution node and select **Add** -> **New Project**.
2. Add a **Blank App (Universal Windows)** project to your solution. Make sure the target version and minimum version are both set to **Windows 10, version 1903** or later.
3. In the UWP app project, install the [Microsoft.Toolkit.Win32.UI.XamlApplication](https://www.nuget.org/packages/Microsoft.Toolkit.Win32.UI.XamlApplication) NuGet package (version v6.0.0-preview7 or later).
4. Open the **App.xaml** file and replace the contents of this file with the following XAML. Replace `MyUWPApp` with the namespace of your UWP app project.

    ```xml
    <xaml:XamlApplication
        x:Class="MyUWPApp.App"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:xaml="using:Microsoft.Toolkit.Win32.UI.XamlHost"
        xmlns:local="using:MyUWPApp">
    </xaml:XamlApplication>
    ```

5. Open the **App.xaml.cs** file and replace the contents of this file with the following code. Replace `MyUWPApp` with the namespace of your UWP app project.

    ```csharp
    namespace MyUWPApp
    {
        sealed partial class App : Microsoft.Toolkit.Win32.UI.XamlHost.XamlApplication
        {
            public App()
            {
                this.Initialize();
            }
        }
    }
    ```

6. Delete the **MainPage.xaml** file from the UWP app project.
7. Build the UWP app project.
8. In your WPF project, right-click the **Dependencies** node and add a reference to your UWP app project.

## Create a custom UWP control

To host a custom UWP control in your WPF app, you must have the source code for the control so you can compile it with your app. Typically custom controls are defined in a UWP class library project for easy portability.

In this section, you will define a simple custom UWP control in a new class library project. You can alternatively define the custom UWP control in the UWP app project you created in the previous section. However, these steps do this in a separate class library project for illustrative purposes because this is typically how custom controls are implemented for portability.

If you already have a custom control, you can use it instead of the control shown here. However, you'll still need to configure the project that contains the control as shown in these steps.

1. In **Solution Explorer**, right-click the solution node and select **Add** -> **New Project**.
2. Add a **Class Library (Universal Windows)** project to your solution. Make sure the target version and minimum version are both set to **Windows 10, version 1903** or later.
3. Right-click the project file and select **Unload Project**. Right-click the project file again and select **Edit**.
4. Before the closing `</Project>` element, add the following XML to disable several properties and then save the project file. These properties must be enabled to host the custom UWP control in a WPF (or Windows Forms) app.

    ```xml
    <PropertyGroup>
      <EnableTypeInfoReflection>false</EnableTypeInfoReflection>
      <EnableXBindDiagnostics>false</EnableXBindDiagnostics>
    </PropertyGroup>
    ```

5. Right-click the project file and select **Reload Project**.
6. Delete the default **Class1.cs** file and add a new **User Control** item to the project.
7. In the XAML file for the user control, add the following `StackPanel` as a child of the default `Grid`. This example adds a ``TextBlock`` control and then binds the ``Text`` attribute of that control to the ``XamlIslandMessage`` field.

    ```xml
    <StackPanel Background="LightCoral">
        <TextBlock>This is a simple custom UWP control</TextBlock>
        <Rectangle Fill="Blue" Height="100" Width="100"/>
        <TextBlock Text="{x:Bind XamlIslandMessage}" FontSize="50"></TextBlock>
    </StackPanel>
    ```

8. In the code-behind file for the user control, add the `XamlIslandMessage` field to the user control class as shown below.

    ```csharp
    public sealed partial class MyUserControl : UserControl
    {
        public string XamlIslandMessage { get; set; }

        public MyUserControl()
        {
            this.InitializeComponent();
        }
    }
    ```

9. Build the UWP class library project.
10. In your WPF project, right-click the **Dependencies** node and add a reference to the UWP class library project.
11. In the UWP app project you configured earlier, right-click the **References** node and add a reference to the UWP class library project.
12. Rebuild the entire solution and make sure all the projects build successfully.

## Host the custom UWP control in your WPF app

1. In **Solution Explorer**, expand the WPF project and open the MainWindow.xaml file or some other window in which you want to host the custom control.
2. In the XAML file, add the following namespace declaration to the `<Window>` element.

    ```xml
    xmlns:xaml="clr-namespace:Microsoft.Toolkit.Wpf.UI.XamlHost;assembly=Microsoft.Toolkit.Wpf.UI.XamlHost"
    ```

3. In the same file, add the following control to the `<Grid>` element. Change the `InitialTypeName` attribute to the fully qualified name of the user control in your UWP class library project.

    ```xml
    <xaml:WindowsXamlHost InitialTypeName="UWPClassLibrary.MyUserControl" ChildChanged="WindowsXamlHost_ChildChanged" />
    ```

4. Open the code-behind file and add the following code to the `Window` class. This code defines a `ChildChanged` event handler that assigns the value of the ``XamlIslandMessage`` field of the UWP custom control to the value of the `WPFMessage` field in the WPF app. Change `UWPClassLibrary.MyUserControl` to the fully qualified name of the user control in your UWP class library project.

    ```csharp
    private void WindowsXamlHost_ChildChanged(object sender, EventArgs e)
    {
        // Hook up x:Bind source.
        global::Microsoft.Toolkit.Wpf.UI.XamlHost.WindowsXamlHost windowsXamlHost =
            sender as global::Microsoft.Toolkit.Wpf.UI.XamlHost.WindowsXamlHost;
        global::UWPClassLibrary.MyUserControl userControl =
            windowsXamlHost.GetUwpInternalObject() as global::UWPClassLibrary.MyUserControl;

        if (userControl != null)
        {
            userControl.XamlIslandMessage = this.WPFMessage;
        }
    }

    public string WPFMessage
    {
        get
        {
            return "Binding from WPF to UWP XAML";
        }
    }
    ```

6. Build and run your app and confirm that the UWP user control displays as expected.

## Add a control from the WinUI library to the custom control

Traditionally, UWP controls have been released as part of the Windows 10 OS and made available to developers through the Windows SDK. The [WinUI library](https://docs.microsoft.com/uwp/toolkits/winui/) is an alternative approach, where updated versions of the first-party UWP controls from the Windows SDK are distributed in a NuGet package that is not tied to Windows SDK releases. This library also includes new controls that aren't part of the Windows SDK and the default UWP platform. See our [WinUI library roadmap](https://github.com/microsoft/microsoft-ui-xaml/blob/master/docs/roadmap.md) for more details.

This section demonstrates how to add a UWP control from the WinUI library to your user control so you can host this control in your WPF app. 

1. In the UWP app project, install the latest prerelease version of the [Microsoft.UI.Xaml](https://www.nuget.org/packages/Microsoft.UI.Xaml) NuGet package.
    > [!NOTE]
    > Make sure you install the latest *prerelease* version. Currently, only prerelease versions of this package will work if you choose to package your app in an [MSIX package](https://docs.microsoft.com/windows/msix) for deployment.

2. In the App.xaml file in this project, add the following child element to the `<xaml:Application>` element.

    ```xml
    <Application.Resources>
        <XamlControlsResources xmlns="using:Microsoft.UI.Xaml.Controls" />
    </Application.Resources>
    ```

    After adding this element, the contents of this file should now look similar to this.

    ```xml
    <xaml:XamlApplication
        x:Class="MyUWPApp.App"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:xaml="using:Microsoft.Toolkit.Win32.UI.XamlHost"
        xmlns:local="using:MyUWPApp">
        <Application.Resources>
            <XamlControlsResources xmlns="using:Microsoft.UI.Xaml.Controls" />
        </Application.Resources>
    </xaml:XamlApplication>
    ```

3. In the UWP class library project, install the latest prerelease version of the [Microsoft.UI.Xaml](https://www.nuget.org/packages/Microsoft.UI.Xaml) NuGet package (the same version that you installed in the UWP app project).

4. In the same project, open the XAML file for the user control and add the following namespace declaration to the `<UserControl>` element.

    ```xml
    xmlns:winui="using:Microsoft.UI.Xaml.Controls"
    ```

5. In the same file, add a `<winui:RatingControl />` element as a child of the `<StackPanel>`. This element adds an instance of the [RatingControl](https://docs.microsoft.com/uwp/api/microsoft.ui.xaml.controls.ratingcontrol?view=winui-2.2) class from the WinUI library. After adding this element, the `<StackPanel>` should now look similar to this.

    ```xml
    <StackPanel Background="LightCoral">
        <TextBlock>This is a simple custom UWP control</TextBlock>
        <Rectangle Fill="Blue" Height="100" Width="100"/>
        <TextBlock Text="{x:Bind XamlIslandMessage}" FontSize="50"></TextBlock>
        <winui:RatingControl />
    </StackPanel>
    ```

6. Build and run your app and confirm that the new rating control displays as expected.

## Package the app

You can optionally package the WPF app in an [MSIX package](https://docs.microsoft.com/windows/msix) for deployment. MSIX is the modern app packaging technology for Windows, and it is based on a combination of MSI, APPX, App-V and ClickOnce installation technologies.

The following instructions show you how to package the all the components in the solution in an MSIX package by using the [Windows Application Packaging Project](https://docs.microsoft.com/windows/msix/desktop/desktop-to-uwp-packaging-dot-net) in Visual Studio 2019. These steps are necessary only if you want to package the WPF app in an MSIX package. Note that these steps currently include some workarounds specific to the scenario of hosting custom UWP controls.

1. Add a new [Windows Application Packaging Project](https://docs.microsoft.com/windows/msix/desktop/desktop-to-uwp-packaging-dot-net) to your solution. As you create the project, select **Windows 10, version 1903 (10.0; Build 18362)** for both the **Target version** and **Minimum version**.

2. In the packaging project, right-click the **Applications** node and choose **Add reference**. In the list of projects, select the WPF project in your solution and click **OK**.

3. Edit the packaging project file. These changes are currently required for packaging WPF apps that target .NET Core 3 and that host XAML Islands.

    1. In Solution Explorer, right-click the packaging project node and select **Edit Project File**.
    2. Locate the `<Import Project="$(WapProjPath)\Microsoft.DesktopBridge.targets" />` element in the file. Replace this element with the following XML. These changes are currently required for packaging WPF apps that target .NET Core 3 and that host UWP controls.

        ``` xml
        <ItemGroup>
            <SDKReference Include="Microsoft.VCLibs,Version=14.0">
            <TargetedSDKConfiguration Condition="'$(Configuration)'!='Debug'">Retail</TargetedSDKConfiguration>
            <TargetedSDKConfiguration Condition="'$(Configuration)'=='Debug'">Debug</TargetedSDKConfiguration>
            <TargetedSDKArchitecture>$(PlatformShortName)</TargetedSDKArchitecture>
            <Implicit>true</Implicit>
            </SDKReference>
        </ItemGroup>
        <Import Project="$(WapProjPath)\Microsoft.DesktopBridge.targets" />
        <Target Name="_StompSourceProjectForWapProject" BeforeTargets="_ConvertItems">
            <ItemGroup>
            <_TemporaryFilteredWapProjOutput Include="@(_FilteredNonWapProjProjectOutput)" />
            <_FilteredNonWapProjProjectOutput Remove="@(_TemporaryFilteredWapProjOutput)" />
            <_FilteredNonWapProjProjectOutput Include="@(_TemporaryFilteredWapProjOutput)">
                <SourceProject></SourceProject>
                <TargetPath Condition="'%(FileName)%(Extension)'=='resources.pri'">app_resources.pri</TargetPath>
            </_FilteredNonWapProjProjectOutput>
            </ItemGroup>
        </Target>
        ```

    3. Save the project file and close it.

4. Edit the package manifest to reference the correct default splash screen image. This workaround is currently required for packaging WPF apps that host custom UWP controls.

    1. In the packaging project, right-click the **Package.appxmanifest** file and click **View Code**.
    2. Locate the following element in the file.

        ```<uap:SplashScreen Image="Images\SplashScreen.png" />```

    3. Change this element to:

        ```<uap:SplashScreen Image="Images\SplashScreen.scale-200.png" />```

    4. Save the **Package.appxmanifest** file and close it.

5. Edit the WPF project file. These changes are currently required for packaging WPF apps that host custom UWP controls.

    1. In Solution Explorer, right-click the WPF project node and select **Unload Project**.
    2. Right-click the WPF project node and select **Edit**.
    3. Locate the last `</PropertyGroup>` closing tag in the file, and add the following XML immediately after that tag.

        ``` xml
        <PropertyGroup>
          <AssetTargetFallback>uap10.0.18362</AssetTargetFallback>
        </PropertyGroup>
        ```

    4. Save the project file and close it.
    5. Right-click the WPF project node and choose **Reload Project**.

6. Build and run the packaging project. Confirm that the WPF runs and the UWP custom control displays as expected.

## Related topics

* [UWP controls in desktop applications](xaml-islands.md)
* [WindowsXamlHost](https://docs.microsoft.com/windows/communitytoolkit/controls/wpf-winforms/windowsxamlhost)
