# This is demonstration code to show how caches can be stale between cache behaviors

## Overview

The example deploys a CloudFront distribution with 2 origins (```lambda1``` and ```lambda2```) and 2 cache behaviors (```app1/*``` and ```app2/*```).

![image](https://github.com/sashee/cloudfront-cross-origin-cache/assets/82075/578cb4a0-b289-4e97-bf20-cb2a7918bc1e)

![image](https://github.com/sashee/cloudfront-cross-origin-cache/assets/82075/8fcb7fc2-1c70-4764-a1c1-2c8b71c7249c)

Both behaviors use a function that removes the first part of the request path before forwarding it to the origin:

```
function handler(event) {
  var request = event.request;
  request.uri = request.uri.replace(/^\/[^/]*\//, "/");
  return request;
}
```

So a request to ```/app1/index.html``` is forwarded to ```lambda1``` with path ```/index.html```.
Then a request to ```/app2/index.html``` is forwarded to ```lambda2``` with path ```/index.html```.

**The problem is that when one cache behavior caches a response for a given path then it will also be cached for the other one.**

## Steps to reproduce

### Deploy the stack

Using CloudFormation deploy the stack

### Note the outputs

The deployed stack outputs two URLs:

![image](https://github.com/sashee/cloudfront-cross-origin-cache/assets/82075/eeb75918-ac09-4016-8c8c-f3796e27c8dc)

### Open the first URL:

![image](https://github.com/sashee/cloudfront-cross-origin-cache/assets/82075/49040db9-d6b1-435d-aeef-38cb5cb84caf)

The request goes to the correct origin.

### Open the second URL:

![image](https://github.com/sashee/cloudfront-cross-origin-cache/assets/82075/a1562512-35a6-414e-97df-6f7f3f091099)

Here, the response was returned from the cache **for the other cache behavior**.

### Other URLs

When a different URL is opened for the second cache behavior it goes to the correct place:

![image](https://github.com/sashee/cloudfront-cross-origin-cache/assets/82075/e55e62c5-90f3-432d-b5fe-667fac3891f8)

But then it will be cached and opening the same path for the other cache behavior will return this incorrectly cached item:

![image](https://github.com/sashee/cloudfront-cross-origin-cache/assets/82075/48dfaabe-88d6-49a7-8a1d-1379fd0c2ecf)

## Cleanup

Delete the CloudFormation stack.
