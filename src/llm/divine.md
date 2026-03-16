# The divine Expression

The `divine` expression is how Sage programs interact with large language models.

## Basic Usage

Since LLM calls can fail (network errors, API errors), `divine` is a fallible operation that requires `try`:

```sage
agent Main {
    on start {
        let result = try divine("What is the capital of France?");
        print(result);  // "Paris" (or similar)
        yield(0);
    }

    on error(e) {
        print("LLM call failed: " ++ e);
        yield(1);
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
        let summary = try divine(
            "Write a 2-sentence summary of: {self.topic}"
        );
        yield(summary);
    }

    on error(e) {
        yield("Research unavailable");
    }
}
```

Multiple interpolations:

```sage
let format = "JSON";
let topic = "climate change";

let result = try divine(
    "Output a {format} object with key facts about {topic}"
);
```

## The Oracle\<T\> Type

`divine` returns `Oracle<T>`, which wraps the LLM's response.

`Oracle<T>` coerces to `T` automatically:

```sage
let response = try divine("Hello!");
print(response);  // Works - Oracle<String> coerces to String
```

## Structured Output

`divine` can return any type, including user-defined records:

```sage
record Summary {
    title: String,
    key_points: List<String>,
    sentiment: String,
}

agent Analyzer {
    topic: String

    on start {
        let result: Oracle<Summary> = try divine(
            "Analyze this topic and provide a structured summary: {self.topic}"
        );
        print("Title: " ++ result.title);
        print("Sentiment: " ++ result.sentiment);
        yield(result);
    }

    on error(e) {
        print("Analysis failed: " ++ e);
        yield(Summary { title: "Error", key_points: [], sentiment: "unknown" });
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
let result = try divine("prompt");
```

Or use `catch` to handle errors inline with a fallback:

```sage
let result = catch divine("prompt") {
    "fallback value"
};
```

## Example: Multi-Step Reasoning

```sage
agent Reasoner {
    question: String

    on start {
        let step1 = try divine(
            "Break down this question into sub-questions: {self.question}"
        );

        let step2 = try divine(
            "Given these sub-questions: {step1}\n\nAnswer each one briefly."
        );

        let step3 = try divine(
            "Given the original question: {self.question}\n\n" ++
            "And these answers: {step2}\n\n" ++
            "Provide a final comprehensive answer."
        );

        yield(step3);
    }

    on error(e) {
        yield("Reasoning failed: " ++ e);
    }
}

agent Main {
    on start {
        let r = summon Reasoner {
            question: "How do vaccines work and why are they important?"
        };
        let answer = try await r;
        print(answer);
        yield(0);
    }

    on error(e) {
        yield(1);
    }
}

run Main;
```

## Concurrent Inference

Multiple `divine` calls can run concurrently via spawned agents:

```sage
agent Summarizer {
    text: String

    on start {
        let summary = try divine(
            "Summarize in one sentence: {self.text}"
        );
        yield(summary);
    }

    on error(e) {
        yield("Summary unavailable");
    }
}

agent Main {
    on start {
        let s1 = summon Summarizer { text: "Long article about AI..." };
        let s2 = summon Summarizer { text: "Long article about robotics..." };
        let s3 = summon Summarizer { text: "Long article about space..." };

        // All three LLM calls happen concurrently
        let r1 = try await s1;
        let r2 = try await s2;
        let r3 = try await s3;

        print(r1);
        print(r2);
        print(r3);
        yield(0);
    }

    on error(e) {
        yield(1);
    }
}

run Main;
```
