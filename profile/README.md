# Cadence Pay

We build **Stellar Recurring**: pull-based subscription billing on Stellar/Soroban. A merchant defines a plan, a subscriber authorizes a capped, expiring token allowance, and a keeper calls a permissionless `charge` once per period. It is **not** payment streaming: one discrete charge per period.

**Status:** runs end to end on the Stellar **testnet**: a merchant creates a plan, a subscriber subscribes through the checkout widget's flow, the keeper charges it automatically twice, and the subscriber cancels. The transaction hashes are in [recurring-contracts/docs/DEMO.md](https://github.com/Cadence-Pay/recurring-contracts/blob/main/docs/DEMO.md). Nothing runs on mainnet, **the contract has not been audited**, and no person has yet used the dashboard or the widget with a real browser wallet.

## Repositories

| Repository                                                                | What it is                                                                                |
| ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| [recurring-contracts](https://github.com/Cadence-Pay/recurring-contracts) | Soroban contract (Rust): plans, subscriptions, `charge`, `cancel`                         |
| [recurring-keeper](https://github.com/Cadence-Pay/recurring-keeper)       | Node service: syncs the chain, charges due subscriptions, sends webhooks, serves a read API |
| [recurring-dashboard](https://github.com/Cadence-Pay/recurring-dashboard) | Next.js dashboard for merchants                                                           |
| [recurring-checkout](https://github.com/Cadence-Pay/recurring-checkout)   | Embeddable React subscribe button                                                         |

## How it fits together

```
 subscriber's browser                 merchant's browser
 +---------------------+              +----------------------+
 |  recurring-checkout |              |  recurring-dashboard |
 |  <SubscribeButton>  |              |  plans, subscribers, |
 |  approve + subscribe|              |  charges, revenue    |
 +----------+----------+              +-----+-----------+----+
            | signs in wallet                | reads     | signs in wallet
            |                                v           |
            |                      +------------------+  |
            |                      | recurring-keeper |  |
            |                      |  read API, sync, |  |
            |                      |  charge loop,    |--+--> merchant webhooks
            |                      |  webhooks        |  |     (HMAC signed)
            |                      +--------+---------+  |
            v                               | charge()   v
      +---------------------------------------------------------+
      |          Stellar network (Soroban RPC)                  |
      |  recurring-contracts: plans, subscriptions, charge,     |
      |  cancel      +  a SEP-41 token (allowance, transfer)    |
      +---------------------------------------------------------+
```

1. A merchant creates a **plan** (token, amount, period) with the dashboard.
2. A subscriber uses the widget: it approves a capped, expiring token allowance and subscribes. The first period is paid at once.
3. Each period, the **keeper** (or anyone, since `charge` is permissionless) calls `charge`, which pulls the plan amount with the allowance. If the payment fails, the call reverts; the keeper retries, then reports `charge.failed` to the merchant.
4. Either side can cancel at any time.

## Contributing

Each repository has a contributing guide and open issues scoped for contributors, each naming the place in the code where the work plugs in. Report security problems privately through the repository's Security tab, never in a public issue.
