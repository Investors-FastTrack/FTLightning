---
stoplight-id: mv0a7tcg1944x
---
## Rate Limiting

Our API enforces rate limiting to ensure high service availability and fair usage. Below are the details of our rate limiting policy:

### Rate Limit

- **Requests per Second**: 5
- **Burst Capacity**: Up to 5 requests at a time.

#### Understanding the Limits

- **Rate Limit**: You're allowed to make up to 5 requests per second. This limit is evaluated over a rolling one-second window. Exceeding this rate will result in a `429 Too Many Requests` response, indicating a need to slow down your request rate.

- **Burst Capacity**: The API supports burst requests up to 5 requests. This allows for short bursts of up to 5 requests in immediate succession, accommodating quick-succession scenarios without throttling.

#### Best Practices

- **Adapt and Monitor**: Regularly monitor your API request rate and adapt usage to remain within prescribed limits.
  
- **Handle 429 Responses**: Implement logic to manage `429 Too Many Requests` responses, such as exponential backoff, to efficiently control request retry behavior.

- **Cache Strategically**: Cache API responses where feasible to minimize redundant requests, aiding in adherence to rate limits while enhancing application performance.

#### Need Help?

For any issues or queries regarding rate limiting, reach out to our support team for assistance.

