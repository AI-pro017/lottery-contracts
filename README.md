# Lottery Contracts

A raffle smart contract for Juno, written in Rust with CosmWasm.

Admins open raffle rounds, players buy in with JUNO or a CW20 token, and when the round ends the contract picks the winners and works out their payouts from a split set when the round was created.

It's still a work in progress. See the known issues below before using it anywhere real.

## How a round works

1. An admin calls `begin_raffle_round` with:
   - `expire_type`: how long the round stays open. `0` is 30 minutes, `1` is an hour, `2` is a day and `3` is a week.
   - `minimum_stake`: the entry price.
   - `winners_distribution`: how the pot is split between winners, for example `[50, 30, 20]` for three winners.
   - `pay_token`: leave empty to use JUNO, or set a CW20 contract address to use that token.
2. Players join with `join_raffle_round_by_juno { id }` and send the stake in `ujuno`. For CW20 rounds, they send the tokens to this contract with a CW20 `send` message that wraps `{ "id": <round id> }`.
3. Once the round has expired, an admin calls `end_raffle_round { id }`. The contract draws the winners, works out each payout from the distribution and saves the result on the round.

Queries:

- `get_raffle_info { id }` returns a round's settings, players, winners and payouts.
- `get_count {}` returns how many rounds have been created.
- `get_total_deposit {}` returns the JUNO the contract is holding.

## Known issues

- The payout messages in `end_raffle_round` are built with `res.clone().add_message(...)`, but the result is never kept. Winners and amounts are recorded, but the transfers aren't actually sent yet.
- Payouts are based on the contract's whole JUNO balance, not just the stakes for that round. With several rounds running at once, or CW20 rounds, the numbers will be off.
- Winners are drawn with replacement, so one player can win more than one slot.
- Randomness comes from a ChaCha RNG seeded with block data and the caller's details. It's fine for a testnet, but it can be predicted on chain. For real money you'd want a VRF or a drand beacon.

`functions.md` has the original feature checklist.

## Building and testing

You'll need Rust 1.58 or newer with the `wasm32-unknown-unknown` target.

```bash
git clone https://github.com/AI-pro017/lottery-contracts.git
cd lottery-contracts
cargo test
cargo wasm
```

For a size optimized build ready to upload to a chain:

```bash
cargo run-script optimize
```

That runs the `cosmwasm/rust-optimizer` Docker image and writes the `.wasm` to `artifacts/`. The JSON schemas for every message are in `schema/` and can be regenerated with `cargo schema`.

`Developing.md`, `Publishing.md` and `Importing.md` have more general notes on building, deploying and reusing CosmWasm contracts.

## License

Apache 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
