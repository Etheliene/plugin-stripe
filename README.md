# Payload Stripe Plugin

[![NPM](https://img.shields.io/npm/v/@payloadcms/plugin-stripe)](https://www.npmjs.com/package/@payloadcms/plugin-stripe)

A plugin for [Payload CMS](https://github.com/payloadcms/payload) to manage a [Stripe](https://stripe.com/) account through Payload.

Core features:
  - Layers your Stripe account with Payload access control
  - Opens custom routes to interface with the Stripe REST API
  - Handles Stripe webhooks to allow for a two-way data sync

## Installation

```bash
  yarn add @payloadcms/plugin-stripe
  # OR
  npm i @payloadcms/plugin-stripe
```

## Basic Usage

In the `plugins` array of your [Payload config](https://payloadcms.com/docs/configuration/overview), call the plugin with [options](#options):

```js
import { buildConfig } from 'payload/config';
import stripe from '@payloadcms/plugin-stripe';

const config = buildConfig({
  plugins: [
    stripe({
      stripeSecretKey: process.env.STRIPE_SECRET_KEY,
    })
  ]
});

export default config;
```

### Options

- `stripeSecretKey`

  Required. Your Stripe secret key.

- `stripeWebhookEndpointSecret`

  Optional. Your Stripe webhook endpoint secret. This is needed only if you wish to sync data from Stripe to Payload.

- `webhooks`

  Optional. An object of Stripe webhook handlers, keyed to the name of the event. See [webhooks](#webhooks) for more details or for a list of all available webhooks, see [here](https://stripe.com/docs/cli/trigger#trigger-event).

### Routes

The following routes are automatically opened to allow you to interact with the Stripe API behind Payload access control. This allows you to read and make updates to your account.

>NOTE: the `/api` part of these routes may be different based on the settings defined in your Payload config.

- `POST /api/stripe/rest`

  Calls the given [Stripe REST API](https://stripe.com/docs/api) method and returns the result.

  ```
    const res = await fetch(`/api/stripe/rest`, {
      body: JSON.stringify({
        stripeMethod: "stripe.subscriptions.list",
        stripeMethodArgs: {
          customer: "abc"
        }
      })
    })
  ```

- `POST /api/stripe/webhooks`

  Returns an http status code. This is where all Stripe webhook events are sent to be handled. See [webhooks](#webhooks).

### Webhooks

This plugin also allows for a two-way data sync from Stripe to Payload using [Stripe webhooks](https://stripe.com/docs/webhooks). Webhooks listen for events on your Stripe account so you can trigger reactions to them. To enable webhooks:

1. Login and [create a new webhook](https://dashboard.stripe.com/test/webhooks/create) from the Stripe dashboard
1. Paste `/api/stripe/webhooks` as the "Webhook Endpoint URL"
1. Select which events to broadcast
1. Then, handle these events using the `webhooks` portion of this plugin's config:

```js
import { buildConfig } from 'payload/config';
import stripe from '@payloadcms/plugin-stripe';

const config = buildConfig({
  plugins: [
    stripe({
      stripeSecretKey: process.env.STRIPE_SECRET_KEY,
      stripeWebhooksEndpointSecret: process.env.STRIPE_WEBHOOKS_ENDPOINT_SECRET,
      webhooks: {
        'customer.subscription.updated': () => {}
      }
    })
  ]
});

export default config;
```

For a full list of available webhooks, see [here](https://stripe.com/docs/cli/trigger#trigger-event).

## TypeScript

All types can be directly imported:

```js
import {
  StripeConfig,
  WebhookHandler
} from '@payloadcms/plugin-stripe/dist/types';
```

### Development

This plugin can be developed locally using any Stripe account that you have access to. Then:

```bash
git clone git@github.com:payloadcms/plugin-stripe.git \
  cd plugin-stripe && yarn \
  cd demo && yarn \
  cp .env.example .env \
  vim .env \ # add your Stripe creds to this file
  yarn dev
```

Now you have a running Payload server with this plugin installed, so you can authenticate and begin hitting the routes. To do this, open [Postman](https://www.postman.com/) and import [our config](https://github.com/payloadcms/plugin-stripe/blob/main/src/payload-stripe-plugin.postman_collection.json). First, login to retrieve your Payload access token. This token is automatically attached to the header of all other requests.

## Screenshots

  <!-- ![screenshot 1](https://github.com/@payloadcms/plugin-stripe/blob/main/images/screenshot-1.jpg?raw=true) -->
