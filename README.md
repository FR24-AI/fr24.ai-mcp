# fr24.ai-mcp

[中文版本](README.zh-CN.md)

Source: [FR24 International Flights MCP Integration Guide](https://fr24.ai/mcp.html)

---

## ModelScope MCP Service Configuration

Use the following service configuration when importing this GitHub repository into the ModelScope MCP marketplace. `X_CID` and `X_PASSKEY` are user-side environment variables for the FR24 buyer ID and 16-character passkey.

```json
{
  "mcpServers": {
    "fr-flight-mcp": {
      "type": "http",
      "url": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "${X_CID}",
        "X-Passkey": "${X_PASSKEY}"
      }
    }
  }
}
```

## About Flightroutes24

FR24 AI connects Flightroutes24's global flight distribution resources with AI Agent workflows, providing end-to-end capabilities for international flight search, price verification, booking, and order lookup across clients such as Cursor, Claude Code, and Codex. Founded in 2014, Shenzhen Flightroutes24 relies on its self-developed JET-X transaction system to connect multiple GDSs, airline direct connections, NDC channels, and low-cost carrier resources, serving travel agencies, OTAs, and travel procurement buyers worldwide. Through MCP, Agents can invoke real-time fares and inventory through the standard workflow of searching flights, locking prices, submitting bookings, and querying orders, upgrading global flight procurement from manual comparison and system switching into an intelligent distribution chain that AI can orchestrate and automate.

---

FR24 AI MCP Server delivers the complete workflow for international flight search, price verification, booking, and order lookup. It supports MCP-compatible AI Agent clients such as Cursor, Claude Code, Codex, WorkBuddy, and OpenClaw.

### 1. Overview

The standard booking flow must be called in order. Steps cannot be skipped:

```text
shopping search flights -> pricing lock price -> booking submit booking -> orderDetail query order
```

### 2. Get CID and Passkey

Integration requires an FR24-assigned Buyer ID (`X-Cid`) and a 16-character buyer secret (`X-Passkey`).

Application process:

1. Business contact: contact the FR24 business team with your company name, use case, and estimated request volume.
2. Qualification review: submit company credentials. FR24 completes the review within 1 business day.
3. Credential issuance: after approval, your account manager sends `X-Cid` and the 16-character `X-Passkey`.
4. Integration testing: configure the MCP endpoint using the examples below. Technical support can help with joint testing.

Contact information:

| Type | Information |
| --- | --- |
| Business email | bd@flightroutes24.com |
| B2B platform | https://b2b.flightroutes24.com |
| Technical support | tech@flightroutes24.com |

> Security: `X-Passkey` is a core credential. Never commit it to public repositories or share it with third parties. Contact support immediately if it may be compromised.

### 3. Configuration

#### 3.1 Endpoint

| Environment | MCP Endpoint |
| --- | --- |
| Production | `https://mcp.fr24.ai/mcp` |

#### 3.2 Authentication

Pass credentials through HTTP headers:

| Header | Description | Example |
| --- | --- | --- |
| `X-Cid` | Buyer ID assigned by FR24 | `FRG` |
| `X-Passkey` | Buyer secret; must be exactly 16 characters | `your16charpasskey` |

`X-Passkey` must be exactly 16 bytes. Incorrect length returns an authentication failure.

#### 3.3 Client Configuration

All clients use the same authentication headers: `X-Cid` for buyer ID and `X-Passkey` for the 16-character secret. Replace `YOUR_CID` and `YOUR_16CHAR_PASSKEY` with credentials assigned by FR24.

Cursor (`.cursor/mcp.json` in project root):

```json
{
  "mcpServers": {
    "fr-flight-mcp": {
      "url": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "YOUR_CID",
        "X-Passkey": "YOUR_16CHAR_PASSKEY"
      }
    }
  }
}
```

Claude Code (project-level `.mcp.json` or user-level `~/.mcp.json`):

```json
{
  "mcpServers": {
    "fr-flight-mcp": {
      "type": "http",
      "url": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "YOUR_CID",
        "X-Passkey": "YOUR_16CHAR_PASSKEY"
      }
    }
  }
}
```

CLI alternative:

```bash
claude mcp add --transport http fr-flight-mcp https://mcp.fr24.ai/mcp \
  -H "X-Cid:YOUR_CID" \
  -H "X-Passkey:YOUR_16CHAR_PASSKEY"
```

Codex (`~/.codex/config.toml`):

```toml
[mcp_servers.fr-flight-mcp]
transport = "http"
url = "https://mcp.fr24.ai/mcp"

[mcp_servers.fr-flight-mcp.headers]
X-Cid = "YOUR_CID"
X-Passkey = "YOUR_16CHAR_PASSKEY"
```

WorkBuddy (add an HTTP MCP Server in settings, or edit `workbuddy.mcp.json`):

```json
{
  "servers": {
    "fr-flight-mcp": {
      "transport": "http",
      "endpoint": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "YOUR_CID",
        "X-Passkey": "YOUR_16CHAR_PASSKEY"
      }
    }
  }
}
```

OpenClaw (`openclaw.config.json` or `~/.openclaw/mcp_servers.json`):

```json
{
  "mcpServers": {
    "fr-flight-mcp": {
      "transport": "streamable-http",
      "url": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "YOUR_CID",
        "X-Passkey": "YOUR_16CHAR_PASSKEY"
      }
    }
  }
}
```

### 4. Tools Reference

#### shopping

Search international flights. Supports 6 search modes and returns up to 10 flight results.

Required parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `origin` | string | Origin IATA code, such as `PEK` or `HKG` |
| `destination` | string | Destination IATA code, such as `BKK` or `NRT` |
| `depDate` | string | Departure date, format `yyyy-MM-dd` |

Optional parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `adultNum` | integer | Number of adults, default 1 |
| `childNum` | integer | Number of children aged 2-12, default 0 |
| `infantNum` | integer | Number of infants under 2, default 0 |
| `retDate` | string | Return date; omit for one-way trips |
| `searchMode` | string | Search mode, see table below |
| `departureTimeRange` | string | `TIME` mode parameter, e.g. `06:00-12:00` |
| `priceRange` | string | `PRICE` mode parameter, e.g. `500-2000` |
| `flightNo` | string | `FLIGHT` mode parameter, e.g. `CA1234` |
| `cabin` | string | `CABIN` mode parameter: `Y`=economy, `C`=business, `F`=first, `P`=premium economy |

Search modes:

| Value | Description | Companion Parameter |
| --- | --- | --- |
| Omit | Default search, sorted by price | - |
| `TIME` | Filter by departure time window | `departureTimeRange` |
| `PRICE` | Filter by total price range | `priceRange` |
| `FLIGHT` | Search by flight number | `flightNo` |
| `TRANSFER` | Connecting flights only | - |
| `CABIN` | Filter by cabin class | `cabin` |

Request examples:

```json
{ "origin": "HKG", "destination": "BKK", "depDate": "2026-08-01" }
```

```json
{
  "origin": "PEK",
  "destination": "NRT",
  "depDate": "2026-08-10",
  "searchMode": "TIME",
  "departureTimeRange": "06:00-12:00"
}
```

```json
{
  "origin": "SZX",
  "destination": "BKK",
  "depDate": "2026-08-15",
  "retDate": "2026-08-20"
}
```

```json
{
  "origin": "HKG",
  "destination": "SIN",
  "depDate": "2026-08-01",
  "adultNum": 1,
  "childNum": 1
}
```

Response fields:

| Field | Description |
| --- | --- |
| `flightNo` | Flight number |
| `airline` | Airline name |
| `cabin` | Cabin class: economy, business, or first |
| `depAirport` | Departure airport, format `CODE(City)` |
| `depTime` | Departure time, format `MM-dd HH:mm` |
| `arrAirport` | Arrival airport |
| `arrTime` | Arrival time |
| `duration` | Flight duration, e.g. `3h45m` |
| `type` | Direct or connecting |
| `priceDisplay` | Total price with currency, e.g. `1200 CNY` |
| `remainingSeats` | Remaining seats |
| `seatsAlert` | Low-seat alert when 3 or fewer seats remain |
| `refundPolicy` | Refund and change policy |

#### pricing

Lock the selected fare for 30 minutes. Must be called after `shopping` and before `booking`. When traveling with children or infants, `adultNum`, `childNum`, and `infantNum` must match the `shopping` request.

Required parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `offerId` | string | `offerId` from the `shopping` response |

Optional parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `adultNum` | integer | Number of adults, default 1; must match search |
| `childNum` | integer | Number of children, default 0 |
| `infantNum` | integer | Number of infants, default 0 |

Response fields:

| Field | Location | Description |
| --- | --- | --- |
| `priceDisplay` | top level | Locked total price with currency |
| `priceBreakdown` | top level | Price breakdown for multiple passengers |
| `remainingSeats` | top level | Remaining seats |
| `msg` | top level | Message |
| `meta.verifyOfferId` | `meta` | `verifyOfferId` for `booking` |
| `meta.isInternational` | `meta` | Whether the route is international |
| `meta.requiredPassengerFields` | `meta` | Required passenger fields; `cardType.allowedValues` lists allowed document types |

`requiredPassengerFields` example:

```json
{
  "birthday": true,
  "gender": true,
  "cardNum": true,
  "cardType": {
    "required": true,
    "allowedValues": ["PP", "GA", "HX", "TB"]
  },
  "cardIssuedPlace": true,
  "cardExpiryDate": true,
  "nationality": true,
  "paxEmail": false,
  "paxMobile": false
}
```

`allowedValues` excludes `ID` on international routes and may include `ID` on domestic routes. The `cardType` used for booking must be selected from this list.

#### booking

Submit a flight booking. Supports multiple passengers including adults, children, and infants.

Required parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `verifyOfferId` | string | `meta.verifyOfferId` from `pricing` |
| `passengersJson` | string | Passenger list as a JSON array string |

Optional parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `allowedCardTypes` | array | `cardType.allowedValues` from `pricing`; strongly recommended |
| `departureDate` | string | Departure date for age validation; may be auto-filled from cache |
| `contactName` | string | Contact name; defaults to the first passenger |
| `contactMobile` | string | Contact mobile number |
| `contactEmail` | string | Contact email |
| `idempotencyKey` | string | UUID idempotency key; the same key creates only one order within 24 hours |
| `partnerOrderNo` | string | Partner custom order number |

`passengersJson` passenger fields:

| Field | Required | Description |
| --- | --- | --- |
| `name` | Required | Surname/given name in uppercase, e.g. `ZHANG/SAN` |
| `paxType` | Required | `ADT`=adult, `CHD`=child, `INF`=infant |
| `gender` | Required | `M`=male, `F`=female |
| `birthday` | Required | Date of birth, format `yyyy-MM-dd` |
| `cardType` | Required | Document type; must be in `allowedValues` |
| `cardNum` | Required | Document number, max 20 characters |
| `cardIssuedPlace` | Conditional | Document issuing country, ISO 2-letter code, e.g. `CN` |
| `cardExpiryDate` | Conditional | Document expiry date, format `yyyy-MM-dd` |
| `nationality` | Conditional | Nationality, ISO 2-letter code, e.g. `CN` |
| `areaCode` | Optional | Mobile area code, default `86` |
| `paxMobile` | Optional | Contact phone |
| `paxEmail` | Optional | Contact email |
| `accompaniedPaxId` | Required for CHD/INF | Linked adult index starting from 1, e.g. `"1"` |

Document types:

| Code | Description | Applicable Routes |
| --- | --- | --- |
| `ID` | National ID | Domestic routes only |
| `PP` | Passport | International and domestic |
| `GA` | HK/Macau travel permit | International and domestic |
| `HX` | Home Return Permit | International and domestic |
| `TB` | Taiwan Compatriot Permit | International and domestic |

Age rules:

| `paxType` | Age at Departure | Seat |
| --- | --- | --- |
| `INF` | Under 2 years | No seat |
| `CHD` | 2 years inclusive to under 12 years | Seat |
| `ADT` | 12 years and above | Seat |

National ID (`ID`) is not allowed on international routes. Each adult may accompany up to 2 children and 1 infant.

#### orderDetail

Query details of a submitted flight order.

Required parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `orderNo` | string | Order number returned by `booking` |

Response fields:

| Field | Description |
| --- | --- |
| `orderNo` | Order number |
| `orderStatus` | Order status |
| `priceDisplay` | Total price |
| `createTime` | Booking time |
| `segments` | Flight segments |
| `passengers` | Passenger information |
| `ticketNos` | Ticket numbers, available after ticketing |

### 5. Full Example

Complete four-step flow for one adult, HKG to BKK:

1. Search flights

```json
{ "origin": "HKG", "destination": "BKK", "depDate": "2026-08-01" }
```

Pick an `offerId` from the `flights` list for the next step.

2. Verify and lock

```json
{ "offerId": "21725576870629376120", "adultNum": 1 }
```

Take `verifyOfferId` and `requiredPassengerFields.cardType.allowedValues` from `meta`.

3. Submit booking

```json
{
  "verifyOfferId": "2172558470381977600",
  "passengersJson": "[{\"name\":\"ZHANG/SAN\",\"paxType\":\"ADT\",\"gender\":\"M\",\"birthday\":\"1990-01-01\",\"cardType\":\"PP\",\"cardNum\":\"E12345678\",\"cardIssuedPlace\":\"CN\",\"cardExpiryDate\":\"2030-12-31\",\"nationality\":\"CN\"}]",
  "allowedCardTypes": ["PP", "GA", "HX", "TB"]
}
```

The response returns `orderNo` and `payUrl` for payment.

4. Query order

```json
{ "orderNo": "21725493286277120" }
```

### 6. Common Error Codes

| Error Code | Description | Resolution |
| --- | --- | --- |
| `AUTH_FAILED` | Authentication failed | Check `X-Cid` and `X-Passkey`; passkey must be exactly 16 characters |
| `PARAM_INVALID` | Invalid parameters | See `msg` for details such as name format, document type, or age mismatch |
| `10701298` | `offerId` invalid after previous verification failure | Run `shopping` again for a new `offerId` |
| `20901997` | Price verification failed | Passenger count mismatch or the flight does not support child fares |
| `OFFER_ID_EXPIRED` | `offerId` expired after more than 30 minutes | Search again for a new `offerId` |
| `PASSENGER_NUM_DIFF` | Passenger count differs from the verification step | Ensure the passenger count in `booking` matches `pricing` |
| `ACCOMPANIED_PAX_ID` | Invalid `accompaniedPaxId` | `accompaniedPaxId` for CHD/INF must reference an ADT in the list |

### 7. Important Notes

1. Do not skip steps: call `shopping -> pricing -> booking` in order; do not call `booking` directly.
2. Price lock: `pricing` remains valid for 30 minutes. Re-verify after timeout.
3. Child fares: pass `childNum` at search time, and use the same passenger counts during verification.
4. International route documents: international routes do not accept national ID (`ID`); use passport (`PP`) or other international documents.
5. Infant rules: `INF` passengers do not occupy seats. Each adult may accompany at most 1 infant and 2 children.
6. Idempotent booking: pass `idempotencyKey` in UUID format on unstable networks to prevent duplicate bookings.
7. Payment: after booking succeeds, complete payment through the `payUrl` on the B2B order page.
