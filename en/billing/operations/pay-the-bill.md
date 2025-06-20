---
title: How to top up your personal account
description: Follow this guide to top up your personal account.
---

# Top up your personal account

{% include [personal-account-balance](../_includes/personal-account-balance.md) %}

{{ yandex-cloud }} reserves the right to automatically debit your linked card during the current reporting period if your account balance exceeds the established credit limit.

 

The method for topping up your personal account depends on your legal status.

{% note info %}

A billing cycle runs automatically for [individuals](../payment/billing-cycle-individual.md) as well as [businesses and individual entrepreneurs](../payment/billing-cycle-business.md) if they have a bank card linked to their billing account.

{% endnote %}

## Individuals {#individuals}

To top up your personal account:

{% list tabs group=instructions %}

- {{ billing-interface }} {#billing}

  1. {% include [move-to-billing-step](../_includes/move-to-billing-step.md) %}
  1. Select a billing account.
  1. Click **{{ ui-key.yacloud_billing.billing.account.dashboard-overview.button_refill }}**.
  1. In the window that opens, enter the payment amount and click **{{ ui-key.yacloud_billing.billing.account.dashboard-overview.button_refill }}**.
  1. Enter your card details and click **Pay**.

{% endlist %}

{% include [payment-card-types](../../_includes/billing/payment-card-types.md) %}

Your payment will be processed in real time and completed within 15 minutes.

## Businesses and individual entrepreneurs {#legal-entities}


To top up your personal account:

1. {% include [move-to-billing-step](../_includes/move-to-billing-step.md) %}
1. Select a billing account.
1. Click **{{ ui-key.yacloud_billing.billing.account.dashboard-overview.button_refill }}**. This button is only available after [switching to paid consumption](activate-commercial.md).
1. Select a payment method:

  {% list tabs group=payments %}

   - Wire transfer {#transfer}

     Enter the payment amount and click **{{ ui-key.yacloud_billing.billing.account.dashboard-overview.popup-refill_button_company-action }}**.

     The system will generate a payment invoice. Print the invoice and use it to make a payment in a bank or using a banking client system.

     Before paying, please make sure the following is correct in your payment order:
     * Payment amount.
     * Banking details of Yandex.Cloud LLC (for Russia), Cloud Services Kazakhstan LLP (for Kazakhstan), Iron Hive doo Beograd (Serbia), or Direct Cursus Technology L.L.C. (Dubai) (for non-resients of Russia and Kazakhstan).

       {% include [legal-entity-nonresidents](../../_includes/billing/legal-entity-nonresidents.md) %}

     * Your company or individual entrepreneur TIN.
     * [Account number](../concepts/personal-account.md#id) in the payment purpose.
     * [Agreement number](../concepts/contract.md) in the payment purpose.

     [How fast the funds will be credited to your personal account](../payment/payment-methods-business.md#limits) depends on the bank performing the transaction.

     {% include [payment-bill-note](../_includes/payment-bill-note.md) %}

  - Credit or debit card {#card}

    Enter the payment amount and click **{{ ui-key.yacloud_billing.billing.account.dashboard-overview.button_refill }}**. Then enter your card details and click **Pay**.

    {% include [payment-card-types](../../_includes/billing/payment-card-types-business.md) %}

    Your payment will be processed in real time and completed within 15 minutes.

  {% endlist %}