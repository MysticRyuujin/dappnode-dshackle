# Emerald Dshackle DAppNode Package

[![DAppNodeStore Available](https://img.shields.io/badge/DAppNodeStore-Available-brightgreen.svg)](http://my.dappnode/#/installer/dshackle.public.dappnode.eth)

[![Emerald Dshackle github](https://img.shields.io/badge/GithubRepo-blue.svg)](https://github.com/emeraldpay/dshackle) (Official Dshackle Repo)

Emerald Dshackle is a Fault Tolerant Load Balancer for Blockchain API.

The goal of the Emerald Dshackle is to provide a stable routing to multiple nodes, and ensure that each request is executed on an appropriate provider. It considers nodes locations, state, current height, RPC methods it can provide and other characteristics.

It tries to recover from connection errors, faulty nodes, invalid responses, etc. If upstream lags behind others, lost peers below required, started to resync or went down, then Dshackle temporarily excludes it from requests and returns it when the upstream problem is fixed.

## Setup

By default this DAppNode package will try to connect to `http://geth.dappnode:8545` (you can change this in the setup wizard) as well as the following publicly available and free endpoints:

[EthereumNodes](https://ethereumnodes.com/) and [ChainList.org](https://chainlist.org)

| Provider | URL | NETWORK | NOTES |
| -------- | --- | ------- | ----- |
| 1rpc | `https://1rpc.io/eth` | Ethereum |
| 1rpc | `https://1rpc.io/matic` | Polygon |
| Ankr  |`https://rpc.ankr.com/eth` | Ethereum |
| Ankr  |`https://rpc.ankr.com/polygon` | Polygon |
| Blast | `https://eth-mainnet.public.blastapi.io` | Ethereum |
| Blast | `https://polygon-mainnet.public.blastapi.io` | Polygon |
| Cloudflare | `https://cloudflare-eth.com` | Ethereum |
| dRPC | `https://eth.drpc.org` | Ethereum |
| dRPC | `https://eth.drpc.org` | Polygon |
| LlamaNodes | `https://eth.llamarpc.com` | Ethereum |
| LlamaNodes | `https://polygon.llamarpc.com` | Polygon |
| MyEtherWallet | `https://nodes.mewapi.io/rpc/eth` | Ethereum |
| Pokt | `https://eth-rpc.gateway.pokt.network` | Ethereum |
| Pokt | `https://poly-rpc.gateway.pokt.network` | Polygon |
| Polygon-RPC | `https://polygon-rpc.com` | Polygon |
| PublicNode | `https://ethereum.publicnode.com` | Ethereum |
| PublicNode | `https://polygon-bor.publicnode.com` | Polygon |
| QuickNode | `https://rpc-mainnet.matic.quiknode.pro` | Polygon |

All of the nodes except the local node (e.g. `geth.dappnode`) are configured with the `role: fallback` setting to ensure that your DAppNode is preferred over any of the other endpoints. Since there is no default endpoint for Polygon, and they are all fallback, it make no difference on Polygon. However, you can certainly add your own `role: primary` endpoint if you are running a local node! Additionally, if you sign up for any Premium or Paid endpoints, you can configure those with `role: secondary` so they're preferred over the free public fallback nodes!

Because Dshackle supports multiple chains, it requires a route to know which chain you are querying. By default the Ethereum chain is configured at `/eth` and Matic/Polygon is configured at `/polygon`

e.g `http://dshackle.public.dappnode:8545/eth` or `ws://dshackle.public.dappnode:8546/polygon`

## Privacy Considerations

Take a look at [ChainList.org](https://chainlist.org) - You may want to add or remove nodes from the default list if you're worried about the privacy policy of various providers, if you're into that kind of thing.

## Configuration

All of the configuration takes place in the `/etc/dshackle/dshackle.yaml` file, you can download, modify, and re-upload this file into the package as needed, then simply restart the container for the changes to take effect.

See official github repo documentation for how to configure `dshackle.yaml` - the default config should work for most people, but you might find that you want to expose additional methods, or add new providers, etc.

You can modify the config to include new providers, or private endpoints, etc.

### MERGE UPDATE

There is now a `priority` field in the `/etcdshackle/dshackle.yaml` file under each provider. Obviously this is a subjective number that would only ever come into play if there was a fork. You may set whatever values you want here, but according to the Dshackle docs every endpoint must have a unique priority.

The higher number is more trusted. My choices in the default configs do not reflect any logic or implied bias on my part, I just started at 100 for the local node and subtracted 10 per endpoint as I went down. Set whatever you want.

This setting does not apply to Polygon endpoints.

## WebSockets

Some notes about WebSocket usage and Dshackle:

Below are some of the issues I've seen with WebSockets:

1. Some providers behave strangely when configured with WebSockets. Dshackle will throw odd errors, etc.
2. Dshackle has issues with very large WebSocket responses (e.g. `debug_` methods) where it will just drop the reply.
3. Dshackle will use WebSockets to subscribe to new headers/blocks. This consumes a decent number of "calls" to your "freemium" nodes.
4. I recently noticed that the official DAppNode Geth client has WebSockets disabled (I don't know why).

WebSockets are a pain in the ass, sorry but they just are. However, since Dshackle has put a lot of effort into supporting them and making them better, I've included support for it in the new release (`v0.3.0`)

If you want to enable WebSockets on the default Geth installation you have to add this to the extra options in the Geth package:

```bash
--ws --ws.addr 0.0.0.0 --ws.port 8546 --ws.origins="*" --ws.api eth,net,web3,txpool
```

You can adjust the `--ws.api` as needed obviously. Also, by default methods like `eth_newFilter` are not enabled by Dshackle, so you'd need to enable them on a supported upstream provider (such as your local node).

Look, it's all configurable... so you do you, but I'm warning you... WebSockets suck.

Adding/Removing WebSocket connections to endpoints is as simple as:

```yaml
    rpc:
        url: "http://geth.dappnode:8545"
    ws:
        url: "ws://geth.dappnode:8546"
```

Adding/Removing methods is in the [docs](https://github.com/emeraldpay/dshackle/blob/master/docs/04-upstream-config.adoc) but it looks like this:

```yaml
    methods:
      enabled:
        - name: eth_newFilter
      disabled:
        - name: eth_sendRawTransaction
```

## TLS/SSL

Dshackle does support TLS/SSL encryption, you can upload certs into the `/etc/dshackle` folder if you want to use it, though it might be easier to just expose Dsahckle through the DAppNode's HTTPS tool.

For more info on how to use/setup TLS for Dshackle see the official repo docs.

## Metrics

Prometheus metrics are enabled in the default config at `:8081/metrics`

## Health Check

A health check is provided at `:8082/health` and will return HTTP 200 if at least 1 Ethereum endpoint and 1 Polygon endpoint is working.

See [Health Checks](https://github.com/emeraldpay/dshackle/blob/master/docs/06-monitoring.adoc#health-checks) for more info.

## Flashbots

WARNING: **FLASHBOTS MAY OR MAY NOT CENSOR TRANSACTIONS, SEE: TWITTER DRAMA OVER TORNADO CASH**

A cool trick you can do to submit transactions through [Flashbots](https://docs.flashbots.net/flashbots-protect/rpc/quick-start/)

1. Add Flashbots RPC

    ```yaml
        rpc:
            url: "https://rpc.flashbots.net"
    ```

2. Disable `eth_sendRawTransaction` on all configured endpoints EXCEPT for the Flashbots RPC endpoint:

    ```yaml
        methods:
        disabled:
            - name: eth_sendRawTransaction
    ```

Now Dshackle has no choice but to route all of your transactions through the only available upstream for that method, the Flashbots RPC endpoint.

## Other Providers

Below is a list of other providers I know of that you can also add to your config by editing your `/etc/dshackle/dshackle.yaml` file.

Also check out [ChainList.org](https://chainlist.org) or [EthereumNodes](https://ethereumnodes.com/) to find more Freemium or Paid providers!

### QuickNode

QuickNode now has a free tier! Using QuickNode is as simple as signing up and creating an endpoint and adding it like this:

```yaml
    rpc:
        url: "https://${name-name-name}.quiknode.pro/${token}/
```

### Chainstack

Chainstack is another provider with a free tier. If you want to use Chainstack you need to send basic auth, here's an example:

```yaml
    rpc:
        url: "https://${whatever}.p2pify.com"
        basic-auth:
            username: connor-macleod
            password: there-can-be-only-one
```

### Pocket Network (POKT)

Pocket Network (a.k.a POKT) is another provider with a free tier that gives you the option to use a "Secret Key" which according to their docs is just an HTTP basic-auth password, without a username. You don't have to enable it, but it's recommended. I haven't actually tested this but theoretically this would work:

```yaml
    rpc:
        url: "https://eth-mainnet.gateway.pokt.network/v1/lb/${whatever}"
        basic-auth:
            username: ""
            password: ${SecretKey}
```

## More Configs

I have another repo that contains various configs for various clients and providers:

[CONFIGS](https://github.com/MysticRyuujin/dshackle-configs)

Be warned, I don't update it very often, it could be outdated. Use with caution.

## License

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

[LICENSE](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
