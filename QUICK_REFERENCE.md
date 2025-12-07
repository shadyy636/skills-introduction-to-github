# Quick Reference Guide - C# File I/O Best Practices

This guide provides quick reference for the key concepts used in your car automotive store project.

## File Opening Modes

```csharp
// FileMode options:
FileMode.Open          // Opens existing file, throws if doesn't exist
FileMode.OpenOrCreate  // Opens existing or creates new file (RECOMMENDED)
FileMode.Create        // Creates new file, overwrites if exists
FileMode.CreateNew     // Creates new file, throws if exists
FileMode.Append        // Opens file and moves to end
```

**For your project:** Use `FileMode.OpenOrCreate` to allow users to create new files or open existing ones.

## Stream Synchronization

```csharp
// When using both StreamReader and StreamWriter:
myfile.Seek(0, SeekOrigin.Begin);    // Move file position
sr.DiscardBufferedData();             // Sync StreamReader with new position
sw.Flush();                           // Ensure all writes are complete
```

**Why?** StreamReader and StreamWriter have internal buffers that can get out of sync with the underlying FileStream.

## Reading Records Safely

```csharp
string line = sr.ReadLine();
if (line != null)
{
    string[] fields = line.Split('|');
    
    // ALWAYS validate array length before accessing
    if (fields.Length >= 4)
    {
        // Now safe to access fields[0], fields[1], fields[2], fields[3]
    }
}
```

## Deleting Records - The Right Way

```csharp
// DON'T: Try to overwrite records in place
myfile.Seek(position, SeekOrigin.Begin);
sw.Write("*");  // ❌ Doesn't work reliably

// DO: Read all records except the one to delete, then rewrite file
List<string> records = new List<string>();
while ((line = sr.ReadLine()) != null)
{
    if (!ShouldDelete(line))
        records.Add(line);
}

// Close and reopen file in Create mode
sr.Close(); sw.Close(); myfile.Close();
myfile = new FileStream(filename, FileMode.Create, FileAccess.ReadWrite);
sw = new StreamWriter(myfile);
sr = new StreamReader(myfile);

// Write back all records
foreach (string record in records)
    sw.WriteLine(record);
sw.Flush();
```

## Resource Management

```csharp
// Class-level variables
FileStream myfile;
StreamReader sr;
StreamWriter sw;

// ALWAYS clean up when closing form
protected override void OnFormClosing(FormClosingEventArgs e)
{
    if (sr != null) sr.Close();
    if (sw != null) sw.Close();
    if (myfile != null) myfile.Close();
    base.OnFormClosing(e);
}
```

**Alternative (better but more complex):** Use `using` statements for automatic disposal:
```csharp
using (FileStream fs = new FileStream(path, FileMode.Open))
using (StreamReader sr = new StreamReader(fs))
{
    // File is automatically closed when leaving this block
    string line = sr.ReadLine();
}
```

## Input Validation

```csharp
// Check for empty inputs
if (string.IsNullOrWhiteSpace(textBox1.Text))
{
    MessageBox.Show("Please enter a value");
    return;
}

// Check for delimiter in data
if (textBox1.Text.Contains('|'))
{
    MessageBox.Show("Pipe character '|' is not allowed");
    return;
}

// Check if file is open
if (myfile == null)
{
    MessageBox.Show("Please open a file first");
    return;
}
```

## Common Patterns

### Search Pattern
```csharp
myfile.Seek(0, SeekOrigin.Begin);
sr.DiscardBufferedData();
sw.Flush();

while ((line = sr.ReadLine()) != null)
{
    string[] fields = line.Split('|');
    if (fields.Length >= 4 && fields[0] == searchTerm)
    {
        // Found it!
        return;
    }
}
// Not found
```

### Add Record Pattern
```csharp
myfile.Seek(0, SeekOrigin.End);
string record = field1 + "|" + field2 + "|" + field3 + "|" + field4;
sw.WriteLine(record);
sw.Flush();
```

### Read Next Record Pattern
```csharp
sw.Flush();  // Ensure all writes are visible
string line = sr.ReadLine();
if (line != null)
{
    string[] fields = line.Split('|');
    if (fields.Length >= 4)
    {
        // Display fields
    }
}
else
{
    MessageBox.Show("No more records");
}
```

### Reset to Beginning Pattern
```csharp
myfile.Seek(0, SeekOrigin.Begin);
sr.DiscardBufferedData();
```

## Common Mistakes to Avoid

❌ **Mistake 1:** Not checking if file is open before operations
```csharp
// BAD
string line = sr.ReadLine();  // Crash if sr is null!

// GOOD
if (sr != null)
    string line = sr.ReadLine();
```

❌ **Mistake 2:** Not validating split array length
```csharp
// BAD
string[] fields = line.Split('|');
textBox1.Text = fields[0];  // Crash if line is empty!

// GOOD
string[] fields = line.Split('|');
if (fields.Length >= 4)
    textBox1.Text = fields[0];
```

❌ **Mistake 3:** Forgetting to flush writes
```csharp
// BAD
sw.WriteLine(record);
string line = sr.ReadLine();  // Might not see the write!

// GOOD
sw.WriteLine(record);
sw.Flush();
string line = sr.ReadLine();
```

❌ **Mistake 4:** Not synchronizing StreamReader after Seek
```csharp
// BAD
myfile.Seek(0, SeekOrigin.Begin);
string line = sr.ReadLine();  // Might read from old position!

// GOOD
myfile.Seek(0, SeekOrigin.Begin);
sr.DiscardBufferedData();
string line = sr.ReadLine();
```

❌ **Mistake 5:** Allowing delimiter character in data
```csharp
// BAD - User enters "BMW|Special"
string record = textBox1.Text + "|" + textBox2.Text;  // Creates extra field!

// GOOD - Validate first
if (textBox1.Text.Contains('|'))
{
    MessageBox.Show("Pipe character not allowed");
    return;
}
```

## Testing Checklist

- [ ] Test adding multiple records
- [ ] Test searching for existing record
- [ ] Test searching for non-existent record
- [ ] Test reading all records with "Read Next"
- [ ] Test deleting a record
- [ ] Test deleting non-existent record
- [ ] Test "Go to Start" button
- [ ] Test clear button
- [ ] Try to search before opening file (should show error)
- [ ] Try to add record with | character (should reject)
- [ ] Close and reopen file to verify saved data
- [ ] Add, then immediately search (test flush)
- [ ] Delete first record
- [ ] Delete last record
- [ ] Delete middle record

## Performance Tips

For large files (1000+ records):
- Consider reading entire file into memory once, then working with List<>
- Use binary format instead of text for better performance
- Consider using a database (SQLite) instead of flat files
- Add indexing for faster searches

## Further Reading

- C# FileStream: https://docs.microsoft.com/en-us/dotnet/api/system.io.filestream
- StreamReader/Writer: https://docs.microsoft.com/en-us/dotnet/api/system.io.streamreader
- File I/O Best Practices: https://docs.microsoft.com/en-us/dotnet/standard/io/

---

**Remember:** Always close your files, validate your inputs, and test edge cases!
