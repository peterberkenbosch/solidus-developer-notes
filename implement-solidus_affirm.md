# Implement solidus_affirm

These notes are from creating the Solidus extension for Affirm, it's an
offside PSP and therefore a bit more challenging then the pure API credit
card PSP's.

References:
* [Affirm](https://www.affirm.com)
* [Affirm Direct API Docs](https://docs.affirm.com/Integrate_Affirm/Direct_API)
* [Solidus Affirm Extension](https://github.com/StemboltHQ/solidus_affirm)

## PaymentMethod

The most easy option for adding a new PaymentMethod is by simply inherit from
`Spree::PaymentMethod`, see [Github source](https://github.com/solidusio/solidus/blob/master/core/app/models/spree/payment_method.rb)

So in this case we start with creating the `Spree::PaymentMethod::Affirm` class
that will inherit from the aforementioned `PaymentMethod`.
We also need to configure the `public_api_key` and the `private_api_key`, since
we need those values while communicating with Affirm.

```ruby
module Spree
  class PaymentMethod::Affirm < PaymentMethod
    preference :public_api_key, :string
    preference :private_api_key, :string
  end
end
```

Next we need to make sure that this `PaymentMethod` is registered in the
<!-- available payment method for the Solidus instance. So in your `lib/solidus_affirm/engine.rb` -->
add this:

```ruby
initializer "spree.gateway.payment_methods", after: "spree.register.payment_methods" do |app|
  app.config.spree.payment_methods << Spree::PaymentMethod::Affirm
end
```

At this point this PaymentMethod can be used.

## Authorize the charge on Affirm.

When a successful payment has been made to Affirm (more about that later), we will receive a `POST` request to the specified confirm URL. The body will contain the `checkout_token`. With this token we will authorize the charge on Affirm and store the `charge_id` that we will get from Affirm after the authorize call.

Also see [Authorize a charge](https://docs.affirm.com/Integrate_Affirm/Direct_API#authorize a_charge).

To make the API call to Affirm I used the [affirm-ruby](https://rubygems.org/gems/affirm-ruby) gem. This is as easy as calling:

```ruby
Affirm::Charge.authorize(checkout_token)
```
