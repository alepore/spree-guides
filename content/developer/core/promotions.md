---
title: "Promotions"
section: core
---

## Overview

Promotions within Spree are used to provide discounts to orders, as well as to add potential additional items at no extra cost. Promotions are one of the most
complex areas within Spree, as there are a large number of moving parts to consider.

Promotions can be activated in three different ways:

* When a user adds a product to their cart
* When a user enters a coupon code during the checkout process
* When a user visits a page within the Spree store

Promotions for these individual ways are activated through their corresponding `PromotionHandler` class, once they've been checked for eligibility.

Promotions relate to two other main components: `actions` and `rules`. When a promotion is activated, the actions for the promotion are performed, passing in the payload from the `fire_event` call that triggered the activator becoming active. Rules are used to determine if a promotion meets certain criteria in order to be applicable.

In some special cases where a promotion has a `code` or a `path` configured for it, the promotion will only be activated if the payload's code or path match the promotion's. The `code` attribute is used for promotion codes, where a user must enter a code to receive the promotion, and the `path` attribute is used to apply a promotion once a user has visited a specific path.

!!!
Path-based promotions will only work when the `Spree::PromotionHandler::Page` class is used, as in `Spree::ContentController` from `spree_frontend`.
!!!

A promotion may also have a `usage_limit` attribute set, which restricts how many times the promotion can be used.

## Actions

There are four actions that come with spree:

* An order-level adjustment
* An item-level adjustment
* Create line items
* Free shipping

### Creating an Adjustment

When a `CreateAdjustment` action is undertaken, an adjustment is automatically applied to the order, unless the promotion has already applied an adjustment to the order.

Once the adjustment has been applied to the order, its eligibility is re-checked every time the order is saved, by way of the `Adjustment#determine_eligibility` method. This calls the `Promotion#eligible?` method, which uses `Promotion#rules_are_eligible?` to determine if the promotion is still eligible based on its rules. For how this process works, please see the [rules section](#rules) below.

An adjustment to an order from a promotion depends on the calculators. For more information about calculators, please see the [Calculators guide](calculators).

### Creating an item adjustment

When a `CreateItemAdjustments` action is undertaken, an adjustment is automatically applied to each item within the order, unless the action has already been performed on that line item

The eligibility of the item for this promotion is re-checked whenever the item is updated. Its eligibility is based off the rules of the promotion.

An adjustment to an order from a promotion depends on the calculators. For more information about calculators, please see the [Calculators guide](calculators).

### Free Shipping

When a `FreeShipping` action is undertaken, all shipments within the order have their prices negated. Just like with prior actions, the eligibility of this promotion is checked again whenever a shipment changes.

### Adding a Line Item

When a `CreateLineItem` action is undertaken, a series of line items are automatically added to the order, which may alter the order's price. The promotion with an action to add a line item can also have another action to add an adjustment to the order to nullify the cost of adding the product to the order.


### Registering a New Action

You can create a new action for Spree's promotion engine by inheriting from `Spree::PromotionAction`, like this:

```ruby
class MyPromotionAction < Spree::PromotionAction
  def perform(options={})
    ...
  end
end
```

***
You can access promotion information using the `promotion` method within any `Spree::PromotionAction`.
***

This action must then be registered with Spree, which can be done by adding this code to `config/initializers/spree.rb`:

```ruby
Rails.application.config.spree.promotions.actions << MyPromotionAction
```

Once this has been registered, it will be available within Spree's interface. To provide translations for the interface, you will need to define them within your locale file. For instance, to define English translations for your new promotion action, use this code within `config/locales/en.yml`:

```yaml
en:
  spree:
    promotion_action_types:
      my_promotion_action:
        name: My Promotion Action
        description: Performs my promotion action.
```

## Rules

There are five rules which come with Spree:

* `FirstOrder`: The user's order is their first.
* `ItemTotal`: The order's total is greater than (or equal to) a given value.
* `Product`: An order contains a specific product.
* `User` The order is by a specific user.
* `UserLoggedIn`: The user is logged in.

Rules are used by Spree to determine if a promotion is applicable to an order and can be matched in one of two ways: all of the rules must match, or one rule must match. This is determined by the `match_policy` attribute on the `Promotion` object.

$$$
Offer examples of ways to use the match_policy attribute
$$$

### Registering a New Rule

To register a new rule with Spree, first define a class that inherits from `Spree::PromotionRule`, like this:

```ruby
class MyPromotionRule < Spree::PromotionRule
  def eligible?(order)
    ...
  end
end
```

The `eligible?` method should then return `true` or `false` to indicate if the promotion should be eligible for an order. You can retrieve promotion information by calling `promotion`.

Then register it using this code inside `config/initializers/spree.rb`:

```ruby
Rails.application.config.spree.promotions.rules << MyPromotionRule
```

NOTE: proper location and file name for the rule in this example would be: app/models/spree/promotion/rules/my_promotion_rule.rb

To get your rule to appear in the admin promotions interface you have a few more changes to make.

Create a partial for your new rule in app/views/spree/admin/promotions/rules/_my_promotion_rule.html.erb.

This file can be as simple as an empty file if your rule requires no parameters, or it may require more complex markup to enable setting values for your new rule. Check out some of the rule partials provided with Spree in the backend sources.

And finally, your new rule must have a name and description defined for the locale you will be using it in. For English, edit config/locales/en.yml and add the following to support our new example rule:

```
en:
  spree:
    promotion_rule_types:
            my_promotion_rule:
              name: "My Promotion Rule"
              description: "Rule to define my new promotion"
```

Restart your application. Once this rule has been registered, it will be available within Spree's admin interface.

$$$
Write about Spree::Promo::CouponApplicator
$$$