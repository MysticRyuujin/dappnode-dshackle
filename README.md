# Emerald Dshackle DAppNode Package

[![DAppNodeStore Available](https://img.shields.io/badge/DAppNodeStore-Available-brightgreen.svg)](http://my.dappnode/#/installer/dshackle.public.dappnode.eth)

[![Emerald Dshackle github](https://img.shields.io/badge/GithubRepo-blue.svg)](https://github.com/emeraldpay/dshackle) (Official Dshackle Repo)

Emerald Dshackle is a Fault Tolerant Load Balancer for Blockchain API.

The goal of the Emerald Dshackle is to provide a stable routing to multiple nodes, and ensure that each request is executed on an appropriate provider. It considers nodes locations, state, current height, RPC methods it can provide and other characteristics.

It tries to recover from connection errors, faulty nodes, invalid responses, etc. If upstream lags behind others, lost peers below required, started to resync or went down, then Dshackle temporarily excludes it from requests and returns it when the upstream problem is fixed.

## Setup

By default this DAppNode package will try to connect to `http://fullnode.dappnode:8545` as well as endpoints from Infura, Rivet, and Alchemy. These providers all have some form of free tier that you can create an API key for and enter into the setup wizard (or place under the environment settings after startup). It also includes the publicly available Cloudflare endpoint `https://cloudflare-eth.com` AND the free public endpoint by Ankr `https://rpc.ankr.com/eth`

All of the nodes except `fullnode.dappnode` are configured with the `role: fallback` setting to ensure that your DAppNode is preferred over any of the other endpoints.

Because dshackle supports multiple chains, it requires a route to know which chain you are querying. By default only Ethereum chain is configured and it's configured at `/eth`

e.g `http://dshackle.public.dappnode:8545/eth`

## Configuration

All of the configuration takes place in the `/etc/dshackle/dshackle.yaml` file, you can download, modify, and re-upload this file into the package as needed, then simply restart the container for the changes to take effect.

See official github repo documentation for how to configure `dshackle.yaml` - the default config should work for most people, but you might find that you want to expose additional methods, or add new providers, etc.

You can modify the config to include new providers, or private endpoints, etc.

## WebSockets

Some notes about WebSocket usage and Dshacke:

1. Some providers behave strangely when configured with WebSockets. Dshackle will throw odd errors, etc.
2. Dshackle has issues with very large WebSocket responses (e.g. `debug_` methods) where it will just drop the reply.
3. Dshackle will use WebSockets to subscribe to new headers/blocks. This consumes a decent number of "calls" to your "freemium" nodes.

For the reasons above I only included a WebSocket config for the DAppNode's `fullnode` endpoint.

If you don't care about the extra calls, and/or you're not doing any large `debug_` calls to your nodes, you can add WebSocket URLs for the endpoints that support it (e.g. Infura). However, your mileage may vary, I get weird errors with Alchemy for instance.

## TLS/SSL

Dshackle does support TLS/SSL encryption, you can upload certs into the `/etc/dshackle` folder if you want to use it, though it might be easier to just expose Dsahckle through the DAppNode's HTTPS tool.

For more info on how to use/setup TLS for Dshackle see the official repo docs.

## Flashbots!

A cool trick you can do to submit transactions through [Flashbots](https://docs.flashbots.net/flashbots-protect/rpc/quick-start/)

1. Add Flashbots RPC
```
    rpc:
        url: "https://rpc.flashbots.net"
```
2. Disable `eth_sendRawTransaction` on all configured endpoints EXCEPT for the Flashbots RPC endpoint:

```
    methods:
      disabled:
        - name: eth_sendRawTransaction
```

Now Dshackle has no choice but to route all of your transactions through the only available upstream for that method, the Flashbots RPC endpoint.

## Other Providers
Below is a list of "other" providers I know of that you can also add to your config. I didn't include them because I don't have a ton of experience using them and I figured setting up three was already enough.

### Chainstack

Chainstack is another provider with a free tier. If you want to use Chainstack you need to send basic auth, here's an example:
```
    rpc:
        url: "https://${whatever}.p2pify.com"
        basic-auth:
            username: connor-macleod
            password: there-can-be-only-one
```

### Pocket Network

Pocket network is another provider with a free tier that gives you the option to use a "Secret Key" which according to their docs is just an HTTP basic-auth password, without a username. You don't have to enable it, but it's recommended. I haven't actually tested this but theoretically this would work:
```
    rpc:
        url: "https://eth-mainnet.gateway.pokt.network/v1/lb/${whatever}"
        basic-auth:
        username: ""
        password: ${SecretKey}
```

### ZMOK.io

Nothing too fancy here, you sign in with a web3 wallet (e.g. MetaMask) and they give you a URL to use.
```
    rpc:
        url: "https://api.zmok.io/mainnet/${whatever}"
```

## More Configs

I have another repo that contains various configs for various clients and providers:

https://github.com/MysticRyuujin/dshackle-configs

## License

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
