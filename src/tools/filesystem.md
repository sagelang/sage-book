# Filesystem

The `Fs` tool provides file operations for agents, allowing them to read, write, and manage files.

## Usage

Declare the tool with `use Fs` in your agent:

```sage
agent FileAgent {
    use Fs

    on start {
        try Fs.write("hello.txt", "Hello, World!");
        let content = try Fs.read("hello.txt");
        print(content);
        yield(0);
    }

    on error(e) {
        print("File error");
        yield(-1);
    }
}

run FileAgent;
```

## Methods

### `Fs.read(path: String) -> String`

Reads the entire contents of a file as a string.

```sage
let content = try Fs.read("config.json");
```

### `Fs.write(path: String, content: String) -> Unit`

Writes content to a file. Creates the file if it doesn't exist, or overwrites if it does. Parent directories are created automatically.

```sage
try Fs.write("output/data.txt", "Hello, World!");
```

### `Fs.exists(path: String) -> Bool`

Checks if a file or directory exists.

```sage
if try Fs.exists("config.json") {
    print("Config found");
}
```

### `Fs.list(path: String) -> List<String>`

Lists the contents of a directory, returning file and directory names.

```sage
let files = try Fs.list(".");
for file in files {
    print(file);
}
```

### `Fs.delete(path: String) -> Unit`

Deletes a file.

```sage
try Fs.delete("temp.txt");
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SAGE_FS_ROOT` | Root directory for all file operations | `.` (current directory) |

All paths are relative to the configured root directory:

```bash
# All file operations will be relative to /data
SAGE_FS_ROOT="/data" sage run myprogram.sg
```

## Examples

### Reading and Processing Files

```sage
agent ConfigReader {
    use Fs

    on start {
        if try Fs.exists("config.txt") {
            let config = try Fs.read("config.txt");
            print("Config loaded: " ++ config);
        } else {
            print("No config found, using defaults");
        }
        yield(0);
    }

    on error(e) {
        yield(-1);
    }
}

run ConfigReader;
```

### Writing Log Files

```sage
agent Logger {
    use Fs
    message: String

    on start {
        let timestamp = "2024-01-15T10:30:00";
        let entry = timestamp ++ " - " ++ self.message ++ "\n";

        // Append to log file
        let existing = catch Fs.read("app.log") { "" };
        try Fs.write("app.log", existing ++ entry);

        yield(0);
    }

    on error(e) {
        yield(-1);
    }
}

run Logger { message: "Application started" };
```

### Processing Directory Contents

```sage
agent DirectoryProcessor {
    use Fs

    on start {
        let files = try Fs.list("input");
        let processed = 0;

        for file in files {
            if str_ends_with(file, ".txt") {
                let content = try Fs.read("input/" ++ file);
                let upper = str_upper(content);
                try Fs.write("output/" ++ file, upper);
                processed = processed + 1;
            }
        }

        print("Processed " ++ int_to_str(processed) ++ " files");
        yield(processed);
    }

    on error(e) {
        yield(-1);
    }
}

run DirectoryProcessor;
```

### Creating Nested Directories

```sage
agent NestedWriter {
    use Fs

    on start {
        // Parent directories are created automatically
        try Fs.write("reports/2024/january/summary.txt", "Monthly summary...");
        yield(0);
    }

    on error(e) {
        yield(-1);
    }
}

run NestedWriter;
```

## Notes

- All file operations are async and non-blocking
- Paths are always relative to `SAGE_FS_ROOT` (default: current directory)
- `write()` automatically creates parent directories
- Binary files are not currently supported; use the Shell tool for binary operations
