# Hello World

Let's write the simplest possible Sage program.

## Create a File

Create a file called `hello.sg`:

```sage
agent Main {
    on start {
        print("Hello from Sage!");
        yield(0);
    }
}

run Main;
```

## Run It

```bash
sage run hello.sg
```

Output:

```
Hello from Sage!
0
```

## What's Happening?

Let's break down this program:

1. **`agent Main { ... }`** — Declares an agent named `Main`. Agents are the basic unit of computation in Sage.

2. **`on start { ... }`** — The `start` handler runs when the agent is spawned. Every agent needs at least one handler.

3. **`print("Hello from Sage!")`** — Prints a message to the console.

4. **`yield(0)`** — Emits a value, signaling that the agent has finished. The emitted value becomes the agent's result.

5. **`run Main`** — Tells the compiler which agent to start. Every Sage program needs exactly one `run` statement.

## Build a Binary

Instead of running directly, you can compile to a standalone binary:

```bash
sage build hello.sg -o out/
./out/hello/hello
```

The binary is self-contained — no Sage installation needed to run it.

## Next Steps

Now let's write something more interesting: [Your First Agent](./first-agent.md).
