# HTTP Client

The `Http` tool provides methods for making HTTP requests.

## Usage

Declare the tool with `use Http` in your agent:

```sage
agent ApiClient {
    use Http

    on start {
        let response = try Http.get("https://api.example.com/data");
        print("Status: " ++ str(response.status));
        print("Body: " ++ response.body);
        emit(response.status);
    }

    on error(e) {
        print("Request failed");
        emit(-1);
    }
}

run ApiClient;
```

## Methods

### `Http.get(url: String) -> HttpResponse`

Performs an HTTP GET request.

```sage
let response = try Http.get("https://httpbin.org/get");
```

### `Http.post(url: String, body: String) -> HttpResponse`

Performs an HTTP POST request with a JSON body.

```sage
let response = try Http.post(
    "https://httpbin.org/post",
    "{\"key\": \"value\"}"
);
```

## HttpResponse

Both methods return an `HttpResponse` with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `status` | `Int` | HTTP status code (e.g., 200, 404, 500) |
| `body` | `String` | Response body as text |
| `headers` | `Map<String, String>` | Response headers |

## Examples

### Fetching JSON Data

```sage
agent JsonFetcher {
    use Http
    url: String

    on start {
        let response = try Http.get(self.url);
        if response.status == 200 {
            emit(response.body);
        } else {
            emit("Error: " ++ str(response.status));
        }
    }

    on error(e) {
        emit("Request failed");
    }
}

run JsonFetcher { url: "https://httpbin.org/json" };
```

### Posting Data

```sage
agent DataPoster {
    use Http

    on start {
        let payload = "{\"message\": \"Hello from Sage!\"}";
        let response = try Http.post("https://httpbin.org/post", payload);
        emit(response.status);
    }

    on error(e) {
        emit(-1);
    }
}

run DataPoster;
```

### Error Recovery

```sage
agent ResilientFetcher {
    use Http
    urls: List<String>

    on start {
        for url in self.urls {
            let response = catch Http.get(url) {
                HttpResponse { status: 0, body: "", headers: {} }
            };
            if response.status == 200 {
                emit(response.body);
                return;
            }
        }
        emit("All URLs failed");
    }
}

run ResilientFetcher {
    urls: ["https://primary.example.com", "https://backup.example.com"]
};
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SAGE_HTTP_TIMEOUT` | Request timeout in seconds | `30` |

The HTTP client automatically sets a `User-Agent` header of `sage-agent/{version}`.
