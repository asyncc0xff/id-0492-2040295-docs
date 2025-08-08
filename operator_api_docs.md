# Operator API v 2.7.0

## Glossary

| Term | Definition |
|------|------------|
| Casino-partner | A side that integrates 100hp's games onto its website. |
| 100hp gaming | A side that acts as an intermediary between Casino-provider and Casino Wallet. Stores games, settings and etc. |
| Casino Wallet | A side that stores players, their balances and confirms transactions. |

## General Description

The system works by using web services that are JSON data based.

100hp gaming is a company specializing in developing and providing its games to online casinos and gaming aggregators.

The current document is intended for technical specialists and describes the following:

- A way to get a list of games and a method to get a link to launch the game.
- Methods that manage financial transactions during the game and use the player's account.

JSON format is used as the format of methods.

## Integration scheme

Casino-partner and Casino Wallet communicate with 100hp gaming API via HTTP requests by calling the methods described in this document.

### System Flow Diagram

```
Player -> Casino-partner -> 100hp gaming -> Casino Wallet
  |           |                |              |
  |      getting a list of games              |
  |      request /games/v1/list               |
  |      <----- response with a list of games |
  |           |                |              |
  |      open the game                        |
  |      request /games/v1/launch             |
  |           |                |              |
  |           |         launch the game       |
  |           |         request /games/v1/launch -> player authorization
  |           |                |              | request /auth
  |      <-- redirect to launch URL           |
  |           |                |          <-- response with a user and balance info
  |           |                |              |
  |           |         player authorization  |
  |           |         request /auth         |
  |           |                |          <-- response with a user and balance info
  |           |                |              |
  |           |                |         withdrawal money
  |           |                |         request /withdrawal
  |           |                |          <-- response with a user and balance info
  |           |                |              |
  |       [all successful]         deposit money
  |           |                |         request /deposit
  |           |                |          <-- response with a user and balance info
  |           |                |              |
  |    [in case of problem]       rollback money
  |           |                |         request /rollback
  |           |                |          <-- response with a user and balance info
  |           |                |              |
  |           |                |         get balance
  |           |                |         request /balance
  |           |                |          <-- response with a user and balance info
  |           |                |              |
  |       [optional]            promo win payout
  |           |                |         request /bonusWin
  |           |                |          <-- response with a user and balance info
```

## Preparation for integration

100hp gaming before starting the integration will provide the Casino-partner with:

1. **HOSTNAME_URL** is the 100hp gaming address that the Casino-partner will access to get the list of games and launch the games.

2. **SECRET** is the secret key that the Casino-partner will pass to 100hp gaming in requests to the 100hp gaming API, and 100hp gaming will pass it in all requests to Casino Wallet API. It is necessary for authentication of requests.

To start the integration, the Casino-partner requires:

1. Develop a Casino Wallet application that will handle methods called by 100hp gaming. The application should develop according to this documentation Casino Wallet API section). If you encounter any problems, please contact the support team: partners@100hp.game.

2. Provide **WALLET_URL**, i.e. address of Casino Wallet API, which will be accessed by 100hp gaming.

> **Note:** If your system does not support any non-required parameters present in the methods, there is no need to send them. Otherwise, you should send them.

---

# 100hp gaming API

The section describes the methods that the Casino-partner implements on its side to request from 100hp gaming:

- **Game List** is the list of games available to the Casino-partner and their parameters.
- **Launch Game URL** is the URL address to run the game.

## Getting a list of games

The method allows Casino-partner to get a list of all available games.

### Endpoint
```
GET /games/v1/list
```

### Request parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| secret | string | ✅ | 100hp gaming provides a secret key for partners in preparation for integration. | An authentication key. It is a query parameter. |

### Example of request

```bash
curl --location 'https://example.com/games/v1/list?secret=<secret>'
```

### Response parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| id | string (Format uuid) | ✅ | 018bb0cd-54a1-72a0-8d59-855841991173 | Game identifier. |
| name | string | ✅ | Game_name_1 | Game name. |
| logoUrl | string | ✅ | image.png | This field is for internal use only and should not be relied upon in your integrations or applications. |
| developer | object | ✅ | | |
| id | string | ✅ | 8da630c9-6378-4f20-b864-320287ea4ebf | Provider identificator |
| name | string | ✅ | developerX | Provider name |
| ownerName | string | ✅ | 100hp gaming | Provider name. Used for backwards compatibility |
| monetizationType | string | ✅ | rake of profit | Monetization type. |
| category | string | ✅ | Crash games | Game type |
| createdAt | string (Format ISO 8601) | ✅ | 2023-12-25T09:59:39.000Z | The game's creation date. |
| updatedAt | string (Format ISO 8601) | ✅ | 2023-12-25T09:59:39.000Z | The game's update date. |
| supportedCurrencies | array | ✅ | ['USD', 'EUR'] | Supported currencies. |
| supportedLanguage | array | ✅ | ["ru", "en"] | Supported languages. |
| hasDemo | string | ✅ | true or false | The presence of a demo mode in the game. |
| hasMobileVersion | string | ✅ | true or false | The presence of a mobile demo mode in the game. |
| hidden | string | ✅ | true or false | Usually for debug games. |

> **Note:** For example, in the `/auth` method, the response includes non-required parameters: `status`, `type`. They are optional, but if you have a value for these parameters, you should send them.

### Example of response

```json
[
  {
    "id": "4d2d758e-5d17-4c65-bc1d-e31d1dd220ea",
    "name": "Lucky Loot",
    "developer": {
      "id": "8da630c9-6378-4f20-b864-320287ea4ebf",
      "name": "developerX"
    },
    "ownerName": "Lucky Loot",
    "logoUrl": "image.png",
    "hidden": false,
    "monetizationType": "profit",
    "category": "Crash game",
    "hasDemo": true,
    "hasMobileVersion": true,
    "hasDedicatedMobileUrl": true,
    "supportedCurrencies": [
      "RUB",
      "USD"
    ],
    "supportedLanguage": [
      "ru",
      "en"
    ],
    "createdAt": "2021-05-14T16:52:56.950Z",
    "updatedAt": "2021-05-14T16:52:56.950Z"
  }
]
```

### Error options

#### Request params validation error
```json
{
  "data": {
    "secret": [
      "secret must be a string",
      "secret should not be empty"
    ]
  },
  "message": "Request params validation error",
  "errCode": 400
}
```

#### Unauthorized
```json
{
  "data": "",
  "message": "Unauthorized",
  "errCode": 401
}
```

#### Internal Server Error
```json
{
  "data": "",
  "message": "Internal Server Error",
  "errCode": 500
}
```

---

## The game launch

The method allows Casino-partner to get the launch URL.

### Endpoint
```
POST /games/v1/launch
```

### Request parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| secretKey | string | ✅ | 100hp gaming provides a secret key for partners in preparation for integration. | An authentication key. |
| sessionId | string | ✅ | 4affa487-c147-4e48-ae90-903223f17d5b | Unique casino-partner generated session. Optional only when launching demo mode. Must be <= 255 characters |
| isDemo | boolean | ✅ | true or false | Launch the game in demo mode. |
| isMobile | boolean | ✅ | true or false | Launch the game in mobile version. |
| gameId | string (Format uuid) | ✅ | 018c877c-7f9f-7334-8d0b-df9c7a52dda9 | Game identifier. |
| language | string (Format ISO 639-1) | | en | The game interface language. The language must be supported by the game. Default language - english. |
| ip | string | | 8.8.8.8 | Player's IP address. Must be <= 255 characters |
| browser | string | | | Player's browser information. Must be <= 255 characters |
| geo | string (Format ISO-3166-1 alpha-2 codes) | | DE | Player's country. Must be <= 2 characters |

### Example of request

```bash
curl --location 'https://example.com/games/v1/launch' \
--header 'Content-Type: application/json' \
--data '{
  "sessionId": "4affa487-c147-4e48-ae90-903223f17d5b",
  "isDemo": false,
  "isMobile": true,
  "secretKey": "<secret>",
  "gameId": "018c877c-7f9f-7334-8d0b-df9c7a52dda9",
  "language": "en",
  "ip": "8.8.8.8",
  "browser": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36",
  "geo": "DE"
}'
```

### Response parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| gameUrl | string | ✅ | | The link to launch the game. |

### Example of response

```json
{
  "gameUrl": "https://example.com/4c38e473-7582-4ba6-8de1-e3b6e4369561"
}
```

### Errors options

#### Session already exists
```json
{
  "data": "",
  "message": "Session already exists.",
  "errCode": 400
}
```

#### Unsupported demo for this game
```json
{
  "data": "",
  "message": "Unsupported demo for this game.",
  "errCode": 400
}
```

#### Unsupported mobile version for this game
```json
{
  "data": "",
  "message": "Unsupported mobile version for this game.",
  "errCode": 400
}
```

#### Settings for game marked as hidden
```json
{
  "data": "",
  "message": "Settings for game marked as hidden.",
  "errCode": 400
}
```

#### Request params validation error
```json
{
  "data": "",
  "message": "Unsupported mobile version for this game.",
  "errCode": 400
}
```

#### Unauthorized
```json
{
  "data": "",
  "message": "Unauthorized",
  "errCode": 401
}
```

#### Game by id not found
```json
{
  "data": "",
  "message": "Game by id not found.",
  "errCode": 404
}
```

#### Internal Server Error
```json
{
  "data": "",
  "message": "Internal Server Error",
  "errCode": 500
}
```

---

# Casino Wallet API

In the integration process, Casino Wallet development takes place on the Casino-partner's side. The application should develop according to 100hp gaming documentation. If any problems or questions arise, you can contact the support team: partners@100hp.game

Casino Wallet is an application that should be able to handle the following requests from 100hp gaming to Casino Wallet API:

- **Player authorization** is a method that is called when the game starts to authenticate the user.
- **Withdrawal money** is a method, which is called at the moment when a player makes a bet. It is used to debit funds from the balance.
- **Deposit money** is a method, which is called when a user wins. It is used to top up the player's balance.
- **Rollback money** is a method that is called if there is no response or there are errors from Casino Wallet to the transaction request or the response contains an error. It is used to cancel the last transaction with the player's balance.
- **Get balance** is a method that returns the player's balance.
- **Get user metadata** is a method that returns players metadata.
- **Credit the player's balance with a promo win.**

## Player authorization

The method authorizes the player in the Casino Wallet.

### Endpoint
```
POST /auth
```

### Request parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| secret | string | ✅ | 100hp gaming provides a secret key for partners in preparation for integration. | An authentication key. |
| sessionId | string | ✅ | 4affa487-c147-4e48-ae90-903223f17d5b | Casino-partner generated session. Must be <= 255 characters |

### Example of request

```bash
curl --location 'https://example.com/auth' \
--header 'Content-Type: application/json' \
--data '{
  "sessionId": "4affa487-c147-4e48-ae90-903223f17d5b",
  "secret": "<secret>"
}'
```

### Response parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| userId | string | ✅ | 238795249760076625 | Player's identifier. Must be <= 255 characters |
| currency | string (Format ISO 4217) | ✅ | USD | Player's currency. |
| balance | string | ✅ | 104.23 | Player's balance. |
| userName | string | ✅ | Player_1 | Player's name. Must be <= 255 characters |
| status | string | | active or blocked | Player's status. |
| type | string | | real or test | Player's type. |
| segment | array[string] | | vip | To which segment does the user belong, for example, "VIP" |

### Example of response

```json
{
  "userId": "238795249760076625",
  "currency": "USD",
  "balance": "184.23",
  "userName": "Player_1",
  "type": "real",
  "status": "active",
  "segment": ["vip"]
}
```

### Errors options

#### Validation error
```json
{
  "data": "string",
  "message": "Validation error",
  "errCode": 400
}
```

**Possible options for data:**
- secret must be not empty
- secret must be a string
- sessionId must be not empty
- sessionId must be <= 255 characters
- sessionId must be a string
- amount must be not empty
- amount must be a string
- amount must be greater than 0
- promoId must be not empty

#### Invalid secret key
```json
{
  "data": "",
  "message": "Invalid secret key",
  "errCode": 601
}
```

#### Insufficient balance
```json
{
  "data": "",
  "message": "Insufficient balance",
  "errCode": 603
}
```

#### Session not found
```json
{
  "data": "",
  "message": "Session not found",
  "errCode": 604
}
```

#### Transaction already exists
```json
{
  "data": "",
  "message": "Transaction already exists",
  "errCode": 607
}
```

#### Internal Server Error
```json
{
  "data": "",
  "message": "Internal Server Error",
  "errCode": 702
}
```

---

## Health Check

Retrieving server status. If Casino Wallet is up and running healthy - return 200 OK.

### Endpoint
```
GET /liveness
```

### Example of request

```bash
curl -X GET http://yourserver.com/liveness
```

### Example of response

```
200 OK
```

---

# Decimal Precision

If parameters `amount` and `balances` has more than 2 decimal places, the decimal part should be truncated to 2 decimal places.

For example, the `amount = 104.129` should be truncated to `104.12`.

---

# Transaction ID Consistency

The `txId` parameter must be the same for withdrawal-deposit transactions to ensure proper tracking and error handling.

## Handling duplicate transactions after rollback

In the event that a deposit with the same transaction ID as a successful rollback is received, an error response should be returned with the following details:

- **errCode**: 608
- **message**: "Transaction already refunded"

This error indicates that the transaction has already been refunded, and no further action is necessary.

## Handling duplicate rollbacks after deposit

In the event that a rollback with the same transaction ID as a successful deposit is received, an error response should be returned with the following details:

- **errCode**: 609
- **message**: "Transaction already settled"

This error indicates that the transaction has already been settled, and no further action is necessary.

---

# Errors format

> **Note:** The HTTP error code must match the errCode

## Error parameters

| Parameters | Type | Required | Description |
|------------|------|----------|-------------|
| data | string | ✅ | Additional info. Can be empty |
| message | string | ✅ | Error message. |
| errCode | int | ✅ | HTTP error code. |

## Error schema

```json
{
  "data": "string",
  "message": "string",
  "errCode": int
}
```

---

# API Changes

## 2.7.0 (Current version)

- The parameters `cashierUrl` and `exitUrl` have been removed from method `/games/v1/launch`

## 2.6.0

- The parameter `sessionId` in `/games/v1/launch` is optional now for demo mode.
- Removed `/getUsersMeta`

## 2.5.0

- Added field `category` to `/games/v1/list` method.

## 2.4.0

- Integration scheme was updated.
- Added the new method for promo win payout - `/bonusWin`
- Edited the duration (from 24 hours to 2 weeks) then a withdrawal must be rolled back - section with recommendation in `/withdrawal` method

## 2.3.0

- The transaction ID returned is now the same as the one sent for `/withdrawal`.
- Added validation errors for `txId` in the `/withdrawal` method.
- For all errors, the `data` parameter now returns an empty string `""` instead of `null`.
- In `/getUsersMeta`, invalid userIds are now ignored if valid ones are present in the request.
- Added clarifications about parameter `logoUrl` in `/games/v1/list`

## 2.2.0

- Updated the `/games/v1/list` method with `developer` and `supportedLanguage` parameters

## 2.1.0

- Added "Transaction already exists" error in `/withdrawal` method
- Added detailed Information for Parameters in `error` and `results` arrays in `/getUsersMeta` method

## 2.0.0

- Corrected a typo in the `segment` parameter in the `/auth` method
- Removed error `Session not found` from `/getUsersMeta`
- Added a section: Decimal Precision
- Updated the `/withdrawal` method
- Added the `/liveness` method

## 1.4.3

- Added Information about handling duplicate transactions after rollback and handling duplicate rollbacks after deposit
- Added clarifications about parameter `txId` to ensure that it is consistently used across withdrawal and deposit transactions for proper tracking and error handling.

## 1.4.2

- Corrected a typo in the `usersIds` parameter in the `/getUsersMeta` method

## 1.4.1

- Added recommendation about rolling back incomplete transactions in the withdrawal method

## 1.4.0

- Added http code 400 in all Casino Wallet API methods

## v 1.3.2

- Added formats and max values for `sessionId`, `userId`, `txId`, `geo`, `ip`, `browser`, `currency`.

## v 1.3.1

- Added info about matching HTTP error codes and errCode.
- Integration scheme was changed.

## v 1.3

- Field `geo` is removed from Casino Wallet API method `/auth`.
- Added field `geo` to `games/v1/launch` method.

## v 1.2

- In Casino Wallet API method `/auth`, the response has been modified: added optional field `segment`

## v 1.1

- In Casino Wallet API method `/auth`, the response has been modified.
- In the `games/v1/list` method, 404 error has been removed.
- The `games/v1/launch` method has been updated with two optional fields: `ip`, `browser`.
- In the `games/v1/launch` method, the parameter "sessionId" is now mandatory.
- Added changelog. characters
- sessionId must be a string

#### Invalid secret key
```json
{
  "data": "",
  "message": "Invalid secret key",
  "errCode": 601
}
```

#### Session not found
```json
{
  "data": "",
  "message": "Session not found",
  "errCode": 604
}
```

#### Internal Server Error
```json
{
  "data": "",
  "message": "Internal Server Error",
  "errCode": 702
}
```

---

## Withdrawal money

Debit the player balance according to in game activity.

> **Recommendation:** If, after 2 weeks from processing the withdrawal, there is no deposit/rollback in the Casino Wallet, then such a withdrawal must be rolled back in the Casino Wallet

### Endpoint
```
POST /withdrawal
```

### Request parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| secret | string | ✅ | 100hp gaming provides a secret key for partners in preparation for integration. | An authentication key. |
| sessionId | string | ✅ | 4affa487-c147-4e48-ae90-903223f17d5b | Casino-partner generated session. Must be <= 255 characters |
| txId | string (Format UUID v7) | ✅ | 017f22e6-79b0-7a33-98bb-1e2b1245aedd | Transaction identifier. |
| amount | string | ✅ | 100 | Bet amount. |

### Example of request

```bash
curl --location 'https://example.com/withdrawal' \
--header 'Content-Type: application/json' \
--data '{
  "sessionId": "4affa487-c147-4e48-ae90-903223f17d5b",
  "secret": "<secret>",
  "txId": "017f22e6-79b0-7a33-98bb-1e2b1245aedd",
  "amount": "100"
}'
```

### Response parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| userId | string | ✅ | 238795249760076625 | Player's identifier. Must be <= 255 characters |
| currency | string (Format ISO 4217) | ✅ | USD | Player's currency. |
| balance | string | ✅ | 104.23 | Player's balance. |
| txId | string (Format UUID v7) | ✅ | 017f22e6-79b0-7a33-98bb-1e2b1245aedd | Transaction identifier is the same as the one sent for withdrawal. |

### Example of response

```json
{
  "userId": "238798831595029329",
  "currency": "USD",
  "balance": "104.23",
  "txId": "017f22e6-79b0-7a33-98bb-1e2b1245aedd"
}
```

### Errors options

#### Validation error
```json
{
  "data": "string",
  "message": "Validation error",
  "errCode": 400
}
```

**Possible options for data:**
- secret must be not empty
- secret must be a string
- txId must be not empty
- txId must be a UUID
- sessionId must be not empty
- sessionId must be <= 255 characters
- sessionId must be a string
- amount must be not empty
- amount must be a string
- amount must be greater than 0

#### Invalid secret key
```json
{
  "data": "",
  "message": "Invalid secret key",
  "errCode": 601
}
```

#### Insufficient balance
```json
{
  "data": "",
  "message": "Insufficient balance",
  "errCode": 603
}
```

#### Session not found
```json
{
  "data": "",
  "message": "Session not found",
  "errCode": 604
}
```

#### Transaction already exists
```json
{
  "data": "",
  "message": "Transaction already exists",
  "errCode": 607
}
```

#### Internal Server Error
```json
{
  "data": "",
  "message": "Internal Server Error",
  "errCode": 702
}
```

---

## Deposit money

Credit the player balance according to in game activity.

### Endpoint
```
POST /deposit
```

### Request parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| secret | string | ✅ | 100hp gaming provides a secret key for partners in preparation for integration. | An authentication key. |
| txId | string (Format UUID v7) | ✅ | 017f22e6-79b0-7a33-98bb-1e2b1245aedd | Transaction identifier provided by the 100hp in withdrawal request. |
| amount | string | ✅ | 100 | Deposit amount. |

### Example of request

```bash
curl --location 'https://example.com/deposit' \
--header 'Content-Type: application/json' \
--data '{
  "txId": "017f22e6-79b0-7a33-98bb-1e2b1245aedd",
  "secret": "<secret>",
  "amount": "100"
}'
```

### Response parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| userId | string | ✅ | 238795249760076625 | Player's identifier. Must be <= 255 characters |
| currency | string (Format ISO 4217) | ✅ | USD | Player's currency. |
| balance | string | ✅ | 104.23 | Player's balance. |

### Example of response

```json
{
  "userId": "238798831595029329",
  "currency": "USD",
  "balance": "104.23"
}
```

### Errors options

#### Validation error
```json
{
  "data": "string",
  "message": "Validation error",
  "errCode": 400
}
```

**Possible options for data:**
- secret must be not empty
- secret must be a string
- txId must be not empty
- txId must be <= 50 characters
- txId must be a UUID
- amount must be not empty
- amount must be a string
- amount must be greater than or equal to 0

#### Invalid secret key
```json
{
  "data": "",
  "message": "Invalid secret key",
  "errCode": 601
}
```

#### Transaction not found
```json
{
  "data": "",
  "message": "Transaction not found",
  "errCode": 605
}
```

#### Transaction already refunded
```json
{
  "data": "",
  "message": "Transaction already refunded",
  "errCode": 608
}
```

#### Transaction already settled
```json
{
  "data": "",
  "message": "Transaction already settled",
  "errCode": 609
}
```

#### Internal Server Error
```json
{
  "data": "",
  "message": "Internal Server Error",
  "errCode": 702
}
```

---

## Rollback money

Money refund. The method that 100hp gaming calls if there is no response from Casino Wallet or an error response to the transaction request.

### Endpoint
```
POST /rollback
```

### Request parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| secret | string | ✅ | 100hp gaming provides a secret key for partners in preparation for integration. | An authentication key. |
| txId | string (Format UUID v7) | ✅ | 017f22e6-79b0-7a33-98bb-1e2b1245aedd | Transaction identifier to be rolled back. |

### Example of request

```bash
curl --location 'https://example.com/rollback' \
--header 'Content-Type: application/json' \
--data '{
  "txId": "017f22e6-79b0-7a33-98bb-1e2b1245aedd",
  "secret": "<secret>"
}'
```

### Response parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| userId | string | ✅ | 238795249760076625 | Player's identifier. Must be <= 255 characters |
| currency | string (Format ISO 4217) | ✅ | USD | Player's currency. |
| balance | string | ✅ | 104.23 | Player's balance. |

### Example of response

```json
{
  "userId": "238798831595029329",
  "currency": "USD",
  "balance": "104.23"
}
```

### Errors options

#### Validation error
```json
{
  "data": "string",
  "message": "Validation error",
  "errCode": 400
}
```

**Possible options for data:**
- secret must be not empty
- secret must be a string
- txId must be not empty
- txId must be <= 50 characters
- txId must be a UUID

#### Invalid secret key
```json
{
  "data": "",
  "message": "Invalid secret key",
  "errCode": 601
}
```

#### Transaction not found
```json
{
  "data": "",
  "message": "Transaction not found",
  "errCode": 605
}
```

#### Transaction already refunded
```json
{
  "data": "",
  "message": "Transaction already refunded",
  "errCode": 608
}
```

#### Transaction already settled
```json
{
  "data": "",
  "message": "Transaction already settled",
  "errCode": 609
}
```

#### Internal Server Error
```json
{
  "data": "",
  "message": "Internal Server Error",
  "errCode": 702
}
```

---

## Get balance

Getting player balance.

### Endpoint
```
POST /balance
```

### Request parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| secret | string | ✅ | 100hp gaming provides a secret key for partners in preparation for integration. | An authentication key. |
| sessionId | string | ✅ | 4affa487-c147-4e48-ae90-903223f17d5b | Casino-partner generated session. |

### Example of request

```bash
curl --location 'https://example.com/balance' \
--header 'Content-Type: application/json' \
--data '{
  "sessionId": "4affa487-c147-4e48-ae90-903223f17d5b",
  "secret": "<secret>"
}'
```

### Response parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| userId | string | ✅ | 238795249760076625 | Player's identifier. Must be <= 255 characters |
| currency | string (Format ISO 4217) | ✅ | USD | Player's currency. |
| balance | string | ✅ | 104.23 | Player's balance. |

### Example of response

```json
{
  "userId": "238798831595029329",
  "currency": "USD",
  "balance": "104.23"
}
```

### Errors options

#### Validation error
```json
{
  "data": "string",
  "message": "Validation error",
  "errCode": 400
}
```

**Possible options for data:**
- secret must be not empty
- secret must be a string
- sessionId must be not empty
- sessionId must be <= 255 characters
- sessionId must be a string

#### Invalid secret key
```json
{
  "data": "",
  "message": "Invalid secret key",
  "errCode": 601
}
```

#### Session not found
```json
{
  "data": "",
  "message": "Session not found",
  "errCode": 604
}
```

#### Internal Server Error
```json
{
  "data": "",
  "message": "Internal Server Error",
  "errCode": 702
}
```

---

## Promo Win Payout

Credit the player's balance with a promo win.

### Endpoint
```
POST /bonusWin
```

### Request parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| secret | string | ✅ | 100hp gaming provides a secret key for partners in preparation for integration. | An authentication key. |
| txId | string (Format UUID v7) | ✅ | 017f22e6-79b0-7a33-98bb-1e2b1245aedd | Transaction identifier. |
| sessionId | string | ✅ | 4affa487-c147-4e48-ae90-903223f17d5b | Casino-partner generated session. Must be <= 255 characters |
| amount | string | ✅ | 100 | Win mount. |
| promoId | string | ✅ | 38eebb80-d93b-4253-aef9-296dc256c3e5 | Promo campaign identifier. Informational field |

### Example of request

```bash
curl --location 'https://example.com/bonusWin' \
--header 'Content-Type: application/json' \
--data '{
  "secret": "b906pA0P2L1SJqwnJshoOVDGztzBu6o9NJ5PaCGaSbWv6xQrnzlf6wa7FIYm3UcLlszTIK24vN8x0uyCeRsQYg4OH7rr31N2RDPJOX3CVppsbJ3Q6xk1AR25YhGmueEn2ttoD1RZEo57d4SdtfwVjVPs8syAzS56CEuw8D9Mi1G4LndaQAwDs0xsmimyO1o0yxb1ODe36ong1CPiAayIjmbK4G4UDOk9JFO6UigxepvioKavXLX5phDKKAsDj7KTdjVs4oGXMdCkELh2SjyUohSLY4QuuVA4sJlQVGKABYyIYXeWsYw5dThKgi7RnpOC",
  "txId": "bf95016e-e549-49b0-b093-f67b10d34e38",
  "sessionId": "bdb6bc24-fad4-42eb-8d69-0ce2135d3200",
  "amount": "104.23",
  "promoId": "promo_1"
}'
```

### Response parameters

| Parameters | Type | Required | Example | Description |
|------------|------|----------|---------|-------------|
| userId | string | ✅ | 238795249760076625 | Player's identifier. Must be <= 255 characters |
| currency | string (Format ISO 4217) | ✅ | USD | Player's currency. |
| balance | string | ✅ | 104.23 | Player's balance. |

### Example of response

```json
{
  "userId": "238798831595029329",
  "currency": "USD",
  "balance": "104.23"
}
```

### Errors options

#### Validation error
```json
{
  "data": "string",
  "message": "Validation error",
  "errCode": 400
}
```

**Possible options for data:**
- secret must be not empty
- secret must be a string
- txId must be not empty
- txId must be a UUID
- sessionId must be not empty
- sessionId must be <= 255 