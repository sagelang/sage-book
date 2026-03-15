# Patterns

Common patterns for building LLM-powered agents.

## Parallel Research

Spawn multiple researchers, combine results:

```sage
agent Researcher {
    topic: String

    on start {
        let result = try infer(
            "Research and provide 3 key facts about: {self.topic}"
        );
        emit(result);
    }

    on error(e) {
        emit("Research failed for topic");
    }
}

agent Synthesizer {
    findings: List<String>

    on start {
        let combined = join(self.findings, "\n\n");
        let synthesis = try infer(
            "Given these research findings:\n{combined}\n\n" ++
            "Provide a unified summary highlighting connections."
        );
        emit(synthesis);
    }

    on error(e) {
        emit("Synthesis failed");
    }
}

agent Coordinator {
    on start {
        // Parallel research
        let r1 = spawn Researcher { topic: "quantum computing" };
        let r2 = spawn Researcher { topic: "machine learning" };
        let r3 = spawn Researcher { topic: "cryptography" };

        let f1 = try await r1;
        let f2 = try await r2;
        let f3 = try await r3;

        // Synthesis
        let s = spawn Synthesizer {
            findings: [f1, f2, f3]
        };
        let result = try await s;

        print(result);
        emit(0);
    }

    on error(e) {
        print("Pipeline failed");
        emit(1);
    }
}

run Coordinator;
```

## Chain of Thought

Break complex reasoning into steps:

```sage
agent ChainOfThought {
    question: String

    on start {
        let understand = try infer(
            "Question: {self.question}\n\n" ++
            "First, restate the question in your own words and identify what's being asked."
        );

        let analyze = try infer(
            "Question: {self.question}\n\n" ++
            "Understanding: {understand}\n\n" ++
            "Now, list the key concepts and relationships involved."
        );

        let solve = try infer(
            "Question: {self.question}\n\n" ++
            "Understanding: {understand}\n\n" ++
            "Analysis: {analyze}\n\n" ++
            "Now, provide a step-by-step solution."
        );

        let answer = try infer(
            "Question: {self.question}\n\n" ++
            "Solution: {solve}\n\n" ++
            "State the final answer concisely."
        );

        emit(answer);
    }

    on error(e) {
        emit("Reasoning failed: " ++ e);
    }
}
```

## Validation Loop

Have agents check each other's work:

```sage
agent Generator {
    task: String

    on start {
        let result = try infer(
            "Complete this task: {self.task}"
        );
        emit(result);
    }

    on error(e) {
        emit("Generation failed");
    }
}

agent Validator {
    task: String
    result: String

    on start {
        let valid = try infer(
            "Task: {self.task}\n\n" ++
            "Result: {self.result}\n\n" ++
            "Is this result correct and complete? " ++
            "Answer YES or NO, then explain briefly."
        );
        emit(valid);
    }

    on error(e) {
        emit("Validation failed");
    }
}

agent Main {
    on start {
        let task = "Write a haiku about programming";

        let gen = spawn Generator { task: task };
        let result = try await gen;

        let val = spawn Validator { task: task, result: result };
        let validation = try await val;

        print("Result: " ++ result);
        print("Validation: " ++ validation);
        emit(0);
    }

    on error(e) {
        emit(1);
    }
}

run Main;
```

## Map-Reduce

Process items in parallel, combine results:

```sage
agent Processor {
    item: String

    on start {
        let result = try infer(
            "Process this item and extract key information: {self.item}"
        );
        emit(result);
    }

    on error(e) {
        emit("Processing failed");
    }
}

agent Reducer {
    items: List<String>

    on start {
        let combined = join(self.items, "\n---\n");
        let result = try infer(
            "Combine these processed items into a summary:\n{combined}"
        );
        emit(result);
    }

    on error(e) {
        emit("Reduction failed");
    }
}

agent MapReduce {
    on start {
        // Map phase - process in parallel
        let p1 = spawn Processor { item: "doc1 content" };
        let p2 = spawn Processor { item: "doc2 content" };
        let p3 = spawn Processor { item: "doc3 content" };

        let r1 = try await p1;
        let r2 = try await p2;
        let r3 = try await p3;

        // Reduce phase
        let reducer = spawn Reducer { items: [r1, r2, r3] };
        let final_result = try await reducer;

        print(final_result);
        emit(0);
    }

    on error(e) {
        emit(1);
    }
}

run MapReduce;
```

## Debate

Multiple agents argue different positions:

```sage
agent Debater {
    position: String
    topic: String

    on start {
        let argument = try infer(
            "You are arguing {self.position} on the topic: {self.topic}\n\n" ++
            "Make your best argument in 2-3 sentences."
        );
        emit(argument);
    }

    on error(e) {
        emit("Argument unavailable");
    }
}

agent Judge {
    topic: String
    arg_for: String
    arg_against: String

    on start {
        let verdict = try infer(
            "Topic: {self.topic}\n\n" ++
            "Argument FOR:\n{self.arg_for}\n\n" ++
            "Argument AGAINST:\n{self.arg_against}\n\n" ++
            "Which argument is stronger and why? Be brief."
        );
        emit(verdict);
    }

    on error(e) {
        emit("Verdict unavailable");
    }
}

agent Main {
    on start {
        let topic = "AI will create more jobs than it destroys";

        let d1 = spawn Debater { position: "FOR", topic: topic };
        let d2 = spawn Debater { position: "AGAINST", topic: topic };

        let arg_for = try await d1;
        let arg_against = try await d2;

        let judge = spawn Judge {
            topic: topic,
            arg_for: arg_for,
            arg_against: arg_against
        };
        let verdict = try await judge;

        print("FOR: " ++ arg_for);
        print("AGAINST: " ++ arg_against);
        print("VERDICT: " ++ verdict);
        emit(0);
    }

    on error(e) {
        emit(1);
    }
}

run Main;
```
