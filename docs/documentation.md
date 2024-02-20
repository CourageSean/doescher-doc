# Dokumentation TabulatePlus

## Code Annotation 

### Codeblocks

Some `code` goes here

### Plain codeblock

A plain  codeblock:

``` c# title="TableProcessingService.cs" linenums="38"
  public async Task<string> ProcessFilesInFolderAsync(string folderPath)
        {
            var files = Directory.GetFiles(folderPath, "*.txt").Where(f => !f.Contains("table")).ToArray();
            var tableFiles =  Directory.GetFiles(folderPath, "*.txt").Where(f => f.Contains("table") && f.Contains($"{files.Length}")).ToArray();
           // Check if table allready exist in folder
            if (tableFiles.Length > 0)
            { 
                return _userNotificationService.GetNotificationMessage("tableAllreadyExist");
                
            }
            var aggregatedData = new Dictionary<string, List<string>>();

            // Initialize aggregatedData with keys from dataPointConfigs and explicitly for MW and MWD
            foreach (var config in dataPointConfigs)
            {
                aggregatedData.TryAdd(config.Key, new List<string>());
            }

            aggregatedData.TryAdd("Zeitpunkt", new List<string>());
            aggregatedData.TryAdd("MW", new List<string>());
            aggregatedData.TryAdd("MWD", new List<string>()); 

            // Initialize a separate structure for P/PmaxGenerator to keep track of data by file
            var ppmaxGeneratorData = new Dictionary<string, List<string>>();

            foreach (var file in files)
            {
                var lines = await File.ReadAllLinesAsync(file);
                var processedData = ProcessFileContent(lines);

                foreach (var key in processedData.Keys)
                {
                    if (key == "P/PmaxGenerator")
                    {
```    