# pinelabs-go

[![Go Reference](https://pkg.go.dev/badge/github.com/plural-pinelabs/pinelabs-go.svg)](https://pkg.go.dev/github.com/plural-pinelabs/pinelabs-go)
[![license](https://img.shields.io/github/license/plural-pinelabs/pinelabs-go)](#license)

Official Go SDK for the **Pine Labs Online Payment Gateway** (Plural).

## Install

```bash
go get github.com/plural-pinelabs/pinelabs-go
```

Requires **Go ≥ 1.13**.

## Quickstart

The Pine Labs API uses **OAuth2 client_credentials**. Exchange your `client_id` / `client_secret` for an access token, then pass it to the client.

```go
package main

import (
	"context"
	"fmt"
	"log"

	pinelabs "github.com/plural-pinelabs/pinelabs-go"
	pinelabsclient "github.com/plural-pinelabs/pinelabs-go/client"
	"github.com/plural-pinelabs/pinelabs-go/option"
)

func main() {
	ctx := context.Background()

	// 1. Get a token (the /token endpoint does not require authentication)
	auth := pinelabsclient.NewClient(
		option.WithBaseURL("https://pluraluat.v2.pinepg.in"),
	)

	tokenResp, err := auth.Authentication.GenerateToken(ctx, &pinelabs.GenerateTokenRequest{
		GrantType:    "client_credentials",
		ClientId:     "<client-id>",
		ClientSecret: "<client-secret>",
	})
	if err != nil {
		log.Fatal(err)
	}

	// 2. Build an authenticated client
	client := pinelabsclient.NewClient(
		option.WithBaseURL("https://pluraluat.v2.pinepg.in"),
		option.WithToken(*tokenResp.AccessToken),
	)

	// 3. Call any operation
	amount := int64(50000)
	currency := "INR"
	order, err := client.Orders.CreateOrder(ctx, &pinelabs.CreateOrderRequest{
		MerchantOrderReference: pinelabs.String("order-001"),
		OrderAmount: &pinelabs.Amount{
			Value:    &amount,    // ₹500.00 (in paisa)
			Currency: &currency,
		},
	})
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Order created: %+v\n", order)
}
```

## Environments

| Environment | Base URL                            |
| ----------- | ----------------------------------- |
| UAT         | `https://pluraluat.v2.pinepg.in`    |
| Production  | `https://api.pluralpay.in`          |

Pass the URL via `option.WithBaseURL(...)`.

## Configuration

All configuration is done via request options passed to `pinelabsclient.NewClient(...)`:

```go
client := pinelabsclient.NewClient(
	option.WithBaseURL("https://api.pluralpay.in"),
	option.WithToken("<access-token>"),
)
```

### Available options

| Option                        | Description                                    |
| ----------------------------- | ---------------------------------------------- |
| `option.WithBaseURL(url)`     | Override the default API base URL               |
| `option.WithToken(token)`     | Set the `Authorization: Bearer <token>` header  |
| `option.WithHTTPClient(c)`    | Use a custom `http.Client`                      |
| `option.WithHTTPHeader(h)`    | Add custom HTTP headers to every request        |
| `option.WithMaxAttempts(n)`   | Set the maximum number of retry attempts        |

### Per-request options

Every API method accepts request options as trailing arguments to override client-level defaults:

```go
order, err := client.Orders.CreateOrder(
	ctx,
	&pinelabs.CreateOrderRequest{...},
	option.WithToken("<different-token>"),
)
```

## Available resources

| Resource                       | Access via                          |
| ------------------------------ | ----------------------------------- |
| Authentication                 | `client.Authentication`             |
| Orders                         | `client.Orders`                     |
| Refunds                        | `client.Refunds`                    |
| Settlements                    | `client.Settlements`                |
| Split Settlements              | `client.SplitSettlements`           |
| Affordability Suite            | `client.AffordabilitySuite`         |
| Card Payments                  | `client.CardPayments`              |
| Pay By Points                  | `client.PayByPoints`               |
| E-Challans                     | `client.EChallans`                 |
| Apple Pay                      | `client.ApplePay`                  |
| International Payments         | `client.InternationalPayments`     |
| BNPL                           | `client.Bnpl`                      |
| Convenience Fee                | `client.ConvenienceFee`            |
| Checkout                       | `client.Checkout`                  |
| Payment Links                  | `client.PaymentLinks`              |
| Customers                      | `client.Customers`                 |
| Tokenization                   | `client.Tokenization`              |
| Payouts                        | `client.Payouts`                   |
| Subscriptions — Plans          | `client.SubscriptionsPlans`        |
| Subscriptions — Subscriptions  | `client.SubscriptionsSubscriptions`|
| Subscriptions — Presentations  | `client.SubscriptionsPresentations`|

## Error handling

API errors are returned as Go errors. Use type assertion to access the status code and response body:

```go
order, err := client.Orders.GetOrder(ctx, &pinelabs.GetOrderRequest{
	OrderId: "ord_invalid",
})
if err != nil {
	// err contains the HTTP status code and error body
	fmt.Println(err.Error())
}
```

## Documentation

- [Pine Labs API Reference](https://docs.pluralonline.com)
- [Go SDK Reference (pkg.go.dev)](https://pkg.go.dev/github.com/plural-pinelabs/pinelabs-go)

## License

MIT — see [LICENSE](LICENSE) for details.
