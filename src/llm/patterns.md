# Patterns

Common patterns for building LLM-powered agents.

## Parallel Research

Spawn multiple researchers, combine results:

```sage
agent Researcher {
    topic: String

    on start {
        let result = try divine(
            "Research and provide 3 key facts about: {self.topic}"
        );
        yield(result);
    }

    on error(e) {
        yield("Research failed for topic");
    }
}

agent Synthesizer {
    findings: List<String>

    on start {
        let combined = join(self.findings, "\n\n");
        let synthesis = try divine(
            "Given these research findings:\n{combined}\n\n" ++
            "Provide a unified summary highlighting connections."
        );
        yield(synthesis);
    }

    on error(e) {
        yield("Synthesis failed");
    }
}

agent Coordinator {
    on start {
        // Parallel research
        let r1 = summon Researcher { topic: "quantum computing" };
        let r2 = summon Researcher { topic: "machine learning" };
        let r3 = summon Researcher { topic: "cryptography" };

        let f1 = try await r1;
        let f2 = try await r2;
        let f3 = try await r3;

        // Synthesis
        let s = summon Synthesizer {
            findings: [f1, f2, f3]
        };
        let result = try await s;

        print(result);
        yield(0);
    }

    on error(e) {
        print("Pipeline failed");
        yield(1);
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
        let understand = try divine(
            "Question: {self.question}\n\n" ++
            "First, restate the question in your own words and identify what's being asked."
        );

        let analyze = try divine(
            "Question: {self.question}\n\n" ++
            "Understanding: {understand}\n\n" ++
            "Now, list the key concepts and relationships involved."
        );

        let solve = try divine(
            "Question: {self.question}\n\n" ++
            "Understanding: {understand}\n\n" ++
            "Analysis: {analyze}\n\n" ++
            "Now, provide a step-by-step solution."
        );

        let answer = try divine(
            "Question: {self.question}\n\n" ++
            "Solution: {solve}\n\n" ++
            "State the final answer concisely."
        );

        yield(answer);
    }

    on error(e) {
        yield("Reasoning failed: " ++ e);
    }
}
```

## Validation Loop

Have agents check each other's work:

```sage
agent Generator {
    task: String

    on start {
        let result = try divine(
            "Complete this task: {self.task}"
        );
        yield(result);
    }

    on error(e) {
        yield("Generation failed");
    }
}

agent Validator {
    task: String
    result: String

    on start {
        let valid = try divine(
            "Task: {self.task}\n\n" ++
            "Result: {self.result}\n\n" ++
            "Is this result correct and complete? " ++
            "Answer YES or NO, then explain briefly."
        );
        yield(valid);
    }

    on error(e) {
        yield("Validation failed");
    }
}

agent Main {
    on start {
        let task = "Write a haiku about programming";

        let gen = summon Generator { task: task };
        let result = try await gen;

        let val = summon Validator { task: task, result: result };
        let validation = try await val;

        print("Result: " ++ result);
        print("Validation: " ++ validation);
        yield(0);
    }

    on error(e) {
        yield(1);
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
        let result = try divine(
            "Process this item and extract key information: {self.item}"
        );
        yield(result);
    }

    on error(e) {
        yield("Processing failed");
    }
}

agent Reducer {
    items: List<String>

    on start {
        let combined = join(self.items, "\n---\n");
        let result = try divine(
            "Combine these processed items into a summary:\n{combined}"
        );
        yield(result);
    }

    on error(e) {
        yield("Reduction failed");
    }
}

agent MapReduce {
    on start {
        // Map phase - process in parallel
        let p1 = summon Processor { item: "doc1 content" };
        let p2 = summon Processor { item: "doc2 content" };
        let p3 = summon Processor { item: "doc3 content" };

        let r1 = try await p1;
        let r2 = try await p2;
        let r3 = try await p3;

        // Reduce phase
        let reducer = summon Reducer { items: [r1, r2, r3] };
        let final_result = try await reducer;

        print(final_result);
        yield(0);
    }

    on error(e) {
        yield(1);
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
        let argument = try divine(
            "You are arguing {self.position} on the topic: {self.topic}\n\n" ++
            "Make your best argument in 2-3 sentences."
        );
        yield(argument);
    }

    on error(e) {
        yield("Argument unavailable");
    }
}

agent Judge {
    topic: String
    arg_for: String
    arg_against: String

    on start {
        let verdict = try divine(
            "Topic: {self.topic}\n\n" ++
            "Argument FOR:\n{self.arg_for}\n\n" ++
            "Argument AGAINST:\n{self.arg_against}\n\n" ++
            "Which argument is stronger and why? Be brief."
        );
        yield(verdict);
    }

    on error(e) {
        yield("Verdict unavailable");
    }
}

agent Main {
    on start {
        let topic = "AI will create more jobs than it destroys";

        let d1 = summon Debater { position: "FOR", topic: topic };
        let d2 = summon Debater { position: "AGAINST", topic: topic };

        let arg_for = try await d1;
        let arg_against = try await d2;

        let judge = summon Judge {
            topic: topic,
            arg_for: arg_for,
            arg_against: arg_against
        };
        let verdict = try await judge;

        print("FOR: " ++ arg_for);
        print("AGAINST: " ++ arg_against);
        print("VERDICT: " ++ verdict);
        yield(0);
    }

    on error(e) {
        yield(1);
    }
}

run Main;
```
