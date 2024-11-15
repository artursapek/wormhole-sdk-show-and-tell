---
theme: "./theme.json"
---


```ts
                        ███⠀⠀⠀▒███░⠀⠀░█░▒███████▒⠀⠀██████████░⠀⠀⠀⠀▒███⠀⠀⠀░███░░███⠀⠀⠀⠀███▒⠀░████████░⠀▒███⠀⠀⠀⠀⠀⠀█████████
                        ███⠀⠀▓████▒⠀░█░███▒⠀⠀⠀▒███⠀███▓⠀⠀⠀▓███⠀⠀⠀█████⠀⠀▓████▒▒███░⠀⠀⠀███▒░███░⠀⠀░███░▒███⠀⠀⠀⠀⠀⠀███ ⠀⠀⠀⠀⠀
                        ███⠀▓█░███▒▒█░⠀███▒⠀⠀⠀▒███░██████████▒⠀░█▒▒███░▓▓░███▒░██████████▒▒███⠀⠀⠀⠀███▒▒███⠀⠀⠀⠀⠀⠀████████ 
                        ████▓⠀⠀█████░⠀⠀███▒⠀⠀⠀▒███⠀███▓⠀▓███▒⠀░█▒⠀▒████▓⠀░███▒░███░⠀⠀░███▒░███⠀⠀⠀⠀███░▒███⠀⠀⠀⠀⠀⠀███ ⠀⠀⠀⠀⠀
import { routes } from  ███▒⠀⠀░███▓⠀⠀⠀⠀⠀▒███████▒ ⠀███▓⠀⠀▓███▒█▒⠀⠀▒███▓⠀⠀░███▒░███⠀⠀⠀⠀███▒⠀▒████████▒⠀⠀▒███████⠀█████████

---

# Agenda

1. Quick origin story
2. Route interface design
3. Backpack integration
4. Connect integration
5. What's next

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

- First chance for new SDK to prove itself
- First native bridging integration in a wallet
- Excellent distribution
    - Web & mobile
- Needed high level interface

---

# High level `Route` interface

Looked to Connect for inspiration

Had SDK code in three different places

Unusable for external integrators like backpack


    ┌───────────────────────────┐
    │     @wormhole/connect     │
    │                           │
    │  React UI                 │
    │                           │
    │  Token Bridge Route       │
    │  CCTP Route               │
    │  Portico Route...         │
    │                           │
    │ ┌───────────────────────┐ │
    │ │ @wormhole/connect/SDK │ │
    │ │                       │ │
    │ │ Token Bridge ABIs     │ │
    │ │ Circle ABIs           │ │
    │ │ Primitives            │ │
    │ │                       │ │
    │ │ ┌───────────────────┐ │ │
    │ │ │   @certusone/SDK  │ │ │
    │ │ │                   │ │ │
    │ │ │ Primitives        │ │ │
    │ │ │ VAA utilities     │ │ │
    │ │ └───────────────────┘ │ │
    │ └───────────────────────┘ │
    └───────────────────────────┘
                                                   
                                            
Problems:
  - Spread across three code bases
  - Protocol code is internal
  - Was never intended for external consumption
      - Not published, documented, consumable

Needed a "headless Connect" in a clean package

---

# Headless Connect

- Bring your own UI
- Bring your own signer

## SDKv2 `Route` design goals

- Agnostic to underlying protocol
- Easy interface to implement; few required methods
  - Third party plugins
- Easy to integrate into existing code base
- Consuming UI code can be "dumb"
  - What's a VAA??

---

# Ideal devex - bridging interface

Adding Wormhole bridging to your app should be as easy as possible.

## Main steps

1. Choose which protocols you want to use
2. Get quotes from chosen protocols for intended transfer
3. Choose the quote you want and initiate the transfer
4. Track the transfer progress until it's complete

## Psuedo-code

```js

    // Step 1
    const wh = new Wormhole(...);

    const router = new wh.router([
      routes.AutomaticTokenBridgeRoute,
      routes.CCTPRoute,
      MayanRoute
    ]);

    // Step 2
    const quotes = router.quotes({
      chain: 'Solana',
      token: 'DezXAZ8z7PnrnRJjz3wXBoRgixCa6xjnB7YaB1pPB263',
    }, {
      chain: 'Ethereum',
      token: '0xaaeE1A9723aaDB7afA2810263653A34bA2C21C7a'
    }, '50.00');

    // Step 3
    const { route, quote } = quotes[0];
    const receipt = route.initiate(quote, signer);
    
    // Step 4
    while (!isCompleted(receipt)) {
        receipt = await quote.track(receipt);
        // Handle update to receipt.status
    }

```

---

# Ideal devex - single route

A `Route` can also be used independently, if you're just using a single protocol.

## Psuedo-code

```js

    const wh = new Wormhole(...);

    const tb = new routes.AutomaticTokenBridgeRoute(wh);

    const quote = tb.quote({
      chain: 'Solana',
      token: 'DezXAZ8z7PnrnRJjz3wXBoRgixCa6xjnB7YaB1pPB263',
    }, {
      chain: 'Ethereum',
      token: '0xaaeE1A9723aaDB7afA2810263653A34bA2C21C7a'
    }, '50.00');

    const receipt = route.initiate(quote, signer);

    while (!isCompleted(receipt)) {
        receipt = await route.track(receipt);
        // Handle update to receipt.status
    }
    
```

---

# `Route` Interface
                                                                                                                  
                                                                                                                                  
     Step                                 Parameters                    Route interface method                                       
     ────                                 ──────────                    ──────────────────────                                       
                                         ┌──────────────────────────┐                                                                
     1. Which networks do you support?   │                          │   supportedNetworks(): Network[];                              
                                         └──────────────────────────┘                                                                
                                         ┌──────────────────────────┐                                                                
     2. Which chains do you support?     │ Network:     Mainnet     │   supportedChains(network: Network): Chain[];                  
                                         └──────────────────────────┘                                                                
                                         ┌──────────────────────────┐                                                                
     3. Which tokens can I get?          │ Network:     Mainnet     │   supportedDestinationTokens<N extends Network>(               
                                         │ From chain:  Solana      │     token: TokenId,                                            
                                         │ To chain:    Ethereum    │     fromChain: ChainContext<N>,                                
                                         │ From token:  JUP         │     toChain: ChainContext<N>,                                  
                                         └──────────────────────────┘   ): Promise<TokenId[]>;                                       
                                                                                                                                     
                                         ┌──────────────────────────┐                                                                
     4. Validate my transfer inputs.     │ Network:     Mainnet     │   validate(                                                    
                                         │ From chain:  Solana      │     request: RouteTransferRequest<N>,                          
                                         │ To chain:    Ethereum    │     params: TransferParams<OP>,                                
                                         │ From token:  JUP         │   ): Promise<ValidationResult<OP>>;                            
                                         │ To token:    MOG         │                                                                
                                         │ Amount:      30.245      │                                                                
                                         └──────────────────────────┘                                                                
                                         ┌──────────────────────────┐                                                                
     5. Give me a quote.                 │ Network:     Mainnet     │   quote(                                                       
                                         │ From chain:  Solana      │     request: RouteTransferRequest<N>,                          
                                         │ To chain:    Ethereum    │     params: ValidatedTransferParams<OP>,                       
                                         │ From token:  JUP         │   ): Promise<QuoteResult<OP, VP>>;                             
                                         │ To token:    MOG         │                                                                
                                         │ Amount:      30.245      │                                                                
                                         └──────────────────────────┘                                                                
                                         ┌──────────────────────────┐                                                                
     6. Start the transfer.              │ Network:     Mainnet     │   initiate(                                                    
                                         │ From chain:  Solana      │     request: RouteTransferRequest<N>,                          
                                         │ To chain:    Ethereum    │     sender: Signer,                                            
                                         │ From token:  JUP         │     quote: Quote<OP, VP>,                                      
                                         │ To token:    MOG         │     to: ChainAddress,                                          
                                         │ Amount:      30.245      │   ): Promise<R>;                                               
                                         │ Sender:      (Signer)    │                                                                
                                         │ Quote:       (Quote obj) │                                                                
                                         │ To address:  0xc02a...   │                                                                
                                         └──────────────────────────┘                                                                
                                         ┌──────────────────────────┐                                                                
     7. Tell me when there's an update.  │ Receipt: (Receipt obj)   │   track(receipt: R, timeout?: number): AsyncGenerator<R>;      
                                         └──────────────────────────┘                                                                
        (Called repeatedly until the                                                                                                 
        transfer is complete)                                                                                                        
                                                                                                                                     
---


# Route implementations today

- TokenBridgeRoute
- AutomaticTokenBridgeRoute
- CCTPRoute
- AutomaticCCTPRoute
- PorticoRoute
- NttRoute
- AutomaticNttRoute
- MayanRoute


                                                                                                                                    
---

# Real example from Backpack

Showing just steps 1 and 2


```ts

import {
  routes,
  ...
} from "@wormhole-foundation/sdk-connect";

// Store an instance of RouteResolver
this.resolver = this.wh.resolver([
  nttAutomaticRoute(NttTokenAddresses),
  routes.AutomaticTokenBridgeRoute,
  MayanRoute,
]);

// Later when the user fills in the bridging UI we make a request to the resolver
const request = await routes.RouteTransferRequest.create(
  this.wh,
  {
    source: Wormhole.tokenId(
      toWormholeChain(srcChain),
      toWormholeNative(srcToken)
    ),
    destination: Wormhole.tokenId(
      toWormholeChain(dstChain),
      toWormholeNative(dstToken)
    ),
  },
  this.getWormholeContext(srcChain),
  this.getWormholeContext(dstChain)
);

// Fetch routes which support this request
const foundRoutes = await this.resolver.findRoutes(request);

// Asynchronously fetch a quote for each route, and join them together
const routesAndQuotes: [routes.Route<Network>, WormholeSwapQuote][] = (
  await Promise.all(
    foundRoutes.map(async (route) => {
      try {
        return await this.getQuoteForRoute(route, req);
      } catch (e) {
        console.warn("Error getting quote for route", e);
        // Handle error...
      }
    })
  )
);

```


---


# `Route` plugins

- `Route` plugins can come from any repository / NPM package.
- Adding a new route should be a single line of code

Wormhole Connect config example:

 

     ┌──────────────────────────────────────────────────────────────────┐        
     │                                                                  │        
     │                           Your app                               │
     │                           ────────                               │
     │                                                                  │        
     │  const config: WormholeConnectConfig = {                         │        
     │    routes: Route[] = [                                           │        
     │      ┌──────────────────────────────────────────────────────┐    │        
     │      │                                                      │    │        
     │      │  wormhole-foundation/wormhole-sdk-ts                 │    │        
     │      │                                                      │    │        
     │      │  TokenBridgeRoute                                    │    │        
     │      │  AutomaticTokenBridgeRoute                           │    │        
     │      │  CCTPRoute                                           │    │        
     │      │  AutomaticCCTPRoute                                  │    │        
     │      │  PorticoRoute                                        │    │        
     │      │                                                      │    │        
     │      └──────────────────────────────────────────────────────┘    │        
     │      ┌──────────────────────────────────────────────────────┐    │        
     │      │                                                      │    │        
     │      │  wormhole-foundation/example-native-token-transfers  │    │        
     │      │                                                      │    │        
     │      │  NttRoute                                            │    │        
     │      │  AutomaticNttRoute                                   │    │        
     │      │                                                      │    │        
     │      └──────────────────────────────────────────────────────┘    │        
     │      ┌──────────────────────────────────────────────────────┐    │        
     │      │                                                      │    │        
     │      │  mayan-finance/wormhole-sdk-route                    │    │        
     │      │                                                      │    │        
     │      │  MayanRoute                                          │    │        
     │      │                                                      │    │        
     │      └──────────────────────────────────────────────────────┘    │        
     │    ]                                                             │        
     │  }                                                               │        
     │                                                                  │        
     └──────────────────────────────────────────────────────────────────┘        
                                                                               

---

# What next?

## SDK feature gaps

We're close to saying goodbye to `@certusone` and Advanced Tools

- Platform support (Near, Algorand)
- Attesting/registering tokens
  - Token Bridge and Monad Bridge
  - Needs similar high level interface
- Arbitrary token support

## Interface improvements

- Arbitrary token support
  - `supportedDestinationTokens` doesn't scale
- `RouteResolver` is inadequate
  - Connect has its own implementation
  - Make it easier to fetch quotes
- Transaction history in SDK
  - Connect has its own implementation
- Standardized types for errors/warnings/edge cases

## Pain points

- Version syncing between multiple repositories/packages

## North star

Time-to-integrate


---

# Key Takeaway

All common use cases should be served by `routes`:

```ts
import { routes } from '@wormhole-foundation/sdk';
```

---


                        Questions?

    ████████████████████████████████
    ███████▓▒▒░⠀⠀⠀⠀⠀⠀⠀⠀⠀▒▒▒▓▓███████
    █████▓▒⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀░▒▓████
    ███▓░⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀▒███
    ███░⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀░░▒▒▓▓▒▒░⠀⠀⠀⠀⠀⠀▓██
    ███░⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀░░▒▓▓▓▓▓▒░░⠀⠀⠀⠀▓██
    ██▓░⠀⠀⠀⠀⠀⠀⠀⠀░▒▒▒▒▓▓▓▓▓▒░⠀⠀▒▒▓███
    ██▓⠀⠀⠀░░░▒▒▒░⠀⠀⠀⠀▒▒▒▒▒░⠀⠀⠀▒▒████
    ██░░▒▒▓▓▓▓▒▒░⠀⠀⠀⠀▒▒▒▒▒▒░⠀▒▒▓████
    ██░⠀⠀░▒▒▒▓▓▓▓▒░░▒▓▓▓▓▓▒▒▒▒░▒▓███
    ██▒⠀⠀⠀⠀⠀⠀⠀░░▒▒▓▓▓▓▓▓▓▒▒▒░⠀⠀⠀▓███
    ██▓▒⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀░▒▒▒▒▒▒░░░⠀⠀⠀▒███
    ████▓▓▓▓▓▓░⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀░░▒▓▓███
    ████████▓░⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀░▓████

