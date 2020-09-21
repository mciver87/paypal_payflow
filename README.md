# NC eCommerce

Use this module to create eCommerce integrations with PayPal's Payflow Payment Gateway.

This will allow you to create a webform associated with a good or service or some kind,
to which you will assign a price in USD. Then you can create an order form with an array
of optional fields, including the ability for the user to select quantity to purchase.

The user will then be sent to a hosted checkout page configured in Your Payflow Link
Gateway account, where they will enter their payment information on a PCI-compliant page.

Once the user successfully complete the payment, PayPal will send Drupal the transaction ID,
which will be updated in the webform submission they completed to get to the Payflow
payment page.

https://www.paypal.com/us/webapps/mpp/payflow-payment-gateway

## Account Setup

At the above link, choose "get started," and create a Payflow Link gateway account.

Set up a "Hosted Checkout Page" -- for more details, visit:
https://developer.paypal.com/docs/payflow/integration-guide/configure-hosted-checkout/

For debugging, set "Transaction Process Mode" to "Test." There is also a place to
enable test mode on the webform handler that should be checked as well in this case.

An ideal "Cancel URL" for the hosted page would be the corresponding webform URL
that you create in Drupal. You can add it later.

Set fields that are required in order to allow a purchase. You can also make the fields
uneditable so that they get passed directly from Drupal (see below).

### Payment Confirmation

OPTIONAL: you may set the confirmation page to be "on my website" and set the path
to https://yoursite.com/purchase-confirmation to display a simple thank you page
with the user's PayPal transaction ID after they complete the purchase.

### Silent Post for Data Transfer

HIGHLY RECOMMENDED: Set "Use Silent Post" to Yes, and the "Silent Post URL" to

https://yoursite.com/paypal-submission-update

This will ensure that PayPal updates the Drupal Webform Submission in the webform
you configure below with the PayPal transaction ID once the purchase is completed.

We also recommend you check "Void transaction when my server fails to receive data sent by the silent post."

Because if Drupal fails to receive the transaction, you won't have the transaction ID in the webform,
which could cause confusion from a customer service perspective.

The return URL for silent post failure could go back to the webform.

## User Credentials

You will need your Merchant Login ID (not the password) in the next step.

You also need to create a user with the "FULL_TRANSACTIONS" role. Take note of the user credentials,
as you'll need to provide those to Drupal when configuring your order form.

## Drupal Configuration - Creating Your Order Webform

Create a new webform and add it to a page. Go to webform -> settings -> handlers:

`/admin/structure/webform/manage/{WEBFORM_ID}/handlers`

add the PayPal Purchase Submission handler. This requires the following:

- Vendor - Your Merchant Login ID
- PayPal Manager Username - The user you created with "FULL_TRANSACTIONS" role
- User Password - The password for the user you created
- Test Implementation - Check if you are going to run test transaction in "Test" mode
- Price (Optional) - The price of the item to be sold in your order form.

Once your handler is configured, here are the field requirements:
Note: The fields must have the values specified below as their `key` in order to work.

### Product Fields: for Selling Multiple Products on one Form

If you do not define price in the handler, then the handler will expect product fields.

(Otherwise, the total price the user sees will be 0!)

Product fields should be keyed as `product_*`, and should have a custom property
under the "advanced" tab in the field settings of `price: X`, where X is the unit
price of the product.

### Required Fields

- `transaction_id` -- Hidden field that will be automatically populated with a PayPal
transaction ID in the event of a successful payment.

### Optional Fields

These fields are totally optional, but if configured with the below "key" values,
they will be passed along to the PayPal hosted checkout form. That way, you can collect
user submission data in Drupal and PayPal without the user entering anything twice.

Note that the hosted checkout page on PayPal can be configured such that these fields
are not editable on the PayPal form, which could ensure consistency across the Drupal
and PayPal submission data.

- `quantity` -- Optional numeric field. If the selected value is > 1, price will be
multiplied by this value. Otherwise transaction price will be as set in handler.
- `first_name` - text field
- `last_name` - text field
- `billingaddress1` - text field
- `billingaddress2` - text field
- `billingcity` - text field
- `billingstate` - select with U.S. state options
- `billingzip` - number field
- `phone` - phone field
- `email` - email field

## Debugging Tips

In test mode, use the CC #s on this page to create test transactions:

https://developer.paypal.com/docs/payflow/payflow-pro/payflow-pro-testing/#credit-card-numbers-for-testing

## Example Webform Configuration

You can import the below config @ `/admin/config/development/configuration/single/import`
to see an example webform using this handler. You'll have to add your own login details.

```
uuid: 8431381b-68aa-4607-b614-b5ac00c61143
langcode: en
status: open
dependencies:
  module:
    - nc_ecommerce
open: null
close: null
weight: 0
uid: 173
template: false
archive: false
id: ncc_paypal_example
title: PayPal Example
description: '<p>Example implementation for Payment Processing via PayPal Payflow</p>'
category: ''
elements: |
  quantity:
    '#type': number
    '#title': Quantity
    '#multiple__no_items_message': '<p>No items entered. Please add items below.</p>'
  first_name:
    '#type': textfield
    '#title': 'First Name'
    '#multiple__no_items_message': '<p>No items entered. Please add items below.</p>'
  last_name:
    '#type': textfield
    '#title': 'Last Name'
    '#multiple__no_items_message': '<p>No items entered. Please add items below.</p>'
  billingaddress1:
    '#type': textfield
    '#title': 'Billing address'
    '#multiple__no_items_message': '<p>No items entered. Please add items below.</p>'
  billingaddress2:
    '#type': textfield
    '#title': 'Billing Address 2'
    '#multiple__no_items_message': '<p>No items entered. Please add items below.</p>'
  billingcity:
    '#type': textfield
    '#title': 'Billing City'
    '#multiple__no_items_message': '<p>No items entered. Please add items below.</p>'
  billingstate:
    '#type': select
    '#title': 'Billing State'
    '#options': state_codes
  billingzip:
    '#type': textfield
    '#title': Billingzip
    '#multiple__no_items_message': '<p>No items entered. Please add items below.</p>'
  phone:
    '#type': tel
    '#title': Phone
    '#multiple__no_items_message': '<p>No items entered. Please add items below.</p>'
  email:
    '#type': email
    '#title': Email
    '#multiple__no_items_message': '<p>No items entered. Please add items below.</p>'
  transaction_id:
    '#type': hidden
    '#title': transaction_id
css: ''
javascript: ''
settings:
  ajax: false
  ajax_scroll_top: form
  ajax_progress_type: ''
  ajax_effect: ''
  ajax_speed: null
  page: true
  page_submit_path: ''
  page_confirm_path: ''
  page_theme_name: ''
  form_title: both
  form_submit_once: false
  form_exception_message: ''
  form_open_message: ''
  form_close_message: ''
  form_previous_submissions: true
  form_confidential: false
  form_confidential_message: ''
  form_remote_addr: true
  form_convert_anonymous: false
  form_prepopulate: false
  form_prepopulate_source_entity: false
  form_prepopulate_source_entity_required: false
  form_prepopulate_source_entity_type: ''
  form_reset: false
  form_disable_autocomplete: false
  form_novalidate: false
  form_disable_inline_errors: false
  form_required: false
  form_unsaved: false
  form_disable_back: false
  form_submit_back: false
  form_autofocus: false
  form_details_toggle: false
  form_access_denied: default
  form_access_denied_title: ''
  form_access_denied_message: ''
  form_access_denied_attributes: {  }
  form_file_limit: ''
  share: false
  share_node: false
  share_theme_name: ''
  share_title: true
  share_page_body_attributes: {  }
  submission_label: ''
  submission_log: false
  submission_views: {  }
  submission_views_replace: {  }
  submission_user_columns: {  }
  submission_user_duplicate: false
  submission_access_denied: default
  submission_access_denied_title: ''
  submission_access_denied_message: ''
  submission_access_denied_attributes: {  }
  submission_exception_message: ''
  submission_locked_message: ''
  submission_excluded_elements: {  }
  submission_exclude_empty: false
  submission_exclude_empty_checkbox: false
  previous_submission_message: ''
  previous_submissions_message: ''
  autofill: false
  autofill_message: ''
  autofill_excluded_elements: {  }
  wizard_progress_bar: true
  wizard_progress_pages: false
  wizard_progress_percentage: false
  wizard_progress_link: false
  wizard_progress_states: false
  wizard_auto_forward: true
  wizard_start_label: ''
  wizard_preview_link: false
  wizard_confirmation: true
  wizard_confirmation_label: ''
  wizard_track: ''
  wizard_prev_button_label: ''
  wizard_next_button_label: ''
  wizard_toggle: false
  wizard_toggle_show_label: ''
  wizard_toggle_hide_label: ''
  preview: 0
  preview_label: ''
  preview_title: ''
  preview_message: ''
  preview_attributes: {  }
  preview_excluded_elements: {  }
  preview_exclude_empty: true
  preview_exclude_empty_checkbox: false
  draft: none
  draft_multiple: false
  draft_auto_save: false
  draft_saved_message: ''
  draft_loaded_message: ''
  draft_pending_single_message: ''
  draft_pending_multiple_message: ''
  confirmation_type: page
  confirmation_title: ''
  confirmation_message: ''
  confirmation_url: ''
  confirmation_attributes: {  }
  confirmation_back: true
  confirmation_back_label: ''
  confirmation_back_attributes: {  }
  confirmation_exclude_query: false
  confirmation_exclude_token: false
  confirmation_update: false
  limit_total: null
  limit_total_interval: null
  limit_total_message: ''
  limit_total_unique: false
  limit_user: null
  limit_user_interval: null
  limit_user_message: ''
  limit_user_unique: false
  entity_limit_total: null
  entity_limit_total_interval: null
  entity_limit_user: null
  entity_limit_user_interval: null
  purge: none
  purge_days: null
  results_disabled: false
  results_disabled_ignore: false
  results_customize: false
  token_view: false
  token_update: false
access:
  create:
    roles:
      - anonymous
      - authenticated
    users: {  }
    permissions: {  }
  view_any:
    roles: {  }
    users: {  }
    permissions: {  }
  update_any:
    roles: {  }
    users: {  }
    permissions: {  }
  delete_any:
    roles: {  }
    users: {  }
    permissions: {  }
  purge_any:
    roles: {  }
    users: {  }
    permissions: {  }
  view_own:
    roles: {  }
    users: {  }
    permissions: {  }
  update_own:
    roles: {  }
    users: {  }
    permissions: {  }
  delete_own:
    roles: {  }
    users: {  }
    permissions: {  }
  administer:
    roles: {  }
    users: {  }
    permissions: {  }
  test:
    roles: {  }
    users: {  }
    permissions: {  }
  configuration:
    roles: {  }
    users: {  }
    permissions: {  }
handlers:
  paypal_purchase_submission:
    id: paypalSubmission
    label: 'Paypal Purchase Submission'
    handler_id: paypal_purchase_submission
    status: true
    conditions: {  }
    weight: 0
    settings:
      vendor: your_merchant_id
      user: your_paypal_user
      user_pw: your_pw_here
      price: '100'
      test: 1
variants: {  }
```
