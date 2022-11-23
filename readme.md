# FastAPI Server Side Events

## Usecases

- Push Notifications
- Stock Market

## Snippets

**Server Side**

```python
import asyncio

'''
Get status as an event generator
'''
status_stream_delay = 5  # second
status_stream_retry_timeout = 30000  # milisecond

async def status_event_generator(request, param1):
    previous_status = None
    while True:
        if await request.is_disconnected():
            logger.debug('Request disconnected')
            break

        if previous_status and previous_status['some_end_condition']:
            logger.debug('Request completed. Disconnecting now')
            yield {
                "event": "end",
                "data" : ''
            }
            break

        current_status = await compute_status(param1)
        if previous_status != current_status:
            yield {
                "event": "update",
                "retry": status_stream_retry_timeout,
                "data": current_status
            }
            previous_status = current_status
            logger.debug('Current status :%s', current_status)
        else:
            logger.debug('No change in status...')

        await asyncio.sleep(status_stream_delay)
```

**Client Side**

```js
const evtSource = new EventSource("http://localhost:8080/status/stream?param1=test");
evtSource.addEventListener("update", function(event) {
    // Logic to handle status updates
    console.log(event)
});
evtSource.addEventListener("end", function(event) {
    console.log('Handling end....')
    evtSource.close();
});
```

## Important Points

- There is a limit on maximum number of live connections that can be maintained for a browser — domain combination. So using server sent events efficiently and closing the connection properly plays a key role while working on this flow. Please check the browser compatibility required for the application and whether it supports SSE. Many client-server flows that uses frequent polling can be updated with server sent events, if the problem in hand matches what SSE can provide.


## References

- https://medium.com/towardsdev/transfer-data-from-back-end-to-front-end-without-sockets-server-sent-events-sse-4a0b54c7eb6e

- https://devdojo.com/bobbyiliev/how-to-use-server-sent-events-sse-with-fastapi

- https://sairamkrish.medium.com/handling-server-send-events-with-python-fastapi-e578f3929af1
