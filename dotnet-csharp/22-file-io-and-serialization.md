# File I/O and Serialization

🚦 **Intermediate** | ⏱ 50-60 min

---

## 📝 Why This Matters

File I/O and serialization are fundamental skills for any production application. Whether you're reading configuration files, processing uploaded documents, generating reports, or persisting data, you need to handle files efficiently and safely. Poor file handling leads to locked files, memory leaks, corrupted data, and security vulnerabilities.

In real-world applications, file I/O appears everywhere: importing CSV data, processing images, reading/writing logs, handling user uploads, generating PDFs, exporting reports to Excel, caching data locally, and more. Each scenario requires understanding streams, proper resource disposal, async I/O for scalability, and error handling for resilience.

Serialization—converting objects to formats like JSON or XML—is equally critical. Modern web APIs communicate via JSON. Configuration files use JSON. Message queues exchange JSON or XML messages. Microservices serialize data for persistence or inter-service communication. Understanding serialization performance, security implications, and the differences between serialization libraries (System.Text.Json vs Newtonsoft.Json) directly impacts application performance and security.

From a performance perspective, file I/O is expensive—disk operations are thousands of times slower than memory access. Using async file operations prevents thread blocking, allowing your application to handle more concurrent requests. Buffering reduces disk reads/writes. Streaming large files prevents loading gigabytes into memory. These patterns distinguish scalable applications from ones that fall over under load.

Security matters too. File path traversal attacks (`../../etc/passwd`) can expose sensitive files. Deserializing untrusted JSON can execute malicious code (deserialization attacks). Leaving files open locks them from other processes. Not validating file sizes allows denial-of-service attacks. Production-ready code must defend against these threats.

In interviews, file I/O questions test practical knowledge: "How do you read a large file without loading it entirely into memory?" (use streams), "Why use `using` statements with files?" (resource disposal), "What's the difference between FileStream and StreamReader?" (binary vs text). Serialization questions probe API design: "How do you handle unknown JSON properties?" (ignore or error), "What are the security risks of deserialization?" (code execution).

Modern .NET has also evolved significantly. System.Text.Json (introduced in .NET Core 3.0) is faster and more secure than Newtonsoft.Json, but has different features. Understanding when to use each, how to configure them, and their performance characteristics shows you're current with the ecosystem.

File I/O also intersects with cloud development. Reading from Azure Blob Storage, AWS S3, or Google Cloud Storage uses similar stream patterns. Understanding the abstractions—Stream, TextReader, etc.—makes you adaptable to different storage systems. The same serialization code works whether persisting to files, databases, or cloud storage.

Practically, file handling is where many junior developers make mistakes: forgetting to dispose resources, blocking threads with synchronous I/O, loading huge files into memory, not handling file locks, and poor error handling when files are missing or corrupted. Mastering these patterns elevates your code quality significantly.

---

## 🎯 Core Ideas

### 1. **File and Directory Operations**

**Basic file operations:**

```csharp
using System.IO;

// Check if file exists
if (File.Exists("data.txt"))
{
    Console.WriteLine("File exists");
}

// Read entire file (simple but loads all into memory)
string content = File.ReadAllText("data.txt");
string[] lines = File.ReadAllLines("data.txt");

// Write to file (overwrites existing)
File.WriteAllText("output.txt", "Hello, World!");
File.WriteAllLines("output.txt", new[] { "Line 1", "Line 2", "Line 3" });

// Append to file
File.AppendAllText("log.txt", "New log entry\n");

// Copy, move, delete
File.Copy("source.txt", "destination.txt", overwrite: true);
File.Move("old.txt", "new.txt");
File.Delete("temp.txt");

// Get file info
FileInfo fileInfo = new FileInfo("data.txt");
Console.WriteLine($"Size: {fileInfo.Length} bytes");
Console.WriteLine($"Created: {fileInfo.CreationTime}");
Console.WriteLine($"Modified: {fileInfo.LastWriteTime}");
```

**Directory operations:**

```csharp
// Check if directory exists
if (Directory.Exists("MyFolder"))
{
    Console.WriteLine("Directory exists");
}

// Create directory (including parent directories)
Directory.CreateDirectory("Parent/Child/GrandChild");

// List files
string[] files = Directory.GetFiles("MyFolder");
string[] txtFiles = Directory.GetFiles("MyFolder", "*.txt");
string[] allFiles = Directory.GetFiles("MyFolder", "*.*", SearchOption.AllDirectories);

// List directories
string[] directories = Directory.GetDirectories("MyFolder");

// Enumerate (better for large directories - lazy evaluation)
foreach (string file in Directory.EnumerateFiles("MyFolder", "*.txt"))
{
    Console.WriteLine(file);
}

// Get directory info
DirectoryInfo dirInfo = new DirectoryInfo("MyFolder");
Console.WriteLine($"Created: {dirInfo.CreationTime}");
Console.WriteLine($"Full path: {dirInfo.FullName}");

// Delete directory
Directory.Delete("MyFolder", recursive: true);
```

**Path manipulation (use Path class, NOT string concatenation):**

```csharp
// ❌ BAD: Platform-specific, error-prone
string badPath = "C:\\Users\\Alice" + "\\" + "Documents" + "\\" + "file.txt";

// ✅ GOOD: Cross-platform, safe
string goodPath = Path.Combine("C:", "Users", "Alice", "Documents", "file.txt");
// Windows: C:\Users\Alice\Documents\file.txt
// Linux: C:/Users/Alice/Documents/file.txt

// Path operations
string directory = Path.GetDirectoryName("/path/to/file.txt");  // /path/to
string filename = Path.GetFileName("/path/to/file.txt");        // file.txt
string extension = Path.GetExtension("/path/to/file.txt");      // .txt
string filenameNoExt = Path.GetFileNameWithoutExtension("/path/to/file.txt"); // file

// Change extension
string newPath = Path.ChangeExtension("data.txt", ".json"); // data.json

// Get temp file path
string tempFile = Path.GetTempFileName(); // C:\Users\...\AppData\Local\Temp\tmp1234.tmp

// Get full path
string fullPath = Path.GetFullPath("../data.txt"); // Resolves relative path
```

### 2. **Streams: The Foundation**

**Understanding Stream abstraction:**

```csharp
// Stream is abstract base class for reading/writing bytes
// Concrete implementations:
// - FileStream: files
// - MemoryStream: in-memory data
// - NetworkStream: network communication
// - GZipStream: compression
// - CryptoStream: encryption

// FileStream: binary file access
using (FileStream fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read))
{
    byte[] buffer = new byte[1024];
    int bytesRead = fs.Read(buffer, 0, buffer.Length);
    Console.WriteLine($"Read {bytesRead} bytes");
}

// StreamReader/StreamWriter: text file access
using (StreamReader reader = new StreamReader("data.txt"))
{
    string line;
    while ((line = reader.ReadLine()) != null)
    {
        Console.WriteLine(line);
    }
}

using (StreamWriter writer = new StreamWriter("output.txt"))
{
    writer.WriteLine("Line 1");
    writer.WriteLine("Line 2");
    writer.Flush(); // Force write to disk
}
```

**Reading large files efficiently:**

```csharp
// ❌ BAD: Loads entire file into memory
string content = File.ReadAllText("largefile.txt"); // Could be gigabytes!

// ✅ GOOD: Stream line by line
public static IEnumerable<string> ReadLines(string filePath)
{
    using var reader = new StreamReader(filePath);
    string? line;
    while ((line = reader.ReadLine()) != null)
    {
        yield return line;
    }
}

// Usage: processes one line at a time
foreach (var line in ReadLines("largefile.txt"))
{
    ProcessLine(line); // Memory usage stays constant
}

// ✅ Even better with File.ReadLines (built-in lazy enumeration)
foreach (var line in File.ReadLines("largefile.txt"))
{
    ProcessLine(line);
}
```

**Binary file operations:**

```csharp
// Writing binary data
using (var fs = new FileStream("data.bin", FileMode.Create))
using (var writer = new BinaryWriter(fs))
{
    writer.Write(42);                    // int
    writer.Write(3.14);                  // double
    writer.Write("Hello");               // string (length-prefixed)
    writer.Write(true);                  // bool
    writer.Write(new byte[] { 1, 2, 3 }); // byte array
}

// Reading binary data
using (var fs = new FileStream("data.bin", FileMode.Open))
using (var reader = new BinaryReader(fs))
{
    int intValue = reader.ReadInt32();
    double doubleValue = reader.ReadDouble();
    string stringValue = reader.ReadString();
    bool boolValue = reader.ReadBoolean();
    byte[] bytes = reader.ReadBytes(3);
}
```

### 3. **Async File I/O**

**Why async matters for files:**

```csharp
// ❌ Synchronous: blocks thread during disk I/O
public void ProcessFile(string path)
{
    var content = File.ReadAllText(path); // Thread blocked waiting for disk
    ProcessContent(content);
}

// ✅ Asynchronous: thread free during disk I/O
public async Task ProcessFileAsync(string path)
{
    var content = await File.ReadAllTextAsync(path); // Thread released
    ProcessContent(content);
}

// In ASP.NET Core, this means:
// - Synchronous: 1 thread per request, limited scalability
// - Asynchronous: threads handle many concurrent requests
```

**Async file operations:**

```csharp
// Read file asynchronously
string content = await File.ReadAllTextAsync("data.txt");
string[] lines = await File.ReadAllLinesAsync("data.txt");
byte[] bytes = await File.ReadAllBytesAsync("data.bin");

// Write file asynchronously
await File.WriteAllTextAsync("output.txt", "Hello, World!");
await File.WriteAllLinesAsync("output.txt", new[] { "Line 1", "Line 2" });
await File.WriteAllBytesAsync("data.bin", new byte[] { 1, 2, 3 });

// Stream asynchronously
using (var stream = new FileStream("data.txt", FileMode.Open))
using (var reader = new StreamReader(stream))
{
    string? line;
    while ((line = await reader.ReadLineAsync()) != null)
    {
        await ProcessLineAsync(line);
    }
}

// Copy file asynchronously
using (var source = new FileStream("source.txt", FileMode.Open))
using (var destination = new FileStream("dest.txt", FileMode.Create))
{
    await source.CopyToAsync(destination);
}
```

**Processing uploaded files in ASP.NET Core:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UploadController : ControllerBase
{
    private readonly IWebHostEnvironment _env;
    
    public UploadController(IWebHostEnvironment env)
    {
        _env = env;
    }
    
    [HttpPost]
    public async Task<IActionResult> Upload(IFormFile file)
    {
        // Validate file
        if (file == null || file.Length == 0)
            return BadRequest("No file uploaded");
        
        // Validate size (10 MB limit)
        if (file.Length > 10 * 1024 * 1024)
            return BadRequest("File too large");
        
        // Validate extension
        var allowedExtensions = new[] { ".jpg", ".png", ".pdf" };
        var extension = Path.GetExtension(file.FileName).ToLowerInvariant();
        if (!allowedExtensions.Contains(extension))
            return BadRequest("Invalid file type");
        
        // Generate safe filename
        var safeFileName = $"{Guid.NewGuid()}{extension}";
        var uploadsFolder = Path.Combine(_env.WebRootPath, "uploads");
        Directory.CreateDirectory(uploadsFolder);
        var filePath = Path.Combine(uploadsFolder, safeFileName);
        
        // Save asynchronously
        using (var stream = new FileStream(filePath, FileMode.Create))
        {
            await file.CopyToAsync(stream);
        }
        
        return Ok(new { filename = safeFileName });
    }
}
```

### 4. **JSON Serialization with System.Text.Json**

**Basic serialization:**

```csharp
using System.Text.Json;

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Email { get; set; }
}

// Serialize to JSON string
var person = new Person { Name = "Alice", Age = 30, Email = "alice@example.com" };
string json = JsonSerializer.Serialize(person);
// {"Name":"Alice","Age":30,"Email":"alice@example.com"}

// Deserialize from JSON string
string json = """{"Name":"Alice","Age":30,"Email":"alice@example.com"}""";
Person? person = JsonSerializer.Deserialize<Person>(json);

// Pretty print
string prettyJson = JsonSerializer.Serialize(person, new JsonSerializerOptions
{
    WriteIndented = true
});
// {
//   "Name": "Alice",
//   "Age": 30,
//   "Email": "alice@example.com"
// }
```

**Customizing serialization:**

```csharp
public class Person
{
    // Change property name in JSON
    [JsonPropertyName("full_name")]
    public string Name { get; set; }
    
    // Ignore property
    [JsonIgnore]
    public string Password { get; set; }
    
    // Ignore if null
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
    public string? MiddleName { get; set; }
    
    public int Age { get; set; }
}

// Global options
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,  // camelCase
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
    WriteIndented = true,
    PropertyNameCaseInsensitive = true // Deserialization is case-insensitive
};

string json = JsonSerializer.Serialize(person, options);
Person? person = JsonSerializer.Deserialize<Person>(json, options);
```

**Reading/writing JSON files:**

```csharp
// Serialize to file
var person = new Person { Name = "Alice", Age = 30 };
using (var stream = File.Create("person.json"))
{
    await JsonSerializer.SerializeAsync(stream, person);
}

// Deserialize from file
using (var stream = File.OpenRead("person.json"))
{
    var person = await JsonSerializer.DeserializeAsync<Person>(stream);
}

// Shorthand (less efficient for large files)
string json = JsonSerializer.Serialize(person);
await File.WriteAllTextAsync("person.json", json);

string json = await File.ReadAllTextAsync("person.json");
var person = JsonSerializer.Deserialize<Person>(json);
```

**Handling JSON arrays:**

```csharp
// Serialize array
var people = new[]
{
    new Person { Name = "Alice", Age = 30 },
    new Person { Name = "Bob", Age = 25 }
};

string json = JsonSerializer.Serialize(people);
// [{"Name":"Alice","Age":30},{"Name":"Bob","Age":25}]

// Deserialize array
Person[]? people = JsonSerializer.Deserialize<Person[]>(json);
List<Person>? peopleList = JsonSerializer.Deserialize<List<Person>>(json);
```

**Streaming large JSON:**

```csharp
// Write large JSON array without loading all into memory
using (var stream = File.Create("large.json"))
using (var writer = new Utf8JsonWriter(stream))
{
    writer.WriteStartArray();
    
    for (int i = 0; i < 1_000_000; i++)
    {
        writer.WriteStartObject();
        writer.WriteNumber("id", i);
        writer.WriteString("name", $"User {i}");
        writer.WriteEndObject();
    }
    
    writer.WriteEndArray();
}

// Read large JSON array streaming
using (var stream = File.OpenRead("large.json"))
{
    var jsonReader = JsonDocument.Parse(stream);
    var root = jsonReader.RootElement;
    
    foreach (var item in root.EnumerateArray())
    {
        int id = item.GetProperty("id").GetInt32();
        string name = item.GetProperty("name").GetString()!;
        // Process one item at a time
    }
}
```

### 5. **System.Text.Json vs Newtonsoft.Json**

**When to use each:**

```csharp
// ✅ Use System.Text.Json when:
// - Performance matters (2-3x faster)
// - Security is important (safer by default)
// - Working with modern .NET (Core 3.0+)
// - Simple serialization needs

// ✅ Use Newtonsoft.Json when:
// - Need advanced features (converters, LINQ to JSON)
// - Working with legacy code
// - Need specific compatibility
// - Complex customization requirements

// System.Text.Json example
using System.Text.Json;

var options = new JsonSerializerOptions { PropertyNameCaseInsensitive = true };
var person = JsonSerializer.Deserialize<Person>(json, options);

// Newtonsoft.Json example
using Newtonsoft.Json;

var settings = new JsonSerializerSettings { ContractResolver = new CamelCasePropertyNamesContractResolver() };
var person = JsonConvert.DeserializeObject<Person>(json, settings);
```

**Feature comparison:**

```csharp
// System.Text.Json: Stricter, safer defaults
// - Throws on duplicate keys
// - Throws on trailing commas (unless configured)
// - No support for $type metadata (prevents deserialization attacks)
// - Better performance

// Newtonsoft.Json: More permissive, more features
// - Allows duplicate keys (last wins)
// - Allows trailing commas
// - Supports $type metadata (security risk!)
// - More converters and customization options
```

### 6. **XML Serialization**

**Basic XML serialization:**

```csharp
using System.Xml.Serialization;

[XmlRoot("Person")]
public class Person
{
    [XmlElement("Name")]
    public string Name { get; set; }
    
    [XmlElement("Age")]
    public int Age { get; set; }
    
    [XmlIgnore]
    public string Password { get; set; }
}

// Serialize to XML
var person = new Person { Name = "Alice", Age = 30 };
var serializer = new XmlSerializer(typeof(Person));

using (var writer = new StringWriter())
{
    serializer.Serialize(writer, person);
    string xml = writer.ToString();
    // <?xml version="1.0"?>
    // <Person>
    //   <Name>Alice</Name>
    //   <Age>30</Age>
    // </Person>
}

// Deserialize from XML
using (var reader = new StringReader(xml))
{
    var person = (Person)serializer.Deserialize(reader)!;
}

// To/from file
using (var stream = File.Create("person.xml"))
{
    serializer.Serialize(stream, person);
}

using (var stream = File.OpenRead("person.xml"))
{
    var person = (Person)serializer.Deserialize(stream)!;
}
```

### 7. **Best Practices and Patterns**

**Always dispose file resources:**

```csharp
// ❌ BAD: Resource leak if exception occurs
StreamReader reader = new StreamReader("file.txt");
string content = reader.ReadToEnd();
reader.Dispose(); // Might not execute if ReadToEnd throws!

// ✅ GOOD: using statement ensures disposal
using (StreamReader reader = new StreamReader("file.txt"))
{
    string content = reader.ReadToEnd();
} // Dispose called automatically, even if exception occurs

// ✅ Even better: using declaration (C# 8.0)
using StreamReader reader = new StreamReader("file.txt");
string content = reader.ReadToEnd();
// Dispose called at end of scope
```

**Safe file operations with error handling:**

```csharp
public class FileService
{
    public async Task<string> ReadFileAsync(string path)
    {
        try
        {
            // Validate path
            if (string.IsNullOrWhiteSpace(path))
                throw new ArgumentException("Path cannot be empty", nameof(path));
            
            // Check if file exists
            if (!File.Exists(path))
                throw new FileNotFoundException("File not found", path);
            
            // Read file
            return await File.ReadAllTextAsync(path);
        }
        catch (UnauthorizedAccessException ex)
        {
            throw new InvalidOperationException("Access denied to file", ex);
        }
        catch (IOException ex)
        {
            throw new InvalidOperationException("Failed to read file", ex);
        }
    }
    
    public async Task WriteFileAsync(string path, string content)
    {
        // Ensure directory exists
        var directory = Path.GetDirectoryName(path);
        if (!string.IsNullOrEmpty(directory))
        {
            Directory.CreateDirectory(directory);
        }
        
        // Write atomically: write to temp, then replace
        var tempPath = Path.GetTempFileName();
        try
        {
            await File.WriteAllTextAsync(tempPath, content);
            File.Move(tempPath, path, overwrite: true);
        }
        catch
        {
            // Clean up temp file on error
            if (File.Exists(tempPath))
                File.Delete(tempPath);
            throw;
        }
    }
}
```

**Preventing path traversal attacks:**

```csharp
public class SecureFileService
{
    private readonly string _baseDirectory;
    
    public SecureFileService(string baseDirectory)
    {
        _baseDirectory = Path.GetFullPath(baseDirectory);
    }
    
    public string GetSecurePath(string filename)
    {
        // ❌ NEVER trust user input directly!
        // User could provide: ../../etc/passwd
        
        // ✅ Get full path and validate it's within base directory
        var fullPath = Path.GetFullPath(Path.Combine(_baseDirectory, filename));
        
        if (!fullPath.StartsWith(_baseDirectory, StringComparison.OrdinalIgnoreCase))
        {
            throw new UnauthorizedAccessException("Access denied: path traversal detected");
        }
        
        return fullPath;
    }
    
    public async Task<string> ReadUserFileAsync(string filename)
    {
        var safePath = GetSecurePath(filename);
        return await File.ReadAllTextAsync(safePath);
    }
}

// Usage
var service = new SecureFileService("/app/user-files");
// ✅ Allowed: /app/user-files/document.txt
var content = await service.ReadUserFileAsync("document.txt");

// ❌ Blocked: Would escape to /etc/passwd
// await service.ReadUserFileAsync("../../etc/passwd"); // Throws UnauthorizedAccessException
```

---

## ⚠️ Common Pitfalls

### 1. **Forgetting to Dispose Resources**

```csharp
// ❌ Resource leak
public void ReadFile()
{
    var stream = new FileStream("file.txt", FileMode.Open);
    // If exception occurs here, stream never disposed!
    var reader = new StreamReader(stream);
    var content = reader.ReadToEnd();
}

// ✅ Always use 'using'
public void ReadFile()
{
    using var stream = new FileStream("file.txt", FileMode.Open);
    using var reader = new StreamReader(stream);
    var content = reader.ReadToEnd();
}
```

### 2. **Loading Large Files into Memory**

```csharp
// ❌ Loads entire 1GB file into memory
var content = File.ReadAllText("huge.log"); // OutOfMemoryException!

// ✅ Stream line by line
foreach (var line in File.ReadLines("huge.log"))
{
    ProcessLine(line); // Constant memory usage
}
```

### 3. **Blocking Threads with Synchronous I/O**

```csharp
// ❌ In ASP.NET Core: blocks thread pool thread
[HttpGet]
public IActionResult GetFile()
{
    var content = File.ReadAllText("data.txt"); // Thread blocked!
    return Ok(content);
}

// ✅ Use async file operations
[HttpGet]
public async Task<IActionResult> GetFile()
{
    var content = await File.ReadAllTextAsync("data.txt");
    return Ok(content);
}
```

### 4. **File Path Concatenation with Strings**

```csharp
// ❌ Platform-specific, breaks on Linux
string path = "C:\\Users\\Alice" + "\\" + "Documents" + "\\" + "file.txt";

// ✅ Use Path.Combine
string path = Path.Combine("C:", "Users", "Alice", "Documents", "file.txt");
```

### 5. **Not Validating User File Uploads**

```csharp
// ❌ Security vulnerabilities
[HttpPost]
public async Task<IActionResult> Upload(IFormFile file)
{
    var path = Path.Combine("uploads", file.FileName); // ❌ Path traversal!
    using var stream = File.Create(path);
    await file.CopyToAsync(stream); // ❌ No size limit! ❌ No type validation!
    return Ok();
}

// ✅ Validate everything
[HttpPost]
public async Task<IActionResult> Upload(IFormFile file)
{
    // Validate file exists
    if (file == null || file.Length == 0)
        return BadRequest();
    
    // Validate size (10MB limit)
    if (file.Length > 10 * 1024 * 1024)
        return BadRequest("File too large");
    
    // Validate extension
    var allowedExts = new[] { ".jpg", ".png", ".pdf" };
    var ext = Path.GetExtension(file.FileName).ToLowerInvariant();
    if (!allowedExts.Contains(ext))
        return BadRequest("Invalid file type");
    
    // Generate safe filename (never trust user input!)
    var safeFileName = $"{Guid.NewGuid()}{ext}";
    var path = Path.Combine("uploads", safeFileName);
    
    using var stream = File.Create(path);
    await file.CopyToAsync(stream);
    return Ok(new { filename = safeFileName });
}
```

### 6. **Deserialization Without Validation**

```csharp
// ❌ Deserialize untrusted JSON without validation
var config = JsonSerializer.Deserialize<Config>(untrustedJson);
// What if JSON contains unexpected values? Null properties? Wrong types?

// ✅ Validate after deserialization
var config = JsonSerializer.Deserialize<Config>(untrustedJson);
if (config == null)
    throw new InvalidOperationException("Invalid JSON");

if (string.IsNullOrWhiteSpace(config.ApiKey))
    throw new InvalidOperationException("ApiKey is required");

if (config.Timeout <= 0 || config.Timeout > 300)
    throw new InvalidOperationException("Timeout must be between 1 and 300 seconds");
```

### 7. **Not Handling File Locks**

```csharp
// ❌ Fails if another process has file open
File.WriteAllText("log.txt", "New entry");

// ✅ Handle sharing and retry
public async Task WriteToSharedLogAsync(string message)
{
    int retries = 3;
    while (retries > 0)
    {
        try
        {
            using var stream = new FileStream("log.txt", 
                FileMode.Append, 
                FileAccess.Write, 
                FileShare.Read); // Allow others to read while we write
            using var writer = new StreamWriter(stream);
            await writer.WriteLineAsync(message);
            return;
        }
        catch (IOException) when (retries > 0)
        {
            retries--;
            await Task.Delay(100); // Wait before retry
        }
    }
    throw new IOException("Failed to write to log after retries");
}
```

---

## ✅ Self-Check Questions

1. **When should you use `File.ReadAllText()` vs streaming with `StreamReader`?**
   - `ReadAllText`: Small files that fit comfortably in memory
   - `StreamReader`: Large files, line-by-line processing
   - Streaming prevents loading gigabytes into memory

2. **Why use async file operations in ASP.NET Core?**
   - Frees threads during I/O operations
   - Improves scalability and concurrent request handling
   - Prevents thread pool starvation under load

3. **What's the difference between `FileStream` and `StreamReader`?**
   - `FileStream`: Binary data (bytes)
   - `StreamReader`: Text data (characters), handles encoding
   - `StreamReader` wraps a stream to provide text operations

4. **How do you safely handle user-provided file paths?**
   - Never trust user input directly
   - Use `Path.GetFullPath()` and validate it's within allowed directory
   - Generate safe filenames (GUID + extension)
   - Prevent path traversal attacks

5. **When would you choose System.Text.Json over Newtonsoft.Json?**
   - System.Text.Json: Performance, security, modern .NET
   - Newtonsoft.Json: Advanced features, legacy compatibility
   - System.Text.Json is default for new projects

---

## 🏋️ Practice

1. **Build a CSV parser** that streams large CSV files, converts each row to an object, and validates the data without loading the entire file into memory.

2. **Create a file upload service** that validates file size, type, and content, prevents path traversal, generates safe filenames, and stores files with metadata in a database.

3. **Implement a configuration system** that reads JSON config from multiple sources (file, environment variables, command line), merges them, validates required settings, and provides type-safe access.

4. **Build a log file analyzer** that processes gigabyte-sized log files line-by-line, extracts error patterns, aggregates statistics, and exports results to JSON.

5. **Create a document converter** that reads Word/PDF files, extracts text, converts to JSON/XML, and streams the output without loading entire documents into memory.

---

## 🎤 Interview Cheat Sheet

**Quick answers for common questions:**

**"How do you read a large file without loading it all into memory?"**
> "I use streaming with `StreamReader` or `File.ReadLines()` which provides lazy enumeration. This processes one line at a time, keeping memory usage constant regardless of file size. For example, `foreach (var line in File.ReadLines('huge.log'))` reads line by line without loading the whole file. For binary files, I read in chunks using a buffer with `FileStream.Read()` or `FileStream.CopyToAsync()` for streaming transfers."

**"Why use async file operations?"**
> "In ASP.NET Core, synchronous file I/O blocks thread pool threads during disk operations. Using async file operations like `ReadAllTextAsync()` frees the thread to handle other requests while waiting for I/O to complete. This dramatically improves scalability—you can handle hundreds of concurrent requests with the same thread pool. Disk I/O is thousands of times slower than memory, so blocking threads on I/O is wasteful."

**"What's the difference between System.Text.Json and Newtonsoft.Json?"**
> "System.Text.Json is the modern, built-in serializer in .NET Core 3.0+. It's 2-3x faster and more secure by default—it doesn't support unsafe features like $type metadata that can lead to deserialization attacks. I use it for new projects. Newtonsoft.Json has more features and better backward compatibility, so I use it when migrating legacy code or when I need advanced customization like custom converters or LINQ to JSON."

**"How do you handle file path security?"**
> "I never trust user-provided file paths directly. For uploads, I generate safe filenames using GUIDs and validate extensions against a whitelist. For reading files based on user input, I use `Path.GetFullPath()` to resolve the path, then verify it starts with my allowed base directory to prevent path traversal attacks like `../../etc/passwd`. I also validate file sizes to prevent denial-of-service and use appropriate file shares when opening files."

**"What's the purpose of the `using` statement with files?"**
> "The `using` statement ensures resources are disposed even if exceptions occur. File handles, streams, and readers implement `IDisposable`, and failing to dispose them causes resource leaks—files stay locked, memory isn't released. The `using` statement calls `Dispose()` automatically when the block exits, whether normally or via exception. It's critical for production code reliability."

**"How do you serialize complex objects to JSON?"**
> "I use `JsonSerializer.Serialize()` with options for customization. I can use attributes like `[JsonPropertyName]` to control property names, `[JsonIgnore]` to exclude properties, and `JsonSerializerOptions` to configure naming policies (camelCase/PascalCase), null handling, and formatting. For complex scenarios, I implement custom converters. For large objects, I use streaming with `Utf8JsonWriter` to avoid loading everything into memory."

**Key phrases that impress:**
- "Stream large files to maintain constant memory usage"
- "Use async I/O for scalability in web applications"
- "Always dispose file resources with using statements"
- "Validate user file uploads: size, type, and path"
- "System.Text.Json for performance and security"
- "Path.Combine for cross-platform path handling"

---

**TL;DR for interviews:**

**File operations:**
- `File.ReadAllText`: simple, loads all into memory
- `File.ReadLines`: lazy, streams line-by-line
- `StreamReader`: text files with encoding
- `FileStream`: binary files

**Must-know patterns:**
- Always use `using` for disposal
- Use `Path.Combine` for paths, never string concat
- Async file operations for scalability
- Stream large files, don't load into memory

**Security:**
- Validate file uploads: size, type, extension
- Prevent path traversal: validate full paths
- Generate safe filenames (GUID + extension)
- Never trust user-provided paths

**Serialization:**
- System.Text.Json: fast, secure, modern
- Newtonsoft.Json: features, legacy
- Attributes: `[JsonPropertyName]`, `[JsonIgnore]`
- Options: naming policy, null handling, indentation

**Common mistakes:**
- Forgetting to dispose (use `using`)
- Loading huge files into memory
- Blocking threads with sync I/O
- String concatenation for paths
- Not validating user uploads

Remember: Files and serialization are everywhere. Master async I/O, streaming, and security.
