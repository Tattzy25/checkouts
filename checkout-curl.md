# Checkout Curl Calls

## Settings

- MCP endpoint: `https://checkouts.anigok.com/mcp`
- Method: `POST`
- Header: `Content-Type: application/json`
- Header: `Accept: application/json, text/event-stream`
- Transport requirement: the current MCP handler requires both `application/json` and `text/event-stream` in the `Accept` header

## hello

```bash
curl -X POST https://checkouts.anigok.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "hello",
      "arguments": {
        "name": "World"
      }
    }
  }'
```

## create_checkout

```bash
curl -X POST https://checkouts.anigok.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "create_checkout",
      "arguments": {
        "shop_domain": "your-shop-domain.myshopify.com",
        "meta": {
          "ucp-agent": {
            "profile": "https://example.com/.well-known/ucp"
          }
        },
        "checkout": {
          "currency": "USD",
          "line_items": [
            {
              "quantity": 2,
              "item": {
                "id": "gid://shopify/ProductVariant/12345678901"
              }
            }
          ],
          "buyer": {
            "email": "buyer@example.com"
          },
          "context": {
            "intent": "Buyer wants two sweaters shipped to Brooklyn",
            "language": "en-US",
            "currency": "USD",
            "eligibility": [
              "com.example.loyalty_gold"
            ]
          },
          "attribution": {
            "referring_domain": "example-agent.com",
            "click_id_tag": "gclid",
            "click_id_value": "abc123xyz",
            "activity_id_tag": "activity_id",
            "activity_id_value": "checkout-start-001",
            "utm_campaign": "spring_sale",
            "utm_source": "example_agent",
            "utm_medium": "agentic_commerce",
            "utm_content": "sweater_recommendation",
            "utm_term": "organic cotton sweater"
          }
        }
      }
    }
  }'
```

## create_checkout from cart

```bash
curl -X POST https://checkouts.anigok.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "create_checkout",
      "arguments": {
        "shop_domain": "your-shop-domain.myshopify.com",
        "meta": {
          "ucp-agent": {
            "profile": "https://example.com/.well-known/ucp"
          }
        },
        "cart_id": "gid://shopify/Cart/cart_abc123"
      }
    }
  }'
```

## get_checkout

```bash
curl -X POST https://checkouts.anigok.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "get_checkout",
      "arguments": {
        "shop_domain": "your-shop-domain.myshopify.com",
        "meta": {
          "ucp-agent": {
            "profile": "https://example.com/.well-known/ucp"
          }
        },
        "id": "gid://shopify/Checkout/abc123?key=xyz789"
      }
    }
  }'
```

## update_checkout

```bash
curl -X POST https://checkouts.anigok.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "update_checkout",
      "arguments": {
        "shop_domain": "your-shop-domain.myshopify.com",
        "meta": {
          "ucp-agent": {
            "profile": "https://example.com/.well-known/ucp"
          }
        },
        "id": "gid://shopify/Checkout/abc123?key=xyz789",
        "checkout": {
          "line_items": [
            {
              "id": "gid://shopify/CartLine/a891d8e3-7dad-42fe-b795-d8ad77ce2af8?cart=hWNF4zCrAqgZcs3OYKm4Vgji",
              "quantity": 1,
              "item": {
                "id": "gid://shopify/ProductVariant/12345678901"
              }
            }
          ],
          "buyer": {
            "email": "buyer@example.com"
          },
          "context": {
            "address_country": "US",
            "address_region": "NY",
            "postal_code": "10001",
            "intent": "Buyer updated the shipping destination",
            "language": "en-US",
            "currency": "USD"
          },
          "attribution": {
            "utm_source": "example_agent"
          }
        }
      }
    }
  }'
```

## complete_checkout

```bash
curl -X POST https://checkouts.anigok.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "complete_checkout",
      "arguments": {
        "shop_domain": "your-shop-domain.myshopify.com",
        "meta": {
          "ucp-agent": {
            "profile": "https://example.com/.well-known/ucp"
          },
          "idempotency-key": "661e9500-f39c-52e5-b827-557766551111"
        },
        "id": "gid://shopify/Checkout/abc123?key=xyz789",
        "checkout": {
          "payment": {
            "instruments": [
              {
                "id": "pm_1234567890abc",
                "handler_id": "gpay_7k2m",
                "type": "card"
              }
            ]
          }
        }
      }
    }
  }'
```

## Notes

- The caller provides `shop_domain`.
- The Worker maps `shop_domain` to `https://{shop-domain}/api/ucp/mcp`.
- The caller provides `meta["ucp-agent"].profile`.
- `complete_checkout` requires `meta["idempotency-key"]` as a UUID.
- `create_checkout` can use `cart_id` instead of `checkout`.
- `update_checkout` sends the full checkout state. Omitted fields are removed.
- Calls to this endpoint must include `Accept: application/json, text/event-stream`.
