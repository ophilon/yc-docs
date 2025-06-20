---
title: Getting started with {{ yandex-cloud }}
description: In this article, you will learn how to get started with {{ yandex-cloud }}. Find out how to create Linux and Windows VMs, use {{ objstorage-name }} (S3) data storage services, set up a network and load balancers, manage access to your resources, and create clusters in a variety of databases.
---

# Getting started with {{ yandex-cloud }}

{{ yandex-cloud }} has several user interfaces, e.g., the [management console]({{ link-console-main }}) and the [command line interface](../cli/). To access any user interface, you will need a _user account_. This can be a personal Yandex account (Yandex ID) or a Yandex 360 account. For detailed instructions, see Help for [Yandex ID](https://yandex.com/support/passport/authorization/registration.html) and [Yandex 360](https://yandex.com/support/business/add-users.html).

## Creating a billing account {#new-account}

When creating your first billing account linked to your user account, you will get your [initial grant](../getting-started/usage-grant.md).

 {% list tabs group=customers %}

   - Legal entities, individual entrepreneurs, or non-residents of Russia and Kazakhstan {#businesses-entrepreneurs}

      {% include [start-for-legal-entities](../_includes/billing/billing-account-create-legal-entities.md) %}

   - Individuals {#individuals}

      {% include [start-for-individuals](../_includes/billing/billing-account-create-individual.md) %}

   {% endlist %}

## FAQ {#qa}

### Questions about billing accounts and paid use {#billing-account}

#### Why do I need a billing account with a linked bank card?

Billing accounts are used to identify users paying for {{ yandex-cloud }} resources. Even if you plan to use only free services, you still need a billing account: this is how you get your grants and promo codes. 

To activate a billing account, we ask you to link a bank card to make sure you are a human, not a robot. As soon as you link a card, a small amount will be debited from it and promptly returned. That is how we verify that your card is real. 

Payments for {{ yandex-cloud }} services and resources can be debited from your bank card only after you explicitly allow us to do it, that is, after you switch to paid use.

#### What happens after the trial period ends? Will you start debiting money right away?

{{ yandex-cloud }} does not debit money and does not invoice you until you have switched to paid use. The transition to paid use never happens automatically.

However, if your [grant](../getting-started/usage-grant.md) has expired, access to your resources will be blocked for 60 days or until you switch to paid use. For more information about the end of the trial period, see [{#T}](../getting-started/free-trial/concepts/trial-ending.md).

### Questions about the initial grant {#grant}

#### I accidentally switched to paid use. Did I lose my initial grant? Can I get it back?

No, you cannot switch back to the trial version, but the grant will not be lost. The initial grant will be used up first. For more information about the order in which funds are spent, see [Billing cycle for individuals](../billing/payment/billing-cycle-individual.md) and [Billing cycle for businesses and individual entrepreneurs](../billing/payment/billing-cycle-business.md).

#### I have not used up the initial grant, but my cloud is blocked. What should I do?

Make sure that a valid card is linked to your account. If you link a card and then delete it, your account may get blocked.

#### I was unable to use up the initial grant within 60 days. Can I still use {{ yandex-cloud }}? 

When the initial grant expires, the total unused amount is offset, and the access to your resources is suspended for 30 days. To continue using {{ yandex-cloud }}, switch to paid use.

### Questions about documents {#documents}

#### Is it safe to pay to {{ yandex-cloud }}?

The {{ yandex-cloud }} platform meets the PCI DSS requirements, which makes its cloud services safe for payment processing. For more information about PCI DSS certification, see [{#T}](../security/conform.md#pci-dss).

#### Where do I find my agreement with {{ yandex-cloud }}? 

{% include [contract-concept](../_includes/billing/contract.md) %}

There is no printed form of the offer.

The invoice is not physically provided either, but you can generate it in the console and print it out.

#### I need an agreement signed by both parties, not an offer. Can I get it?

Yes, companies and individual entrepreneurs can sign a bilateral contract. To do this, submit a request using the [Message]({{ link-console-support }}). A {{ yandex-cloud }} manager will contact you to discuss the terms.

#### How do I get invoiced?

To get an invoice for a bank payment, follow the steps described in [{#T}](../billing/operations/pay-the-bill.md#legal-entities).

{{ yandex-cloud }} does not provide paper payment documents.

## Getting started with {#what-is-next} services

Start exploring {{ yandex-cloud }} services.

{% include [quickstart-all-no-billing](../_includes/quickstart-all-no-billing.md) %}

{% include [clickhouse-disclaimer](../_includes/clickhouse-disclaimer.md) %}
