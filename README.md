## Overview
http-responses-ts is designed to be a simple way to construct standardized http responses throughout your node applications. The module is written in TypeScript and consists of a parent class from which a number of sub-classes extend:

### `HttpResponse`
```typescript
class HttpResponse {
  statusCode: number
  status: string
  message?: string
}
```

## Usage

### Pre-Defined Responses
http-responses-ts comes with a number of pre-built response classes that collectively implement all the [standard HTTP status codes](https://www.restapitutorial.com/httpstatuscodes.html).

```typescript
import { Created, BadRequest } from 'http-responses-ts'
const error = new BadRequest()
const response = new Created()
```

### Messages
You can optionally specify a message with a response. If you don't define one manually, the message will default to the status.

```typescript
import { Created, BadRequest } from 'http-responses-ts'

const error = new BadRequest('You did something wrong!')
const response = new Created('User Created!')
```

### Custom Responses
It's easy to create your own response types to suit your individual needs. It's always best to extend a subclass of the  `HttpResponse` classe, not the parent class itself. This will ensure your API is following standard HTTP conventions.

```typescript
import { PaymentRequired } from 'http-responses-ts'
// Status Code for this class will be 402...
class CreditCardExpired extends PaymentRequired {
  constructor() {
    super('Your credit card has expired')
  }
}
```

You can also define your own custom status messages. This can be helpful if your upstream clients need to take action based on custom response types that your API returns:

```typescript
import { BadRequest } from 'http-responses-ts'

class ValidationError extends BadRequest {
  constuctor(message: string) {
    super(message, 'Validation Error')
  }
}
```

Your client applications can now distinguish between different types of 400's and react to `Validation Error` response types specifically.

### Using with an http client
There will be times where your service might make requests to external API's. It's easy to use the parent `HttpResponse` class to standardize the errors/responses your client might return. Here's an example using axios:

```typescript
import axios from 'axios'
import { HttpResponse } from 'http-responses-ts'
async function requester(): HttpResponse {
  try {
    const res = await axios.get('https://api.foo.com/resource')
    return new HttpResonse({ statusCode: res.status, status: res.statusText })
  } catch (e) {
    throw new HttpResponse({ statusCode: e.code })
  }
}

requester()
.then(res => {
  // do something...
})
.catch(e => {
  if (e instanceof HttpResponse) {
    // safely resume running
  } else {
    // confidently kill the process as this is an uncaught exception...
  }
})
```

### Unknown Statuses
The library uses the excellent [http-status-codes](https://www.npmjs.com/package/http-status-codes) module under the hood to map status text to status codes. If a status code is used that is not defined in that module, the resulting status will always be "Unknown".
