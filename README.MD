# Template Solution - Custom Windows UI

> **Important** - You cannot use a framework higher than DotNetFramework 4.0. 

#### Packages
A Package Manager is used to simplify Project References and keep them in a single point of access - `$(SolutionDir)\packages\Sybrin`.

In your Build Events (Project Properties -> Build Events -> Pre-Built Events) you should have the following:
>```
> if /I "$(ConfigurationName)" == "Debug" "$(SolutionDir)SybrinPackageManager.exe" /p "$(ProjectPath) " /s "$(SolutionDir) " /g "C:\SDKs\Sybrin10" "C:\SDKs\Sybrin7" 
> if /I "$(ConfigurationName)" == "Release" "$(SolutionDir)SybrinPackageManager.exe" /p "$(ProjectPath) " /s "$(SolutionDir) " /g "C:\SDKs\Sybrin10" "C:\SDKs\Sybrin7" 
> if /I "$(ConfigurationName)" == "Jenkins" "$(SolutionDir)SybrinPackageManager.exe" /p "$(ProjectPath) " /s "$(SolutionDir) " /g "C:\Jenkins\Sybrin10" "C:\Jenkins\Sybrin7" /v
>```

This tells Visual Studio to execute the `SybrinPackageManager.exe` application in the Solution Directory to update the References with the latest DLLs in your local SDK folder. It will automatically resolve missing Package References and keep them updated.

#### Post Build Events
The Post Build events kills the common applications that are responsible for locking the DLLs in the Bin path (`Sybrin Client`, `Sybrin Enterprise Manager`, and `PresentationHost`), and copies the dlls in your output directory, to the Bin. This just simplifies the process of copying the latest dlls from your process to your bin, in order to test/debug Enterprise Manager/Sybrin Client.

This is done through executing a `.bat` file that is found in your Solution Directory: `"$(SolutionDir)PostBuildEvents.bat"`

## Test Application
The Test Application will enable you to configure the settings for your Control and preview the changes on the fly, without the need to open Enterprise Manager to configure, and Client to preview the screen.

##### Required References
 * **Project Reference** - Add a reference to your project to be able to load its Main View in the Test App.
 * **PresentationCore** - Required for the NotifyChange behaviour
 * **Telerik.Windows.Controls** - Required for multiple Telerik Controls, including the BusyIndicator.
 * **Telerik.Windows.Themes.Windows8.dll** - Styling the Telerik Components
 * **System.Windows.Forms** - Required for using the PropertyGrid to configure your control.
 * **Sybrin10.Kernel** - Required for the Sybrin Enhanced PropertyGrid

Some dlls are not required as a reference, but will be needed in the output directory in order for the application to function normally:
 * **Sybrin.dll** - Required for creating an instance of the Sybrin Client and connecting the Sybrin 7 license.
 * **Xceed.Compression.dll & Xceed.Compressionn.Formats.dll** - Required for connecting the Client.
 * **BitMiracle.LibTiff.NET.dll** - Required for reading config? Not sure.

## Actual Control

WindowsUI controls require you to implement the Interface `ICustomControl` which resides in the `Sybrin10.Windows.UI` assembly.

##### Required References
 * **System.ComponentModel.Composition** - Required for Exporting class Metadata.
 * **Sybrin10.Windows.UI** - Contains the `ICustomControl` Interface required for WindowsUI to recognise the control.
 * **WindowsBase** - Required for the CustomControl class which you have to inherit from.
 * **Sybrin10.Kernel** - A lot of the base logic resides in this assembly. `ICustomControl` is also dependent on this assembly.
 * **Sybrin.Extensions** - Contains a lot of Extension methods that makes things easier to iterate through Arrays, Logging, etc.
 * **Newtonsoft.Json.dll** - Responsible for serialization classes and to ignore some properties for Serialization.
 * **System.Xaml** - To include WPF UserControls using XAML in your Project.
 * **WindowsFormsIntegration** - To include an ElementHost to launch your WPF view in a WindowsForms environment
 * **Telerik.Windows.Controls** - Required for multiple Telerik Controls, including the BusyIndicator.

##### Control.cs
This control will be added to a single pane in WindowsUI. It�s a visual control, but little to no logic is added to this control. The final visual control will be WPF, implementing the MVVM pattern to give the control a more modern look.

This control is used to bind to WindowsUI, and will be able to access all the Bindable Properties in WindowsUI, in order to communicate with all the other controls on the screen (Index Fields, Document Linking Control, etc.)

```cs
    public class MyControl : UserControl, ICustomControl, INotifyPropertyChanged {
        
    }
```

#### Base Classes Explained

##### CustomDescriptorBase
This class is responsible for decorating the PropertyGrid and giving it more flexibility in order to make it easier for a configurator to use. 

This includes `DynamicPropertyFilter`, `NotifyChange` events, etc.

# FAQ

**Question:** WindowsUI doesn�t allow me to save, saying 'Failed to get Parameter String'

**Answer:** Something fails to serialize. This can be caused by a few scenarios, including:
 * One of the Following Properties are not set correctly:
   * **ApplicationPath**: The Location where the DLL can be found. This is usually `Environment.SpecialFolder.LocalApplicationData`
   * **ApplicationObjectName**: This should be the Namespace of your Control
   * **ObjectClassName:** This should be your Object's Name, as it is in your Namespace.
   * **SubFolder:** This path will be appended to the `ApplicationPath`, and will indicate the exact full Path of the dll.
 
OR
 * A Property that exists on your base control that cannot be serialized. Mark the Property with the `[XmlIgnore, JsonIgnore]` attributes.

OR 
 * Your `Properties` object is null. Instantiate the Object

-----

**Question:** Trying to set a Pipe in WindowsUI's base throws an exception saying `Object Reference not set to an instance of an Object`

**Answer:** You are using a different Document that has been configured on the Pipe. You can only use the same document that you have selected in the Pipe, in all its processes.

----- 

**Question:** My object's `ToString()` does not work on the PropertyGrid

**Answer:** You must specify a Typeconverter on the object to be able to override its ToString():

```cs
[TypeConverter(typeof(TypeConverter))]
public MyObject MyObject = new MyObject();
```

----- 

**Question:** My View does not update when the DataContext changes

**Answer:** Either the View's dataContext is set incorrectly, or the property that you have changed does not implement the INotifyPropertyChanged interface correctly. The INotifyPropetyChanged interface is responsible for bubbling up the changed property back to the UI. A handy tool to inspect the DataContext of a view is Snoop. (https://snoopwpf.codeplex.com/)

-----

