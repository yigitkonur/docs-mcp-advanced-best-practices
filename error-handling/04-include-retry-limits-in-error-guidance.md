# Include Retry Limits and Fallback Instructions in Errors

Unbounded retry guidance ("try again") can trap the model in infinite loops. Always include a limit and a fallback.

```json
{
  "content": [{
    "type": "text",
    "text": "An unknown error occurred while processing the payment. Retry now. After three consecutive failures, provide the user a link to https://dashboard.example.com/manual-payment to complete the payment manually."
  }],
  "isError": true
}
```

**Implementation pattern:**
```python
@tool
def process_payment(order_id: str, _retry_count: int = 0):
    try:
        return payment_api.charge(order_id)
    except TransientError:
        if _retry_count >= 2:
            return {
                "content": [{
                    "type": "text",
                    "text": (
                        f"Payment processing failed after 3 attempts for order '{order_id}'. "
                        f"Ask the user to complete payment manually at: "
                        f"https://dashboard.example.com/orders/{order_id}/pay"
                    )
                }],
                "isError": True
            }
        return {
            "content": [{
                "type": "text",
                "text": (
                    f"Payment processing temporarily failed (attempt {_retry_count + 1}/3). "
                    f"Call process_payment(order_id='{order_id}', _retry_count={_retry_count + 1}) to retry."
                )
            }],
            "isError": True
        }
```

**Why it matters:** Without limits, some models will retry 10+ times, burning tokens and time. The fallback also prevents the model from giving up entirely - it provides a human escalation path.

**Source:** alpic.ai/blog - "Better MCP tool call error responses"; Stainless - "Error Handling And Debugging MCP Servers"
