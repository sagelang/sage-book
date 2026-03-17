# Shell

The `Shell` tool allows agents to execute shell commands and capture their output.

## Usage

Declare the tool with `use Shell` in your agent:

```sage
agent ShellAgent {
    use Shell

    on start {
        let result = try Shell.run("echo 'Hello from shell'");
        print(result.stdout);
        yield(result.exit_code);
    }

    on error(e) {
        print("Command failed");
        yield(-1);
    }
}

run ShellAgent;
```

## Methods

### `Shell.run(command: String) -> ShellResult`

Executes a shell command using `sh -c` and returns the result.

```sage
let result = try Shell.run("ls -la");
print(result.stdout);
```

## ShellResult

Command execution returns a `ShellResult` with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `exit_code` | `Int` | Exit code from the command (0 = success) |
| `stdout` | `String` | Standard output |
| `stderr` | `String` | Standard error |

## Examples

### Basic Command Execution

```sage
agent BasicShell {
    use Shell

    on start {
        let result = try Shell.run("whoami");
        print("Running as: " ++ str_trim(result.stdout));
        yield(0);
    }

    on error(e) {
        yield(-1);
    }
}

run BasicShell;
```

### Checking Exit Codes

```sage
agent ExitCodeChecker {
    use Shell

    on start {
        let result = try Shell.run("test -f /etc/passwd");

        if result.exit_code == 0 {
            print("File exists");
        } else {
            print("File not found");
        }

        yield(result.exit_code);
    }

    on error(e) {
        yield(-1);
    }
}

run ExitCodeChecker;
```

### Handling Errors

```sage
agent ErrorHandler {
    use Shell

    on start {
        let result = try Shell.run("ls /nonexistent");

        if result.exit_code != 0 {
            print("Error: " ++ result.stderr);
        } else {
            print("Output: " ++ result.stdout);
        }

        yield(result.exit_code);
    }

    on error(e) {
        yield(-1);
    }
}

run ErrorHandler;
```

### Complex Commands with Pipes

```sage
agent PipelineAgent {
    use Shell

    on start {
        // Multiple commands with pipes
        let result = try Shell.run("cat /etc/passwd | grep root | head -1");
        print(result.stdout);

        // Command with environment variables
        let result2 = try Shell.run("echo $HOME");
        print("Home: " ++ str_trim(result2.stdout));

        yield(0);
    }

    on error(e) {
        yield(-1);
    }
}

run PipelineAgent;
```

### Running Git Commands

```sage
agent GitAgent {
    use Shell

    on start {
        let status = try Shell.run("git status --short");
        if status.stdout == "" {
            print("Working directory clean");
        } else {
            print("Changes detected:");
            print(status.stdout);
        }

        let branch = try Shell.run("git branch --show-current");
        print("Current branch: " ++ str_trim(branch.stdout));

        yield(0);
    }

    on error(e) {
        yield(-1);
    }
}

run GitAgent;
```

### Building and Testing

```sage
agent BuildAgent {
    use Shell

    on start {
        print("Running tests...");
        let test_result = try Shell.run("cargo test 2>&1");

        if test_result.exit_code == 0 {
            print("Tests passed!");
        } else {
            print("Tests failed:");
            print(test_result.stdout);
        }

        yield(test_result.exit_code);
    }

    on error(e) {
        yield(-1);
    }
}

run BuildAgent;
```

## Security Considerations

- Commands are executed via `sh -c`, so shell features like pipes, redirects, and variable expansion are available
- Be cautious when constructing commands from user input to avoid command injection
- Consider using the `Fs` tool for file operations instead of shell commands when possible

## Notes

- Commands run in the current working directory
- Environment variables from the parent process are inherited
- Long-running commands will block the agent until completion
- Use timeouts in your agent logic if command execution time is a concern
