# custom-headers-prism
Example of using a proxy in front of Prism for programatic example selection

## Use cases

One use case is when you are testing a web application which connects to an API. Using a proxy
as middleware between the web application and the API allows programatic control to specify a 
particular example or HTTP response from a [Prism](https://stoplight.io/open-source/prism) mock
server of the API, _without_ needing to modify the web application's API endpoint for each
testing scenario. Instead all you need to do is specify the proxy server as the API endpoint.

## Prerequisites to run the example

- [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)

## Codespace

An easy way to run the example is to
[create a free GitHub Codespace](https://docs.github.com/en/codespaces/developing-in-codespaces/creating-a-codespace-for-a-repository#creating-a-codespace-for-a-repository), as that will give
you an instant development environment in the cloud.  All you need is a GitHub account and a web
browser (or [VS Code](https://code.visualstudio.com/)).

## Running the example

1. [Install Prism](https://docs.stoplight.io/docs/prism/f51bcc80a02db-installation):

    `npm install -g @stoplight/prism-cli`

2. Install [node-http-proxy](https://github.com/http-party/node-http-proxy):

    `npm install http-proxy --save`

3. Start a Prism mock server based on our example Petstore OpenAPI spec:

   `prism mock petstore.oas3.yaml`

   This OpenAPI spec is an exact copy of [the petstore example on Prism's own GitHub 
   repo](https://github.com/stoplightio/prism/blob/master/examples/petstore.oas3.yaml).
   The only modification was to remove the security section, as that allows us to make
   a plain HTTP request to Prism (e.g. http://127.0.0.1:4010/pets/123) without needing
   to provide authorization headers.

4. Start a Proxy server:

   `node proxy.js`

   The proxy server is a stand-alone
   [`node-http-proxy` server](https://github.com/http-party/node-http-proxy), an exact copy of 
   their [stand-alone proxy server with proxy request header
   re-writing example](https://github.com/http-party/node-http-proxy#setup-a-stand-alone-proxy-server-with-proxy-request-header-re-writing).
   The only modifications were to
   [point the proxy server to Prism](https://github.com/agilepathway/custom-headers-prism/pull/5/files),
   and to 
   [rewrite the `Prefer` header](https://github.com/agilepathway/custom-headers-prism/pull/4/files)
   so that Prism serves up the example we specify there.

5. Request a pet directly from Prism, and you will see you get [the first example in the OpenAPI
   spec back (Fluffy the cat)](petstore.oas3.yaml#L349-L361)

   `curl http://127.0.0.1:4010/pets/123`

6. Request a pet from the proxy, and you will see you get [the example specified in the proxy 
  server](proxy.js#L18) back [(Sharik the dog)](petstore.oas3.yaml#L362-L374)

   `curl http://127.0.0.1:5050/pets/123`

That's all there is to it.  Going back to the [example use case above](#use-cases), it would be
trivial to startup and teardown a proxy server for individual test cases, programatically
specifying which Prism example or response code should be returned each time.

