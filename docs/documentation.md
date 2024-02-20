# Dokumentation für TabulatePlus

Diese Dokumentation bietet einen Überblick über die Kernkomponenten der AvaloniaApplication, um Entwicklern und Kunden einen schnellen Einstieg und Verständnis der Anwendung zu ermöglichen. Die Anwendung besteht aus verschiedenen Views und ViewModels, die zusammenarbeiten, um die Funktionalität der Anwendung zu realisieren.

--------

## Verwendete Technologien


### AvaloniaUI

AvaloniaUI ist ein plattformübergreifendes Framework für .NET, das die Entwicklung von Desktop-Anwendungen unter Windows, Linux und macOS ermöglicht. Es wird für die Gestaltung der Benutzeroberfläche verwendet.

### .NET 7.0

Als Basis der Anwendung dient das .NET 7.0 Framework, das eine moderne, leistungsfähige Umgebung für die Entwicklung von Anwendungen bietet.
CommunityToolkit.Mvvm
Diese Bibliothek wird für die Implementierung des MVVM (Model-View-ViewModel) Musters verwendet, um eine klare Trennung zwischen der Benutzeroberfläche und der Geschäftslogik zu gewährleisten.

### FluentAvaloniaUI

FluentAvaloniaUI ist eine Bibliothek, die Fluent Design System Komponenten für AvaloniaUI bereitstellt. Es wird verwendet, um der Anwendung ein modernes Aussehen und Gefühl zu geben.


### Microsoft.Extensions.DependencyInjection

Diese Bibliothek wird für die Dependency Injection verwendet, um die Verwaltung von Abhängigkeiten zwischen verschiedenen Teilen der Anwendung zu vereinfachen.


### CommunityToolkit.Labs.Extensions.DependencyInjection

Eine Erweiterung für die Dependency Injection, die zusätzliche Funktionalitäten für die Verwaltung von Services und ViewModels bietet.

## Hauptkomponenten
---------
## Views

Die Hauptansicht der Anwendung, die als Container für die anderen Views dient. Es verwendet ein SplitView-Layout, um eine Navigationsleiste und den Hauptinhalt zu trennen.

``` c# title="MainWindow.axaml"
     <Grid RowDefinitions="Auto, *">
        <Border Grid.Row="0" Height="32">
            <TextBlock Text="{Binding Title, RelativeSource={RelativeSource FindAncestor, AncestorType=Window }}"
                       VerticalAlignment="Center" Margin="10 0"/>
        </Border>
        <SplitView Grid.Row="1"
                   IsPaneOpen="{Binding IsPaneOpen}"
                   CompactPaneLength="46"
                   DisplayMode="CompactInline"
```    

Die Startseite der Anwendung, die als Einstiegspunkt dient. Sie enthält Elemente für die Interaktion mit dem Benutzer, wie das Hochladen von Verzeichnissen und das Anzeigen von Benachrichtigungen.

``` c# title="HomePageView.axaml"
   
            <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">  
            <PathIcon Data="{StaticResource Arrowuploadregular}" Margin="0 0 0 80"></PathIcon>
            </StackPanel>
            <TextBlock xml:space="preserve" VerticalAlignment="Center">
                Verzeichnis hierher ziehen
                oder zum Hochladen anklicken
            </TextBlock>
        <Canvas x:Name="DottedBorderCanvas" Width="300" Height="150" HorizontalAlignment="Center" VerticalAlignment="Center">
        </Canvas>
        </Grid>
        </StackPanel>
            <StackPanel>
            </StackPanel>
        <Button Name="CreateTableButton" Classes="tableGenerateButton" Padding="13" Cursor="Hand" Command="{Binding CreateTableCommand}" HorizontalAlignment="Center">
            <StackPanel Orientation="Vertical">
                <TextBlock>Tabelle Erstellen</TextBlock>
            </StackPanel> 
        </Button>
                 <StackPanel IsVisible="{Binding IsTableGeneratedSuccessfully}" Orientation="Horizontal" HorizontalAlignment="Center">
                     <TextBlock Text="Die Tabelle wurde erfolgreich generiert." FontSize="15" Foreground="#107c10" HorizontalAlignment="Center" Margin="0 0 0 10"/>
                     <PathIcon Data="{StaticResource Openregular}" Cursor="Hand" VerticalAlignment="Top" PointerPressed="OpenFolderIcon_PointerPressed"></PathIcon>
                 </StackPanel>
            <TextBlock Text="{Binding ErrorMessage}"  FontSize="15" TextWrapping="Wrap" Foreground="#d83b01" HorizontalAlignment="Center" Margin="0 0 0 10"/>
            <TextBlock Text="{Binding ErrorMessage}"  FontSize="15" TextWrapping="Wrap" Foreground="#d2d0ce" HorizontalAlignment="Center" Margin="0 0 0 10"/>
           <StackPanel Margin="20 0">
               <ToggleSwitch IsEnabled="{Binding IsButtonEnabled}"></ToggleSwitch>
                <ToggleSwitch IsEnabled="{Binding IsButtonEnabled}"></ToggleSwitch>
           </StackPanel>
``` 



## ViewModels

Das ViewModel für das MainWindow, das die Logik für die Navigation und das Zustandsmanagement der Anwendung enthält.

``` c# title="MainWindowViewModel.cs"
         [ObservableProperty]
    private bool _isPaneOpen = false;

    [ObservableProperty] 
    private ViewModelBase _currentPage;

    [ObservableProperty]
    private ListItemTemplate? _selectedListItem;
    
    public MainWindowViewModel(IErrorMessageService errorHandlingService, ITableProcessingService tableProcessingService)
    {
        _currentPage = new HomePageViewModel(tableProcessingService, errorHandlingService);
    }

    partial void OnSelectedListItemChanged(ListItemTemplate? value)
    {
        if (value is null) return;

        var instance = Design.IsDesignMode
            ? Activator.CreateInstance(value.ModelType)
            : Ioc.Default.GetService(value.ModelType);
```  


Das ViewModel für die HomePageView, das die Logik für das Hochladen von Verzeichnissen, die Erstellung von Tabellen und die Fehlerbehandlung enthält.

``` c# title="HomePageViewModel.cs"
    public partial class HomePageViewModel : ViewModelBase
    {
        private IErrorMessageService _errorMessageService;
        private string _errorMessage = "";
        private string _folderPath = "Kein Ordner ausgewählt";
        private string _notification = "";
        private bool _isFolderPathValid;
        private bool _isTableGeneratedSuccessfully = false;
        private readonly ITableProcessingService _tableProcessingService;
        private readonly IUserNotificationService _notificationService;

        public HomePageViewModel(ITableProcessingService tableProcessingService, IErrorMessageService errorMessageService)
        {
            //SetFolderPathCommand = new RelayCommand<string>(SetFolderPath);
            _errorMessageService = errorMessageService;
            _tableProcessingService = tableProcessingService;
        }
        [ObservableProperty]
        private bool _isButtonEnabled = true;
``` 



## Services

Ein Service, der Fehlermeldungen basierend auf einem Schlüssel bereitstellt. Wird im HomePageViewModel verwendet, um Benutzerfreundliche Fehlermeldungen anzuzeigen.

``` c# title="ErrorMessageService.cs"
    public partial class HomePageViewModel : ViewModelBase
    {
        private IErrorMessageService _errorMessageService;
        private string _errorMessage = "";
        private string _folderPath = "Kein Ordner ausgewählt";
        private string _notification = "";
        private bool _isFolderPathValid;
        private bool _isTableGeneratedSuccessfully = false;
        private readonly ITableProcessingService _tableProcessingService;
        private readonly IUserNotificationService _notificationService;

        public HomePageViewModel(ITableProcessingService tableProcessingService, IErrorMessageService errorMessageService)
        {
            //SetFolderPathCommand = new RelayCommand<string>(SetFolderPath);
            _errorMessageService = errorMessageService;
            _tableProcessingService = tableProcessingService;
        }
        [ObservableProperty]
        private bool _isButtonEnabled = true;
``` 

Ein Service, der die Logik für die Verarbeitung von Tabellen enthält. Wird im HomePageViewModel verwendet, um die Hauptlogik der Anwendung auszuführen.

``` c# title="TableProcessingService.cs"
    public class TableProcessingService : ITableProcessingService
     {
         private IUserNotificationService _userNotificationService;
        public TableProcessingService(IUserNotificationService userNotificationService)
        {
            this._userNotificationService = userNotificationService;
        }
        private List<DataPointConfig> dataPointConfigs = new List<DataPointConfig>
        {
            new DataPointConfig
            {
                Key = "Zeitpunkt",
                Formatter = (value) => $"t {value.Substring(0, 10)}_{value.Substring(10).Replace('_', ':')};;"
            },
            // Add more configurations here as needed for other dynamic data points
        };
``` 

## Dependency Injection

Die Anwendung verwendet Dependency Injection, um die Services und ViewModels zu verwalten. Die Konfiguration erfolgt in App.axaml.cs.

``` c# title="App.axaml.cs"

    [Singleton(typeof(MainWindowViewModel))]
    [Transient(typeof(HomePageViewModel))]
    [Transient(typeof(UserNotificationService))]
    [Transient(typeof(ErrorMessageService))]
    [Transient(typeof(TableProcessingService))]
    internal static partial void ConfigureViewModels(IServiceCollection services);

    [Singleton(typeof(MainWindow))]
    [Transient(typeof(HomePageView))]
    internal static partial void ConfigureViews(IServiceCollection services);
``` 


## Architektur

Die Anwendung folgt dem MVVM (Model-View-ViewModel) Architekturmuster, das eine klare Trennung zwischen der Benutzeroberfläche (Views) und der Geschäftslogik (ViewModels) ermöglicht. Services werden verwendet, um wiederverwendbare Logik und Funktionalitäten bereitzustellen, die von den ViewModels konsumiert werden.


## Navigation

Die Navigation zwischen den verschiedenen Views wird durch das MainWindowViewModel gesteuert, das auf Benutzeraktionen reagiert und das CurrentPage-Property entsprechend aktualisiert.