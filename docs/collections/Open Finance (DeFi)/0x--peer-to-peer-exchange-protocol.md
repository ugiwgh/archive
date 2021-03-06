---
title: 0x  Peer-to-peer exchange protocol
summary: 0x is an open protocol that facilitates peer-to-peer exchange of digital assets on the Ethereum blockchain. 0x is open source, free to use, and provides a drop-in exchange solution for developers to build on top of. Developers use 0x protocol to build decentralized exchanges (DEXs), marketplaces for digital collectibles, and to integrate exchange functionality into wallets. - Description from ethhub.io This article originally appeared in the 0x developer portal Build a Relayer A relayer is any p
authors:
  - Kauri Team (@kauri)
date: 2019-04-08
some_url: 
---

# 0x  Peer-to-peer exchange protocol

![](https://ipfs.infura.io/ipfs/QmeM4VKZzMJL79gNpEAmUj57PzYq15bVc5DKdJYnfvvU89)


> 0x is an open protocol that facilitates peer-to-peer exchange of digital assets on the Ethereum blockchain. 0x is open source, free to use, and provides a drop-in exchange solution for developers to build on top of. Developers use 0x protocol to build decentralized exchanges (DEXs), marketplaces for digital collectibles, and to integrate exchange functionality into wallets. - Description from [ethhub.io](https://docs.ethhub.io/built-on-ethereum/open-finance/0x-protocol/#0x-protocol-overview)

_This article originally appeared in the [0x developer portal](https://0x.org/wiki#Build-A-Relayer)_

## Build a Relayer

A relayer is any party or entity which hosts an off-chain orderbook. They provide a way for users to add, remove and update this orderbook through an API, GUI or both. In doing so, relayers help traders discover counter-parties and ferry cryptographically signed orders between them. Once two parties agree on the terms of an order, the order is settled directly on the Ethereum blockchain via the 0x protocol smart contracts.

#### Advantages of decentralized exchange

There are many advantages to a decentralized exchange over centralized exchange. One of the largest problems with centralized Exchanges is that they must hold and secure the funds of all the traders on their platform. This causes there to be a single point of failure that if hacked or mismanaged, means that all these traders could lose their funds. Hundreds of millions have already been stolen in this way. With a decentralized exchange, traders do not need to deposit funds with a centralized entity, but instead can trade directly from their individual wallets (including hardware wallets!). This means the user is in control of their funds at all times, eliminating this single point of failure from the equation.

#### 0x protocol overview

In 0x protocol, orders are transported off-chain over any arbitrary medium, massively reducing gas costs and reducing blockchain bloat. Relayers help broadcast orders and can choose to collect a fee each time they facilitate a transaction. Anyone can build a relayer.

The simplest example of a relayer is a website allowing users to create, discover and fill orders. The relayer must build out a UI and host a backend database to provide this functionality.

<div align="center">
    <img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/relayer_diagram.png" style="padding-bottom: 20px; padding-top: 20px; max-width: 342px;" width="80%" />
</div>

To simplify the process of interacting with the 0x protocol, we have written a Javascript/Typescript library called [0x.js](http://0x.org/docs/0x.js). This library helps relayers interact with the 0x protocol smart contracts through a higher-level, easier to use interface. It also provides many useful utilities for hashing, signing, validating, and serializing 0x orders. Additionally, we have built the [0x-launch-kit](https://github.com/0xproject/0x-launch-kit), an open-source, free-to-use API-only 0x relayer template that you can use as a starting point for your own project.

Before getting started with [0x.js](http://0x.org/docs/0x.js), [0x-launch-kit](https://github.com/0xproject/0x-launch-kit) or the 0x protocol, it is helpful to introduce a few concepts. There are two parties involved in every trade, a maker and a taker. The maker creates an order for an amount of TokenA in exchange for an amount of TokenB. The maker then submits these to a relayer. Takers discover orders via a relayer and fill them by sending them directly to the 0x protocol smart contracts. The 0x protocol smart contracts performs an atomic swap, exchanging the maker and taker tokens.

#### Order

Below is an interface description of the 0x protocol order format:

```typescript
interface Order {
    senderAddress: string;
    // Ethereum address of the Maker
    makerAddress: string;
    // Ethereum address of the Taker. If no address specified, anyone can fill the order.
    takerAddress: string;
    // How many ZRX the Maker will pay as a fee to the relayer
    makerFee: BigNumber;
    // How many ZRX the Taker will pay as a fee to the relayer
    takerFee: BigNumber;
    // The amount of an asset the Maker is offering to exchange
    makerAssetAmount: BigNumber;
    // The amount of an asset the Maker is willing to accept in return
    takerAssetAmount: BigNumber;
    // The identifying data about the asset the Maker is offering
    makerAssetData: string;
    // The identifying data about the asset the Maker is requesting in return
    takerAssetData: string;
    // A salt to guarantee OrderHash uniqueness. Usually a milisecond timestamp of when order was made
    salt: BigNumber;
    // The address of the 0x protocol exchange smart contract
    exchangeAddress: string;
    // The address (user or smart contract) that will receive the fees
    feeRecipientAddress: string;
    // When the order will expire (unix timestamp in seconds)
    expirationTimeSeconds: BigNumber;
}
```

It is a relayer's job to collect cryptographically signed versions of these orders into an off-chain database. This collection of orders is what we refer to as an orderbook. A relayer displays their orderbook to potential takers. The incentive here is for a relayer to collect fees from the orders they host. By specifying themselves as the fee recipient, relayers can earn fees in ZRX tokens.

Check out this tutorial on how to [Create, Validate, and Fill Orders](https://0x.org/wiki#Create,-Validate,-Fill-Order). If you want to jump straight into a working relayer codebase, check out [0x-launch-kit](https://github.com/0xProject/0x-launch-kit/).

If you want to take a deep-dive into the 0x protocol, take a look at the [0x Protocol Specification](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md).

#### Relayer strategies

The 0x protocol leaves room for some flexibility on how exactly relayers operate. There are a few different strategies relayers have employed so far, each with their respective advantages and drawbacks. You can read more about the different strategies in the [Relayer Strategies](https://0x.org/wiki#Matching) section of the wiki to discover the strategy that suits your needs the best.

#### Shared liquidity

Because all relayers represent orders using the 0x protocol order format, an order created on one relayer (using the open-orderbook strategy) can be filled by users on another relayer. What this means is that rather than each relayer having a siloed liquidity pool, they can share orders to create a shared liquidity pool. New relayers can bootstrap their liquidity off of existing relayers, immediately becoming an interesting place to trade. It is important to note that fees always go to the relayer where the order was first submitted. We have defined a [Standard Relayer API](https://github.com/0xProject/standard-relayer-api) to help relayers share liquidity and to simplify the integration process for market makers, providing a unified interface for them to build against.

#### Pruning your orderbooks

Over time, orders may expire, partially filled, cancelled or no longer fillable. It is best to keep your orderbook free of expired or unfillable orders. Iterating over the orderbook periodically and checking each order's validity is a simple way to accomplish this. For a more advanced and efficient approach, you could use an [OrderWatcher](https://0x.org/wiki#0x-OrderWatcher). [0x-launch-kit](https://github.com/0xProject/0x-launch-kit/) is built using OrderWatcher out-of-the-box.

#### Next steps

Now that you have a high level idea of what a relayer does, it's time to get started learning how they work. Check out [0x-launch-kit](https://github.com/0xProject/0x-launch-kit/) for a fully-working relayer example, or dive into the [Create, Validate, and Fill Orders](https://0x.org/wiki#Create,-Validate,-Fill-Order) tutorial to master the basics of dealing with 0x orders. You may also want to decide on a [Relayer Strategy](https://0x.org/wiki#Open-Orderbook). We recommend the open orderbook strategy. If you're looking for more orders to add to your orderbook, take a look at the [Standard Relayer API](https://github.com/0xProject/standard-relayer-api) and [0x Connect](https://0x.org/docs/connect). There are several relayers that have already implemented it and are broadcasting orders. Remember to keep your orderbook free of stale orders by using an [OrderWatcher](https://0x.org/wiki#0x-OrderWatcher).


---

- **Kauri original title:** 0x  Peer-to-peer exchange protocol
- **Kauri original link:** https://kauri.io/0x-peertopeer-exchange-protocol/81054d98abe9412d87f91cd86719e90f/a
- **Kauri original author:** Kauri Team (@kauri)
- **Kauri original Publication date:** 2019-04-08
- **Kauri original tags:** dex, open-finance
- **Kauri original hash:** QmZB1EVYcW7gP3vHWjxWwuWgrf4E6dR9sHXmXtdQQEopHz
- **Kauri original checkpoint:** QmRS3wCLX2MRi62bg9NTM89qNkgm3XjpKXciLvCKAr1f1g



