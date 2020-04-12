
# 3 Refactoring to an Immutable Architecture

## Vocabulary Used

Immutability => inability to Change Data
State => Data That changes over time
Side Effect => Change that is made to some state

## Why Does Immutability Matter?

Mutable operations = Dishonest code

*   increased readability
*   Single place for validation invariants
*   Automatic thread safety

## Mutability and Temporal Coupling

=> lift all dependencies to parameter level

## Immutability Limitations

* CPU Usage
* Memory Usage

=> **Builder Class** String or Immutable List

## How to Deal with Side Effects

CQRS => Separate Methods with side effects from without side effects  
Separating Domain Logic (Immutable Core) from Mutating State (Mutable Shell)  

## Mutable Implementation

Audit Manager

```C#
  public class AuditManager
    {
        private readonly int _maxEntriesPerFile;
        private readonly string _directoryName;

        public AuditManager(int maxEntriesPerFile, string directoryName)
        {
            _maxEntriesPerFile = maxEntriesPerFile;
            _directoryName = directoryName;
        }

        public void AddRecord(string visitorName, DateTime timeOfVisit)
        {
            string[] filePaths = Directory.GetFiles(_directoryName);
            (int index, string path)[] sorted = SortByIndex(filePaths);

            string newRecord = visitorName + ';' + timeOfVisit.ToString("s");

            if (sorted.Length == 0)
            {
                string newFile = Path.Combine(_directoryName, "audit_1.txt");
                File.WriteAllText(newFile, newRecord);
                return;
            }

            (int currentFileIndex, string currentFilePath) = sorted.Last();
            List<string> lines = File.ReadAllLines(currentFilePath).ToList();

            if (lines.Count < _maxEntriesPerFile)
            {
                lines.Add(newRecord);
                string newContent = string.Join("\r\n", lines);
                File.WriteAllText(currentFilePath, newContent);
            }
            else
            {
                int newIndex = currentFileIndex + 1;
                string newName = $"audit_{newIndex}.txt";
                string newFile = Path.Combine(_directoryName, newName);
                File.WriteAllText(newFile, newRecord);
            }
        }

        private (int index, string path)[] SortByIndex(string[] files)
        {
            return files
                .Select(path => (index: GetIndex(path), path))
                .OrderBy(x => x.index)
                .ToArray();
        }

        private int GetIndex(string filePath)
        {
            // File name example: audit_1.txt
            string fileName = Path.GetFileNameWithoutExtension(filePath);
            return int.Parse(fileName.Split('_')[1]);
        }
    }
```

## Refactoring the First Method

```C#
    public class AuditManager
    {
        private readonly int _maxEntriesPerFile;

        public AuditManager(int maxEntriesPerFile)
        {
            _maxEntriesPerFile = maxEntriesPerFile;
        }

        public FileUpdate AddRecord(
            FileContent[] files,
            string visitorName,
            DateTime timeOfVisit)
        {
            (int index, FileContent file)[] sorted = SortByIndex(files);

            string newRecord = visitorName + ';' + timeOfVisit.ToString("s");

            if (sorted.Length == 0)
            {
                return new FileUpdate("audit_1.txt", newRecord);
            }

            (int currentFileIndex, FileContent currentFile) = sorted.Last();
            List<string> lines = currentFile.Lines.ToList();

            if (lines.Count < _maxEntriesPerFile)
            {
                lines.Add(newRecord);
                string newContent = string.Join("\r\n", lines);
                return new FileUpdate(currentFile.FileName, newContent);
            }
            else
            {
                int newIndex = currentFileIndex + 1;
                string newName = $"audit_{newIndex}.txt";
                return new FileUpdate(newName, newRecord);
            }
        }

        private (int index, FileContent file)[] SortByIndex(
            FileContent[] files)
        {
            return files
                .Select(file => (index: GetIndex(file.FileName), file))
                .OrderBy(x => x.index)
                .ToArray();
        }

        private int GetIndex(string fileName)
        {
            // File name example: audit_1.txt
            string name = Path.GetFileNameWithoutExtension(fileName);
            return int.Parse(name.Split('_')[1]);
        }
    }

      public struct FileUpdate
    {
        public readonly string FileName;
        public readonly string NewContent;

        public FileUpdate(string fileName, string newContent)
        {
            FileName = fileName;
            NewContent = newContent;
        }
    }

    public class FileContent
    {
        public readonly string FileName;
        public readonly string[] Lines;

        public FileContent(string fileName, string[] lines)
        {
            FileName = fileName;
            Lines = lines;
        }
    }

```

## Mutable Shell 

As Dumb and Thin as Possible

```C#
   public class Persister
    {
        public FileContent[] ReadDirectory(string directoryName)
        {
            return Directory
                .GetFiles(directoryName)
                .Select(x => new FileContent(
                    Path.GetFileName(x),
                    File.ReadAllLines(x)))
                .ToArray();
        }

        public void ApplyUpdate(string directoryName, FileUpdate update)
        {
            string filePath = Path.Combine(directoryName, update.FileName);
            File.WriteAllText(filePath, update.NewContent);
        }
    }

    public class ApplicationService
    {
        private readonly string _directoryName;
        private readonly AuditManager _auditManager;
        private readonly Persister _persister;

        public ApplicationService(string directoryName, int maxEntriesPerFile)
        {
            _directoryName = directoryName;
            _auditManager = new AuditManager(maxEntriesPerFile);
            _persister = new Persister();
        }

        public void AddRecord(string visitorName, DateTime timeOfVisit)
        {
            FileContent[] files = _persister.ReadDirectory(_directoryName);
            FileUpdate update = _auditManager.AddRecord(
                files, visitorName, timeOfVisit);
            _persister.ApplyUpdate(_directoryName, update);
        }
    }
```



