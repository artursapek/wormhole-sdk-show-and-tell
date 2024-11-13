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

# High level `Route` interface

Looked to Connect for inspiration

Connect had its own super-SDK!


     ┌───────────────────────────┐ 
     │     @wormhole/connect     │ 
     │                           │ 
     │  React UI                 │ 
     │                           │ 
     │ ┌───────────────────────┐ │ 
     │ │ @wormhole/connect/SDK │ │ 
     │ │                       │ │ 
     │ │ Token Bridge Route    │ │ 
     │ │ CCTP Route            │ │ 
     │ │ Portico Route...      │ │ 
     │ │                       │ │ 
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


---

# How to use the `Route` interface
                                                                                                                  
                                                                                                                                  
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


# `Route` plugins

`Route` plugins can come from any repository / NPM package.

With Wormhole Connect for example, integrators can add arbitrary `Route` plugins via Connect's config.


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

# `Route` interface today


talk about protocols and platforms

track method

leave 10-15 mins

places we could use help

near and algorand?

touch on adv tools deprecation




