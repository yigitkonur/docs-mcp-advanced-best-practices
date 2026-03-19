# Declare Output Schemas and Return structuredContent

MCP supports output schemas on tools — declare the shape of your response, and the SDK validates it at runtime. This gives the model a contract it can rely on and catches server bugs before they reach the agent.

**The critical rule:** If you declare an `outputSchema`, you **must** return `structuredContent`. The TypeScript SDK throws if you declare an output schema but only return text `content`. Always return both for backward compatibility.

```typescript
import { z } from "zod";

server.registerTool("get_weather", {
  description: "Get current weather for a city",
  inputSchema: {
    city: z.string().describe("City name"),
  },
  outputSchema: {
    temperature: z.number().describe("Temperature in Celsius"),
    conditions: z.string().describe("Weather conditions"),
    humidity: z.number().describe("Humidity percentage"),
  },
}, async ({ city }) => {
  const weather = await fetchWeather(city);

  return {
    // Text content for clients that don't support structured output
    content: [{
      type: "text",
      text: `${city}: ${weather.temperature}°C, ${weather.conditions}, ${weather.humidity}% humidity`,
    }],
    // Structured content validated against outputSchema
    structuredContent: {
      temperature: weather.temperature,
      conditions: weather.conditions,
      humidity: weather.humidity,
    },
  };
});
```

**Why this matters:**

1. **Agent reliability** — the model knows the exact response shape before calling the tool, so it can plan multi-step workflows without exploratory calls
2. **Runtime validation** — the SDK rejects malformed responses at the server, not at the agent
3. **Backward compatibility** — older clients that don't understand `structuredContent` still get the text `content`

**Common mistake:** Declaring an output schema during development, then forgetting to populate `structuredContent`. The SDK will throw a validation error at runtime. If you're not ready to commit to a schema, don't declare one — add it when the response shape stabilizes.

**Source:** [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) (output schema validation logic); [MCP specification — tools](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)
