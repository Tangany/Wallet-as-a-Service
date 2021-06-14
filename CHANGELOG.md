# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.8.2] - 2021-06-14

### Changed

- Support EIP-155 transactions in Ethereum custom networks

### Fixed

- Fix "Could not process invalid cookie value" responses when communicating with certain custom networks
- Fix certain Bitcoin recipient addresses declined with `Invalid recipient address`
- Fix prematurely purging `Completed` asynchronous request IDs

## [1.8.1] - 2021-06-07

### Changed

- Check Bitcoin transactions for a valid fee configuration before the asynchronous hand-over

### Fixed

- Add missing tag query params to pagination links in `GET /wallets`
- Fix `GET /wallet/:wallet/sign` not throwing expected errors for invalid encodings
- Fix async Bitcoin transactions stuck in the `awaiting transaction confirmation` stage and time out eventually even
  though the transaction is mined successfully
- Fix bug where a deleted wallet could be updated
- Fix incorrect error message about gas price for gas-less custom Ethereum networks
- Fix a rare case where an async Ethereum request status may indicate a failure (error code `828065450941776`) even
  though the transaction is mined successfully
- Fix cases where async Ethereum transaction processes may unexpectedly fail due to various recoverable errors

### Added

- Add support for tag value queries (e.g. `?tag["my-tag"]="my-value"`) in `GET /wallets`

## [1.8.0] - 2021-03-10

### Added

- (**opt-in beta**) Monitor wallet-based transactions and get notified in real-time about transaction status changes via
  a webhook
    - Create new wallet-based transaction with extensive filtering options: `POST /eth/wallet/:wallet/monitors`
    - Replace exiting monitors: `PUT /eth/wallet/:wallet/monitor/:monitor`
    - Partially update existing monitors `PATCH /eth/wallet/:wallet/monitor/:monitor`. Supports RFC 6902
    - Delete monitor `DELETE /eth/wallet/:wallet/monitor/:monitor`
    - Query monitor status `GET /eth/wallet/:wallet/monitor/:monitor`
    - List all monitors for current wallet `GET /eth/wallet/:wallet/monitors`
    - List all monitors `GET /eth/monitors`
- Added a new header `tangany-ethereum-chain-id` to enforce desired chain ID in a custom Ethereum network
- Included a new field `data` to Ethereum smart contract estimation response
  of `POST /eth/contract/:contract/:wallet/estimate-fee` that converts the smart contract ABI call to hexadecimal data
- Categorize wallets and add custom meta data via wallet tags
    - Add tags to wallets and update exiting tags using the new endpoint  `PATCH /wallet/:wallet` that supports atomic
      updates via [RFC 6902 (JSON Patch)](https://tools.ietf.org/html/rfc6902)
- Added new, improved `GET /wallets` endpoint to query wallets
    - Includes pagination and sorting options
    - Allows tag filtering using the `?tag` & `?xtag` query parameters

### Changed

- Extended responses of sending Ethereum endpoints with `nonce` and HATEOAS links to adjacent resources
- Requests using the custom network URL in the `tangany-ethereum-network` header are now checked against a
  customer-specific whitelist. Contact the support team to enable new custom Ethereum networks for a user account.
  Custom Ethereum networks that were used by existing customers were automatically whitelisted before the launch

### Fixed

- Improved the behaviour of sent Ethereum transactions in asynchronous requests. `unknown` transactions are now rejected
  after a shorter time span
- Fixed a case where `tangany-ethereum-nonce` header were not enforced in an Ethereum transaction if the provided nonce
  was less than the current blockchain nonce
- Vastly improved the throughput of asynchronous Ethereum transactions from a single wallet
- Reduced the likelihood of overlapping nonces when sending multiple Ethereum transactions from a single wallet
  simultaneously

### Deprecated

- Soft deprecate `POST /wallet`. This endpoint will from now on be documented as `POST /wallets`.
- Soft deprecate `GET /wallet`. Migrate to compatible endpoint with extended features `GET /wallets`

## [1.7.1] - 2021-02-01

### Changed

- Improve the information content in error messages

### Fixed

- Fix wallet count of hsm-secured wallets in `GET /wallets` always returns 0
- Fix wallet substitution in contract calls when argument is a deep nested array of type wallet

### Added

- Add error messages in case of invalid argument filters in contract event searches

### Deprecated

- **Synchronous transaction endpoints are deprecated and will permanently cease function on 01 July 2021. We advise a
  timely migration to asynchronous endpoint variants to prevent possible service interruptions**

| Deprecated endpoint | API URL |
| :--- | :--- |
| Ethereum - Make wallet transaction | `POST /eth/wallet/:wallet/send` |
| Ethereum - Sign wallet transaction | `POST /eth/wallet/:wallet/sign` |
| Ethereum ERC20 - Make ERC20 token transaction | `POST /eth/erc20/:token/:wallet/send` |
| Ethereum ERC20 - Execute ERC20 "transferFrom" | `POST /eth/erc20/:token/:wallet/transfer-from` |
| Ethereum ERC20 - Execute ERC20 "approve" | `POST /eth/erc20/:token/:wallet/approve` |
| Ethereum ERC20 - Execute ERC20 "mint" | `POST /eth/erc20/:token/:wallet/mint` |
| Ethereum ERC20 - Execute ERC20 "burn" | `POST /eth/erc20/:token/:wallet/burn` |
| Bitcoin - Make wallet transaction |  `POST /btc/wallet/:wallet/send` |
| Bitcoin - Sign wallet transaction | `POST /btc/wallet/:wallet/sign` |

## [1.7.0] - 2020-12-22

### Changed

- Display total number of wallets per vault in `GET /wallet`
- Extend the wallet status with the public secp256k1 key
- Return `AUTO` response header if `tangany-bitcoin-max-fee-rate` header was not provided in the client request
- Allow positive integer values in the `tangany-bitcoin-tx-confirmations` configuration header
  matching `tangany-ethereum-tx-confirmations`
- Make `inputs` body parameter optional for all Ethereum smart contract endpoints

### Fixed

- A custom Ethereum RPC Network URL passed via `tangany-ethereum-network` won't get cut-off after the host portion
  anymore
- Fix bug in Ethereum Event response where `logIndex` was `null` instead of `0`
- Fix broken session cookies for public Ethereum networks
- Fix a rare case where rapid transactions from the same wallet may be dropped from the transaction queue
- Fix bug in BTC amounts where integers in decimal notations like `1.000` get rejected

### Added

- Add sorting options `transactionIndex` and `transactionIndexDesc` to read tx api endpoints
- Send asynchronous Bitcoin wallet transaction via `POST /btc/wallet/:wallet/send-async`
- Query Ethereum transaction events using smart contract call arguments in `GET /eth/contract/:contract/events`
- Add support for array type arguments in Ethereum smart contract endpoints
- Add payload signing endpoint `POST /wallet/:wallet/sign`
- Add signature verification endpoint `POST /wallet/:wallet/verify`

## [1.6.1] - 2020-08-12

### Changed

- Fix empty return value when executing a `bool` response type smart contract call

### Fixed

- Fix misleading error message for recipients address mismatch when using the `wallet` request body parameter

### Added

- Add convenience route `GET /eth/contract/:contract/call/:method` for calling Ethereum smart contract methods without
  arguments
- Add optional query parameter `?type` to define the return types for smart contract method calling
  routes `GET /eth/contract/:contract/call/:method` and `GET /eth/contract/:contract/:wallet/call/:method`. The `?type`
  query accepts valid Solidity data types and defaults to is `uint256`

## [1.6.0] - 2020-08-10

### Changed

- Enable `amount` argument for `POST /eth/contract/:contract/:wallet/send-async` to allow interacting with payable smart
  contract methods. This also changes the `amount` argument to become an optional argument in all Ethereum sending
  endpoints (defaults to 0)

### Fixed

- Allow empty `inputs` in request bodies of Ethereum contract endpoints

### Added

- Call Ethereum smart contract methods via `POST /eth/contract/:contract/call`
  , `POST /eth/contract/:contract/:wallet/call` and `GET /eth/contract/:contract/:wallet/call/:method`
- Sweep Bitcoin wallet by transferring all funds to address via `POST /btc/wallet/:wallet/sweep-async`
- Wallet names of the current `tangany-vault-url` can be used as a substitution for its Ethereum addresses in compatible
  endpoints

## [1.5.0-hotfix.1] - 2020-07-23

### Changed

### Fixed

- Fix erratic transaction events search results when log index is 0
- Fix `GET /eth/transaction/:hash` fail with "Invalid params" with fetching status for a new transaction without a block
  number

### Added

## [1.5.0-hotfix.0] - 2020-07-13

### Changed

### Fixed

- Fix ERC20 endpoint transactions fail with "invalid number value" when trying to send / mint / burn / approveFrom token
  amounts over "1000"

### Added

## [1.5.0] - 2020-07-01

### Changed

- Enhance `GET /eth/transaction/:hash` to return more relevant transaction data
- Add `blockHash` property to `GET /btc/hash/:hash`

### Fixed

- Improve the error message when sending less than 1 wei in an Ethereum transaction
- Return 404 for unknown async request ids via `GET /request/{id}`
- Fix a rare case where `blockNr` property is missing from the `GET /btc/transaction/:hash` response
- Fix ancient Ethereum transactions being reported as erroneous

### Added

- Support new custom header `tangany-ethereum-gas` to enforce user defined gas amount for Ethereum transactions
- Support new custom header `tangany-ethereum-nonce` to enforce given nonce in Ethereum transactions
- Estimate Ethereum transaction fees via `POST /eth/wallet/:wallet/estimate-fee`
  and `POST /eth/contract/:contract/:wallet/estimate-fee`
- Explore Ethereum transactions via `GET /eth/transactions` and `GET /eth/wallet/:wallet/transactions`
- Examine Ethereum and Bitcoin node status via `GET /eth/status` and `GET /btc/status`
- Explore Ethereum smart contract events via `GET /eth/contract/:contract/events` and query individual events
  via `GET /eth/transaction/:hash/event/:index`

## [1.4.0-hotfix.0] - 2020-06-02

### Changed

- Improve performance

## [1.4.0] - 2020-04-02

### Changed

- Disable UTF-8 conversion of Ethereum data payload to allow interacting with arbitrary smart contracts
  via `POST /eth/wallet/{wallet}/send`. Now only hexadecimal strings are accepted for `data`

### Fixed

- Limit allowed decimal places for transaction amount
- Fix Bitcoin endpoints not processing the affinity cookie sent in a request
- Support new custom header `tangany-ethereum-gas-price`

### Added

- Add support for sending ethereum asynchronously via `POST /eth/{wallet}/send-async`
- Add status endpoint for asynchronous requests: `GET /request/{id}`
- Add support for `tangany-use-gas-tank` header for ERC20 methods to support ether-less Token workflows on ethereum
  public networks
- Add smart contract support via `POST /eth/contract/:contract/:wallet/send-async`
- Add ethereum signing endpoint: `POST /eth/wallet/{wallet}/sign`
- Add bitcoin signing endpoint: `POST /btc/wallet/{wallet}/sign`

## [1.3.0] - 2019-12-04

### Changed

- Change 404 response for unknown hashes to `unknown` status for `GET /eth/transaction/{hash}`

### Fixed

- Improve latency for sending Bitcoin from wallets with a large transaction history

### Added

- Add call configuration metadata to the response headers
- Add optional `data` field to `POST /eth/{wallet}/send`
- Add `data` to the `GET /eth/transaction/{hash}` response
- Add `tangany-ethereum-tx-confirmations` header to evaluate transaction validity
- Add `status` to the `GET /eth/transaction/{hash}` response to get insights about the transaction lifecycle
- Add status `unknown` to `GET /btc/transaction/{hash}` response
- Add `blockNr` to `GET /btc/transaction/{hash}` response
- Add session cookie to maintain a sticky session with the backend blockchain node
- Add support for `HEAD` connection method to `HEAD /eth/transaction/{hash}` & `HEAD /btc/transaction/{hash}` to fetch
  the session cookie

## [1.2.1-hotfix.2] - 2019-11-11

### Changed

- Improved Bitcoin performance for wallets with lots of UTXOs

## [1.2.1-hotfix.1] - 2019-10-17

### Fixed

- Fixed the request exception in `POST /btc/{wallet}/send` for trying to send below minimum allowed amount in the
  Bitcoin protocol

## [1.2.1-hotfix.0] - 2019-10-02

### Changed

- Change internal monitoring

## [1.2.1] - 2019-08-13

### Changed

- Changed `GET /btc/transaction/{hash}` to return a 404 status when given transaction cannot be found

### Fixed

- Fixed `POST /eth/erc20/{token}/{wallet}/approve` asserting the Ether balance instead of token balance to execute the
  approve transaction

### Added

- Added optional "to" parameter to  `POST /eth/erc20/{token}/{wallet}/mint` to enable minting ERC20 tokens to arbitrary
  addresses

## [1.2.0-hotfix.2] - 2019-07-29

### Fixed

- Fixed missing functionality for `POST /eth/erc20/{token}/{wallet}/burn`

## [1.2.0-hotfix.1] - 2019-07-26

### Fixed

- Fixed the bitcoin header `tangany-bitcoin-max-fee-rate` not accepting a string-type input
- Fixed `POST /btc/wallet/{wallet}/estimate-fee` to always execute despite the `tangany-bitcoin-max-fee-rate` header
  setting

## [1.2.0] - 2019-07-26

### Changed

- Made the HTTP header `tangany-ethereum-network` optional by using the default value "mainnet" for Ethereum network

### Fixed

### Added

- Added `POST /eth/erc20/{token}/{wallet}/approve` and  `POST /eth/erc20/{token}/{wallet}/transfer-from` to support the
  matching standard Ethereum ERC20 methods
- Added `POST /eth/erc20/{token}/{wallet}/burn` and  `POST /eth/erc20/{token}/{wallet}/mint` to support the matching
  extended Ethereum ERC20 methods
- Added support for Bitcoin

## [1.1.3-hotfix.2] - 2019-06-18

### Fixed

- Fix internal headers

## [1.1.3-hotfix.1] - 2019-06-18

### Changed

- Remove internal tests

## [1.1.3-hotfix.0] - 2019-05-15

### Fixed

- Fix the server response for attempting to make an Ethereum transaction with insufficient funds

## [1.1.3] - 2019-03-26

### Changed

- Add paging to `GET /wallet` via the `skiptoken` query parameter to prevent long running requests when trying to query
  hundreds of wallets

## [1.1.2] - 2019-03-22

### Fixed

- Fix `GET /eth/erc20/{token}/{wallet}` returning a decimal value instead of float for token balance

## [1.1.1] - 2019-03-22

### Added

- Add HTTP header `tangany-ethereum-tx-speed` to affect the additional gas that is added to the network gas to
  accelerate the transaction

## [1.1.0] - 2019-03-21

### Added

- Add support for Ethereum RPC networks via `tangany-ethereum-network` HTTP header

## [1.0.1] - 2019-03-18

### Fixed

- Fix `GET /wallet` fail with "allKeys.map is not a function" when zero wallets are available
- Fix `POST /eth/erc20/{token}/{wallet}/send` to return status 201 on success instead of 200

## [1.0.0] - 2019-03-15

### Added

- Initial release
