# conversations/v1/

## POST /:conversation\_id/messages/

Example request body:

```ruby
{
  content: "Hi! I'd like to buy your bike.",
  sender_id: "5678dcba"
}
```

# transactions/v1/

## POST /create

Example request body:

```ruby
{ transaction:
  { payment_process: :none # or :postpay, :preauthorize
  , payment_gateway: :paypal # or :checkout, :braintree
  , community_id: 501
  , starter_id: "5678dcba"
  , listing_id: 1234
  , listing_title: "Old city-bike"
  , listing_price: Money.new(50, "USD")
  , listing_author_id: "1234abcd"
  , listing_quantity: 1
  , automatic_confirmation_after_days: 7
  }

  # Paypal
, gateway_fields: {} # No additional fields for Paypal needed

  # Braintree
, gateway_fields: # Only for :preauthorize
  { cardholder_name: "Mikko Koski"
  , credit_card_number: "4000 5000 6000 7000 9"
  , cvv: "1234"
  , credit_card_expiration_month: 12
  , credit_card_expiration_year: 2019
  }
}
```

Response:

```ruby
{ transaction:
  { id: 1234
  , conversation_id: 3344,
  , payment_process: :none # or :postpay, :prepay
  , payment_gateway: :paypal # or :checkout, :braintree
  , community_id: 501
  , starter_id: "5678dcba"
  , listing_id: 1234
  , listing_title: "Old city-bike"
  , listing_price: Money.new(50, "USD")
  , listing_author_id: "1234abcd"
  , listing_quantity: 1
  , automatic_confirmation_after_days: 7
  , created_at: <Time>
  , updated_at: <Time>
  , last_transition_at: <Time>
  , current_state: :free       # or :initiated for Paypal
                               # or :preauthorized for preauthorized Braintree
                               # or :pending for postpay Braintree and Checkout
  , payment_total: Money.new(50, "USD") # only for Braintree if :preauthorized
  }

  # PayPal
, gateway_fields:
  { redirect_url: "https://paypal.com/token?EJAHGOSKLGAHSG"
  }

  # Braintree
, gateway_fields: {} # No additional fields for Braintree

}
```

## POST /:transaction_id/preauthorize

Only for **preauthorize** and **paypal**

```ruby
{
  gateway_fields:
  { payment_gateway: paypal
  , token: "ECJHGOAGIHADG"
  }
}
```

Response:

```ruby
{ transaction:
  { id: 1234
  , conversation_id: 3344,
  , payment_process: :preauthorize
  , payment_gateway: :paypal
  , community_id: 501
  , starter_id: "5678dcba"
  , listing_id: 1234
  , listing_title: "Old city-bike"
  , listing_price: Money.new(50, "USD")
  , listing_author_id: "1234abcd"
  , listing_quantity: 1
  , automatic_confirmation_after_days: 7
  , created_at: <Time>
  , updated_at: <Time>
  , last_transition_at: <Time>
  , current_state: :preauthorized
  , payment_total: Money.new(50, "USD")
  }
}
```

## POST /:transaction_id/reject

Request: Empty

Response:

```ruby
{ transaction:
  { id: 1234
  , conversation_id: 3344,
  , payment_process: :preauthorize
  , payment_gateway: :paypal
  , community_id: 501
  , starter_id: "5678dcba"
  , listing_id: 1234
  , listing_title: "Old city-bike"
  , listing_price: Money.new(50, "USD")
  , listing_author_id: "1234abcd"
  , listing_quantity: 1
  , automatic_confirmation_after_days: 7
  , created_at: <Time>
  , updated_at: <Time>
  , last_transition_at: <Time>
  , current_state: :rejected
  , payment_total: Money.new(50, "USD")
  }
}
```

## POST /:transaction\_id/complete\_preauthorization

Only for **preauthorize** and **paypal** and **braintree**

Request: Empty

Response:

```ruby
{ transaction:
  { id: 1234
  , conversation_id: 3344,
  , payment_process: :preauthorize
  , payment_gateway: :paypal
  , community_id: 501
  , starter_id: "5678dcba"
  , listing_id: 1234
  , listing_title: "Old city-bike"
  , listing_price: Money.new(50, "USD")
  , listing_author_id: "1234abcd"
  , listing_quantity: 1
  , automatic_confirmation_after_days: 7
  , created_at: <Time>
  , updated_at: <Time>
  , last_transition_at: <Time>
  , current_state: :rejected
  , payment_total: Money.new(50, "USD")
  }

  # PayPal
, gateway_fields:
  { pending_reason: :multicurrency
  }

  # Braintree
, gateway_fields: {} # No additional fields
}
```

## POST /:transaction_id/invoice

Only for **postpay** and **checkout** and **braintree**

Request:

```ruby
{
  # Checkout
  gateway_fields:
  { payment_rows: [] # Some payment row stuff here
  }

  # Braintree
, gateway_fields
  { total_sum: Money.new(5000, "EUR")
  }
}
```

Response:

```ruby
{ transaction:
  { id: 1234
  , conversation_id: 3344,
  , payment_process: :postpay
  , payment_gateway: :braintree
  , community_id: 501
  , starter_id: "5678dcba"
  , listing_id: 1234
  , listing_title: "Old city-bike"
  , listing_price: Money.new(50, "USD")
  , listing_author_id: "1234abcd"
  , listing_quantity: 1
  , automatic_confirmation_after_days: 7
  , created_at: <Time>
  , updated_at: <Time>
  , last_transition_at: <Time>
  , current_state: :accepted
  , payment_total: Money.new(50, "USD")
  }
}
```

## POST /:transaction\_id/pay\_invoice

Only for **postpay** and **checkout** and **braintree**

Request:

```ruby
{
  # Checkout
  gateway_fields:
  { ??? }

, gateway_fields
  { cardholder_name: "Mikko Koski"
  , credit_card_number: "4000 5000 6000 7000 9"
  , cvv: "1234"
  , credit_card_expiration_month: 12
  , credit_card_expiration_year: 2019
  }
}
```

Response:

```
{ transaction:
  { id: 1234
  , conversation_id: 3344,
  , payment_process: :postpay
  , payment_gateway: :braintree
  , community_id: 501
  , starter_id: "5678dcba"
  , listing_id: 1234
  , listing_title: "Old city-bike"
  , listing_price: Money.new(50, "USD")
  , listing_author_id: "1234abcd"
  , listing_quantity: 1
  , automatic_confirmation_after_days: 7
  , created_at: <Time>
  , updated_at: <Time>
  , last_transition_at: <Time>
  , current_state: :paid
  , payment_total: Money.new(50, "USD")
  }
}
```

## POST /:transaction_id/complete

Request: Empty

Response:

```ruby
{ transaction:
  { id: 1234
  , conversation_id: 3344,
  , payment_process: :preauthorize
  , payment_gateway: :paypal
  , community_id: 501
  , starter_id: "5678dcba"
  , listing_id: 1234
  , listing_title: "Old city-bike"
  , listing_price: Money.new(50, "USD")
  , listing_author_id: "1234abcd"
  , listing_quantity: 1
  , automatic_confirmation_after_days: 7
  , created_at: <Time>
  , updated_at: <Time>
  , last_transition_at: <Time>
  , current_state: :completed
  , payment_total: Money.new(50, "USD")
  }
}
```

## POST /:transaction_id/cancel

Request: Empty

Response:

```ruby
{ transaction:
  { id: 1234
  , conversation_id: 3344,
  , payment_process: :preauthorize
  , payment_gateway: :paypal
  , community_id: 501
  , starter_id: "5678dcba"
  , listing_id: 1234
  , listing_title: "Old city-bike"
  , listing_price: Money.new(50, "USD")
  , listing_author_id: "1234abcd"
  , listing_quantity: 1
  , automatic_confirmation_after_days: 7
  , created_at: <Time>
  , updated_at: <Time>
  , last_transition_at: <Time>
  , current_state: :canceled
  , payment_total: Money.new(50, "USD")
  }
}
```
