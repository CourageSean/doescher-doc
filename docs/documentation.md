# Dokumentation für Tabellen-Software

Diese Dokumentation bietet einen Überblick über die Kernkomponenten der AvaloniaApplication, um Entwicklern und Kunden einen schnellen Einstieg und Verständnis der Anwendung zu ermöglichen. Die Anwendung besteht aus verschiedenen Views und ViewModels, die zusammenarbeiten, um die Funktionalität der Anwendung zu realisieren.

--------

## Verwendete Technologien


### AvaloniaUI

Ein plattformübergreifendes Framework für .NET, ideal für die Entwicklung von Desktop-Anwendungen auf verschiedenen Betriebssystemen. Erlaubt eine flexible Gestaltung der Benutzeroberfläche mit XAML und unterstützt das MVVM-Muster für eine saubere Trennung von UI und Business-Logik.

### .NET 8.0

 Bietet eine moderne und leistungsfähige Entwicklungsumgebung. Nutzt aktuelle C#-Features und stellt eine breite Palette von Bibliotheken zur Verfügung, um Entwicklungsprozesse zu optimieren.


 ### CommunityToolkit.Mvvm
 
 Vereinfacht die Implementierung des MVVM-Designmusters, unterstützt die Entwicklung wartbarer und testbarer Anwendungen durch die Bereitstellung von Hilfsklassen für Bindings, Commands und ObservableProperties.

### FluentAvaloniaUI

FluentAvaloniaUI ist eine Bibliothek, die Fluent Design System Komponenten für AvaloniaUI bereitstellt. Es wird verwendet, um der Anwendung ein modernes Aussehen und Gefühl zu geben.


### Microsoft.Extensions.DependencyInjection

Diese Bibliothek wird für die Dependency Injection verwendet, um die Verwaltung von Abhängigkeiten zwischen verschiedenen Teilen der Anwendung zu vereinfachen.


### CommunityToolkit.Labs.Extensions.DependencyInjection

Eine Erweiterung für die Dependency Injection, die zusätzliche Funktionalitäten für die Verwaltung von Services und ViewModels bietet.

## Hauptkomponenten
---------
## Views

MainWindow: Die Hauptansicht agiert als Container für die anderen Views und setzt ein SplitView-Layout ein, um eine Navigationsleiste von dem Hauptinhalt zu trennen. Beispielcode demonstriert, wie AvaloniaUI für das Layout verwendet wird.

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

HomePageView: Dient als Startpunkt der Anwendung und bietet Benutzern die Möglichkeit, mit der Anwendung zu interagieren, z.B. durch das Hochladen von Dateien. Beispielcode zeigt die Verwendung von Event Bindings und UI-Elementen.

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

### Tabellengenerierung

Die Tabellengenerierung ist ein zentraler Bestandteil der Anwendung, der es Benutzern ermöglicht, aus verschiedenen Datenquellen Tabellen zu erstellen. Der Prozess umfasst typischerweise das Einlesen von Daten, deren Verarbeitung und die Ausgabe in einem strukturierten Format.

Ein Service, der Fehlermeldungen basierend auf einem Schlüssel bereitstellt. Wird im HomePageViewModel verwendet, um Benutzerfreundliche Fehlermeldungen anzuzeigen.

``` c# title="TableProcessingService.cs"
public class TableProcessingService : ITableProcessingService
{
    public Table GenerateTableFromData(IEnumerable<DataPoint> dataPoints)
    {
        var table = new Table();
        // Konfigurieren der Spalten basierend auf den DataPoint-Eigenschaften
        table.Columns.Add(new Column("ID", typeof(int)));
        table.Columns.Add(new Column("Wert", typeof(string)));
        table.Columns.Add(new Column("Datum", typeof(DateTime)));

        // Befüllen der Tabelle mit Daten
        foreach (var dataPoint in dataPoints)
        {
            var row = table.NewRow();
            row["ID"] = dataPoint.Id;
            row["Wert"] = dataPoint.Value;
            row["Datum"] = dataPoint.Timestamp;
            table.Rows.Add(row);
        }

        return table;
    }
}


```
In diesem Beispiel wird eine einfache Methode innerhalb eines TableProcessingService dargestellt, die eine Tabelle aus einer Sammlung von Datenpunkten erstellt. Es illustriert, wie man eine Tabelle mit Spalten definiert und dann Zeilen mit Daten aus den Datenpunkten hinzufügt.



### Graphgenerierung


Die Graphgenerierung ermöglicht es Benutzern, visuelle Darstellungen ihrer Daten zu erstellen. Dies kann durch die Verwendung von Datenvisualisierungsbibliotheken erfolgen, die in das Projekt integriert sind.

``` c# title="GraphDialogViewModel.cs"
 public GraphDialogViewModel(IEnumerable<double> pPmaxValues, IEnumerable<double> generatorValues, double startFreq,
        double stopFreq, string fileName)
    {
        double stepSize = (stopFreq - startFreq) / (pPmaxValues.Count() - 1);
        var pPmaxPoints = pPmaxValues.Select((value, index) => new ObservablePoint(startFreq + index * stepSize, value))
            .ToArray();
        var generatorPoints = generatorValues
            .Select((value, index) => new ObservablePoint(startFreq + index * stepSize, value)).ToArray();

        Series = new ISeries[]
        {
            new LineSeries<ObservablePoint>
            {
                Values = pPmaxPoints, Name = $"{fileName} - P/Pmax[%]", GeometrySize = 0, LineSmoothness = 0
            },
            new LineSeries<ObservablePoint>
            {
                Values = generatorPoints,
                Name = $"{fileName} - Generator[%]",
                GeometrySize = 0,
                LineSmoothness = 0
            }
        };

```


``` c# title="ErrorMessageService.cs"
 public class ErrorMessageService : IErrorMessageService
{
    public string GetErrorMessage(string messageKey)
    {
        return messageKey switch
        {
            "notDirectory" => "Die ausgewählte Datei entspricht keinem Verzeichnis. Bitte wählen Sie ein nicht leeres Verzeichnis aus.",
            "InvalidFolderPath" => "Verzeichnis fehlt. Bitte auswählen.",
            // Add more cases here as needed
            _ => "Unbekannter Fehler"
        };
    }
    
}
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


## Architektur und Designmuster

Die Anwendung folgt dem MVVM-Architekturmuster, das eine klare Trennung zwischen der Benutzeroberfläche (Views), der Geschäftslogik (ViewModels) und den Diensten (Services) gewährleistet. Diese Struktur fördert Wiederverwendbarkeit, Testbarkeit und Wartbarkeit. Dependency Injection spielt eine Schlüsselrolle in der Architektur, indem es das Management von Abhängigkeiten zwischen den Komponenten vereinfacht und ihre Kopplung reduziert.



## Navigation

Die Navigation innerhalb der Anwendung wird durch das MainWindowViewModel gesteuert, das auf Benutzerinteraktionen reagiert und entsprechend das CurrentPage-Property aktualisiert. Ein Beispielcode-Snippet könnte zeigen, wie die Navigation zwischen verschiedenen Views umgesetzt wird, einschließlich der Verwendung von Commands und der Reaktion auf Benutzeraktionen.