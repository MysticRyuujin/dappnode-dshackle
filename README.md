# Emerald Dshackle DAppNode Package

[![DAppNodeStore Available](https://img.shields.io/badge/DAppNodeStore-Available-brightgreen.svg)](http://my.dappnode/#/installer/dshackle.public.dappnode.eth)

[![Emerald Dshackle github](https://img.shields.io/badge/GithubRepo-blue.svg)](https://github.com/emeraldpay/dshackle) (Official)

Emerald Dshackle is a Fault Tolerant Load Balancer for Blockchain API.

The goal of the Emerald Dshackle is to provide a stable routing to multiple nodes, and ensure that each request is executed on an appropriate provider. It considers nodes locations, state, current height, RPC methods it can provide and other characteristics.

It tries to recover from connection errors, faulty nodes, invalid responses, etc. If upstream lags behind others, lost peers below required, started to resync or went down, then Dshackle temporarily excludes it from requests and returns it when the upstream problem is fixed.

## Setup

By default this DAppNode package will try to connect to `http://fullnode.dappnode:8545` as well as endpoints from Infura, Rivet, and Alchemy. These providers all have some form of free tier that you can create an API key for and enter into the setup wizard (or place under the environment settings after startup). It also includes the publicly available Cloudflare endpoint `https://cloudflare-eth.com`

All of the nodes except `fullnode.dappnode` are configured with the `role: fallback` setting to ensure that your DAppNode is preferred over any of the other endpoints.

Because dshackle supports multiple chains, it requires a route to know which chain you are querying. By default only Ethereum chain is configured and it's configured at `/eth`

e.g `http://dshackle.public.dappnode:8545/eth`

## Configuration

All of the configuration takes place in the `/etc/dshackle/dshackle.yaml` file, you can download, modify, and re-upload this file into the package as needed, then simply restart the container for the changes to take effect.

See official github repo documentation for how to configure `dshackle.yaml` - the default config should work for most people, but you might find that you want to expose additional methods, or add new providers, etc.

You can modify the config to include new providers, or private endpoints, etc.

## Notes

There seems to be some kind of bug/issue with Alchemy and Rivet when using WebSocket connections so Alchemy and Rivet are not configured with a WebSocket connection by default. Cloudflare does not expose a WebSocket endpoint to my knowledge, so that's also not configured.

Only with this setup do I get a relatively clean log, without endless streams of errors. You can of course play with these settings yourself. Some of these bugs are already fixed in the dshackle master, so this might in the future.

RPC+WS is preferred however, dshackle does have some issues with very large WebSocket responses (think large debug method responses) so your mileage may vary. Experiment. Play. Have fun!

**WARNING: Using WebSockets results in more calls being used on your free tier as it subscribes to new headers and queries blocks more often, even if it's a fallback endpoint, you'll have to watch your usage and adjust accordingly**

## Chainstack

Chainstack is another provider with a free tier. If you want to use Chainstack you need to send basic auth, here's an example:
```
    url: "https://nd-123-456-789.p2pify.com"
    basic-auth:
        username: connor-macleod
        password: there-can-be-only-one-highlander
```

## License

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
