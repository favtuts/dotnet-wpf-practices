# Basic MVVM and ICommand Usage Example
* https://tuts.heomi.net/basic-mvvm-and-icommand-usage-example/

# Introduction

We will learn about WPF Commands. Commands go very well with MVVM pattern (Model- View-ViewModel). We will also see how actually the view knows about its `ViewModel` and how the view invokes the operations on `ViewModel` which can be done by using Commands in WPF.

# Background

Let’s have a look at the MVVM architecture.

![MVVM_Architecture](./images/MVVM_image.png)


We use the standard conventions for naming the classes as follows:

* `Views` are suffixed with `View` after the name of the `View` (e.g.: `StudentListView`)
* `ViewModels` are suffixed with `ViewModel` after the name of the `ViewModel`. (e.g.: `StudentListViewModel`)
* `Models` are suffixed with `Model` after the name of the `Model` (e.g.: `StudentModel`).


# Using the Code

Let’s dive into the code and see a working example for MVVM and how to use commands in MVVM.

Create a new WPF project in Visual Studio. Rename the `MainWindow` as `MainWindowView` to follow up our conventions.

Next, we need to create a new class with name `MainWindowViewModel` that will act as the `ViewModel` for the `MainWindowView`.

What we do here in MVVM is that we tell the `View` about what its `ViewModel` will be. This can be done by setting the Data Context for that view. `Models` are used in the `ViewModel` and they do not have any connection between some specific views.

Example code for setting the `Datacontext` goes like this.

Open the `MainWindowView.xaml.cs` and set the data context as follows.

```xml
<Window x:Class="WpfExample.MainWindowView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MainWindow" Height="350" Width="525"
        xmlns:local="clr-namespace:WpfExample">

    <Window.DataContext>
        <local:MainWindowViewModel/>
    </Window.DataContext>

    <Grid>
    </Grid>
</Window>
```

Here the `local` is an alias to our namespace `WpfExample`. It’s required so that the framework knows where `MainWindowViewModel` is available.

Now we have set the MVVM pattern. Now the `MainWindowView` knows that’s its `ViewModel` is `MainWindowViewModel`.

We will verify this by using a simple binding.

Let's add a button to view and set the button content using a instance from `ViewModel`.

## The View
We will add the button to the view and we will set its content using binding as follows:

**MainWindowView.xaml.cs**
```xml
<Window x:Class=" WpfMvvmExample.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:local="clr-namespace:WpfMvvmExample"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MainWindow" Height="350" Width="525">

    <Window.DataContext>
        <local:MainWindowViewModel/>
    </Window.DataContext>

    <Grid>
        <Button Width="100" 
        Height="100" Content="{Binding ButtonContent}"/>
    </Grid>
</Window>
```

**The ViewModel**
```cs
namespace WpfExample
{
    class MainWindowViewModel
    {
        public string ButtonContent
        {
            get
            {
                return "Click Me";
            }
        }
    }
}
```

In the above code, we are telling the `View` to fetch the content of `button` from `ButtonContent` property present in the `ViewModel`.


Now if we run the application, we can see that the button content is set to `string “Click Me”`.

![View_Clickme_Button](./images/View-ClickMe-Button.jpg)

So this interprets that our MVVM is working properly.


# Now We Move In To the ICommand Interface

Now, let’s try to add click functionality to the button using Commands in WPF.

Commands provide a mechanism for the view to update the model in the MVVM architecture.

First, we have a look at the `ICommand` interface.

```cs
bool CanExecute(object parameter);
void Execute(object parameter);
event EventHandler CanExecuteChanged;
```

We will create a sample application that displays a message box with message “HI” when a button is clicked and we will add another button that toggles whether the Hi button can be clicked.

We create a class called `RelayCommand` which implements `ICommand` interface. This class acts as Enhancement for the `ICommand` and extracts the boiler plate code to a separate class.

```cs
public class RelayCommand : ICommand
   {
       private Action<object> execute;

       private Predicate<object> canExecute;

       private event EventHandler CanExecuteChangedInternal;

       public RelayCommand(Action<object> execute)
           : this(execute, DefaultCanExecute)
       {
       }

       public RelayCommand(Action<object> execute, Predicate<object> canExecute)
       {
           if (execute == null)
           {
               throw new ArgumentNullException("execute");
           }

           if (canExecute == null)
           {
               throw new ArgumentNullException("canExecute");
           }

           this.execute = execute;
           this.canExecute = canExecute;
       }

       public event EventHandler CanExecuteChanged
       {
           add
           {
               CommandManager.RequerySuggested += value;
               this.CanExecuteChangedInternal += value;
           }

           remove
           {
               CommandManager.RequerySuggested -= value;
               this.CanExecuteChangedInternal -= value;
           }
       }

       public bool CanExecute(object parameter)
       {
           return this.canExecute != null && this.canExecute(parameter);
       }

       public void Execute(object parameter)
       {
           this.execute(parameter);
       }

       public void OnCanExecuteChanged()
       {
           EventHandler handler = this.CanExecuteChangedInternal;
           if (handler != null)
           {
               //DispatcherHelper.BeginInvokeOnUIThread(() => handler.Invoke(this, EventArgs.Empty));
               handler.Invoke(this, EventArgs.Empty);
           }
       }

       public void Destroy()
       {
           this.canExecute = _ => false;
           this.execute = _ => { return; };
       }

       private static bool DefaultCanExecute(object parameter)
       {
           return true;
       }
   }
```

The `CommandManager.RequerySuggested` is responsible for enabling and disabling of "Click to Hii" button.

**TheView**

```xml
<Window x:Class="WpfExample.MainWindowView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MainWindow" Height="350" Width="525"
        xmlns:local="clr-namespace:WpfExample">

    <Window.DataContext>
        <local:MainWindowViewModel/>
    </Window.DataContext>

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition />
            <RowDefinition/>
        </Grid.RowDefinitions>

        <Button Grid.Row="0" Command="{Binding HiButtonCommand}" 
        CommandParameter="Hai" Content="{Binding HiButtonContent}"
                Width="100"
                Height="100"  />

        <Button Grid.Row="1" Content="Toggle Can Click" 
        Command="{Binding ToggleExecuteCommand}"  Width="100" Height="100"/>
    </Grid>

</Window>
```

**TheViewModel**

```cs
class MainWindowViewModel
    {
        private ICommand hiButtonCommand;

        private ICommand toggleExecuteCommand { get; set; }

        private bool canExecute = true;

        public string HiButtonContent
        {
            get
            {
                return "click to hi";
            }
        }

        public bool CanExecute
        {
            get
            {
                return this.canExecute;
            }

            set
            {
                if (this.canExecute == value)
                {
                    return;
                }

                this.canExecute = value;
            }
        }

        public ICommand ToggleExecuteCommand
        {
            get
            {
                return toggleExecuteCommand;
            }
            set
            {
                toggleExecuteCommand = value;
            }
        }

        public ICommand HiButtonCommand
        {
            get
            {
                return hiButtonCommand;
            }
            set
            {
                hiButtonCommand = value;
            }
        }

        public MainWindowViewModel()
        {
            HiButtonCommand = new RelayCommand(ShowMessage, param => this.canExecute);
            toggleExecuteCommand = new RelayCommand(ChangeCanExecute);
        }

        public void ShowMessage(object obj)
        {
            MessageBox.Show(obj.ToString());
        }

        public void ChangeCanExecute(object obj)
        {
            canExecute = !canExecute;
        }
    }
```

Final application looks like this:

![Final_App](./images/Final_app_Screenshot.jpg)