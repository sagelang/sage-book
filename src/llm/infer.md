# The infer Expression

The `infer` expression is how Sage programs interact with large language models.

## Basic Usage

Since LLM calls can fail (network errors, API errors), `infer` is a fallible operation that requires `try`:

```sage
agent Main {
    on start {
        let result = try infer("What is the capital of France?");
        print(result);  // "Paris" (or similar)
        emit(0);
    }

    on error(e) {
        print("LLM call failed: " ++ e);
        emit(1);
    }
}

run Main;
```

## String Interpolation

Use `{identifier}` to include variables in prompts:

```sage
agent Researcher {
    topic: String

    on start {
        let summary = try infer(
            "Write a 2-sentence summary of: {self.topic}"
        );
        emit(summary);
    }

    on error(e) {
        emit("Research unavailable");
    }
}
```

Multiple interpolations:

```sage
let format = "JSON";
let topic = "climate change";

let result = try infer(
    "Output a {format} object with key facts about {topic}"
);
```

## The Inferred\<T\> Type

`infer` returns `Inferred<T>`, which wraps the LLM's response.

`Inferred<T>` coerces to `T` automatically:

```sage
let response = try infer("Hello!");
print(response);  // Works - Inferred<String> coerces to String
```

## Structured Output

`infer` can return any type, including user-defined records:

```sage
record Summary {
    title: String,
    key_points: List<String>,
    sentiment: String,
}

agent Analyzer {
    topic: String

    on start {
        let result: Inferred<Summary> = try infer(
            "Analyze this topic and provide a structured summary: {self.topic}"
        );
        print("Title: " ++ result.title);
        print("Sentiment: " ++ result.sentiment);
        emit(result);
    }

    on error(e) {
        print("Analysis failed: " ++ e);
        emit(Summary { title: "Error", key_points: [], sentiment: "unknown" });
    }
}
```

The runtime automatically:
1. Injects the expected schema into the prompt
2. Parses the LLM's response as JSON
3. Retries with error feedback if parsing fails (configurable via `SAGE_INFER_RETRIES`)

This works with any OpenAI-compatible API, including Ollama.

## Error Handling

Use `try` to propagate errors to the agent's `on error` handler:

```sage
let result = try infer("prompt");
```

Or use `catch` to handle errors inline with a fallback:

```sage
let result = catch infer("prompt") {
    "fallback value"
};
```

## Example: Multi-Step Reasoning

```sage
agent Reasoner {
    question: String

    on start {
        let step1 = try infer(
            "Break down this question into sub-questions: {self.question}"
        );

        let step2 = try infer(
            "Given these sub-questions: {step1}\n\nAnswer each one briefly."
        );

        let step3 = try infer(
            "Given the original question: {self.question}\n\n" ++
            "And these answers: {step2}\n\n" ++
            "Provide a final comprehensive answer."
        );

        emit(step3);
    }

    on error(e) {
        emit("Reasoning failed: " ++ e);
    }
}

agent Main {
    on start {
        let r = spawn Reasoner {
            question: "How do vaccines work and why are they important?"
        };
        let answer = try await r;
        print(answer);
        emit(0);
    }

    on error(e) {
        emit(1);
    }
}

run Main;
```

## Concurrent Inference

Multiple `infer` calls can run concurrently via spawned agents:

```sage
agent Summarizer {
    text: String

    on start {
        let summary = try infer(
            "Summarize in one sentence: {self.text}"
        );
        emit(summary);
    }

    on error(e) {
        emit("Summary unavailable");
    }
}

agent Main {
    on start {
        let s1 = spawn Summarizer { text: "Long article about AI..." };
        let s2 = spawn Summarizer { text: "Long article about robotics..." };
        let s3 = spawn Summarizer { text: "Long article about space..." };

        // All three LLM calls happen concurrently
        let r1 = try await s1;
        let r2 = try await s2;
        let r3 = try await s3;

        print(r1);
        print(r2);
        print(r3);
        emit(0);
    }

    on error(e) {
        emit(1);
    }
}

run Main;
```
