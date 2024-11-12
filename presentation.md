---
theme: "./theme.json"
---

# "SDK v2" and its Route interface

1. Quick origin story
2. Backpack integration
3. Route interface design
4. Connect integration

---

# Origin story

- Andreas -> Ben
- Low level protocol code

## Advantages over `@certusone`

- Much better use of TypeScript
- Metaprogramming/layouting
- More "modern" Nodejs practices
- 28 micro-packages, bundle size friendly

---

# Backpack integration

- First native bridging integration in a wallet
- Excellent distribution
    - Web & mobile
- Needed high level interface

---

# `Route` interface

Looked to Connect for inspiration

Connect had its own super-SDK!
                                                
     ┌───────────────────────────┐              
     │     @wormhole/connect     │              
     │                           │              
     │ ┌───────────────────────┐ │              
     │ │ @wormhole/connect/SDK │ │              
     │ │                       │ │              
     │ │ Token Bridge Route    │ │              
     │ │ CCTP Route            │ │              
     │ │ Portico Route         │ │              
     │ │ ...                   │ │              
     │ │ ┌───────────────────┐ │ │              
     │ │ │   @certusone/SDK  │ │ │              
     │ │ │                   │ │ │              
     │ │ │ Primitives        │ │ │              
     │ │ │ VAA utilities     │ │ │              
     │ │ └───────────────────┘ │ │              
     │ └───────────────────────┘ │              
     └───────────────────────────┘              
                                            
- Problem: confusing to have two disparate SDKs

- Problem: Connect SDK is internal
  - Can't be used by other integrators

- Needed a "headless Connect"

---

# Headless Connect

- Bring your own UI
- Bring your own signer
- Bring your own routes

## `Route` design goals

- Agnostic to underlying protocol
- Small interface, few required methods
- Easy to integrate into existing code base
- Easy to write plugins with it
  - Third party repos
- Consuming UI code can be "dumb"
  - What's a VAA??



```TypeScript
interface Route {

  supportedNetworks(): Network[];

  supportedChains(network: Network): Chain[];

  supportedSourceTokens(fromChain: ChainContext<Network>): Promise<TokenId[]>;

  supportedDestinationTokens<N extends Network>(
    token: TokenId,
    fromChain: ChainContext<N>,
    toChain: ChainContext<N>,
  ): Promise<TokenId[]>;

  quote()

  track()
}
```

talk about protocols and platforms

track method

leave 10-15 mins

places we could use help

near and algorand?

touch on adv tools deprecation
