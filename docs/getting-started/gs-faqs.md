---
title: FAQs
description: Common questions about implementing UID2.
hide_table_of_contents: false
sidebar_position: 20
---

# Frequently Asked Questions

Frequently asked questions for UID2 are grouped into general categories by audience.

<!-- This page includes:

- [FAQs&#8212;General](#faqsgeneral)
- [FAQs for Publishers](#faqs-for-publishers)
- [FAQs for Advertisers and Data Providers](#faqs-for-advertisers-and-data-providers)
- [FAQs for DSPs)](#faqs-for-dsps) -->

## FAQs&#8212;General

Here are some frequently asked questions regarding the UID2 framework.

   - [Will all integration partners in the EUID infrastructure (SSPs, third-party data providers, measurement providers) be automatically integrated with UID2?](#will-all-integration-partners-in-the-euid-infrastructure-ssps-third-party-data-providers-measurement-providers-be-automatically-integrated-with-uid2)
   - [Can users opt out of targeted advertising tied to their UID2 identity?](#can-users-opt-out-of-targeted-advertising-tied-to-their-uid2-identity)
   - [How does a user know where to access the opt-out portal?](#how-does-a-user-know-where-to-access-the-opt-out-portal)

#### Will all integration partners in the EUID infrastructure (SSPs, third-party data providers, measurement providers) be automatically integrated with UID2?
<!-- FAQ_01 -->
No. UID2 has its own framework, which is separate from EUID. As such, paperwork relating to accessing and using the EUID framework does not automatically grant usage and access to the UID2 framework. New contracts are required to be signed for UID2.

#### Can users opt out of targeted advertising tied to their UID2 identity?
<!-- FAQ_02 -->
Yes. Through the [Transparency and Control Portal](https://www.transparentadvertising.com/), users can opt out from being served targeted ads tied to their UID2 identity. Each request is distributed through the UID2 Opt-Out Service and UID2 Operators to all relevant participants. 

#### How does a user know where to access the opt-out portal?
<!-- FAQ_03 -->
Publishers, SSO providers, or consent management platforms disclose links to the [Transparency and Control Portal](https://www.transparentadvertising.com/) in their login flows, consent flows, and privacy policies, and by other means.

## FAQs for Publishers

Here are some frequently asked questions for publishers using the UID2 framework.

  - [How can I test that the DII sent and the returned token match up?](#how-can-i-test-that-the-dii-sent-and-the-returned-token-match-up)
  - [Do I need to decrypt tokens?](#do-i-need-to-decrypt-tokens)
  - [How will I be notified of user opt-out?](#how-will-i-be-notified-of-user-opt-out)
  - [Where should I make token generation calls&#8212;from the server or client side?](#where-should-i-make-token-generation-callsfrom-the-server-side-or-the-client-side)
  - [Can I make token refresh calls from the client side?](#can-i-make-token-refresh-calls-from-the-client-side)
  - [How can I test the refresh token workflow?](#how-can-i-test-the-refresh-token-workflow)
  - [What is the uniqueness and rotation policy for UID2 tokens?](#what-is-the-uniqueness-and-rotation-policy-for-uid2-tokens)
  - [What does a UID2 token look like in the bid stream?](#what-does-a-uid2-token-look-like-in-the-bid-stream)

#### How can I test that the DII sent and the returned token match up?

You can use the [POST /token/validate](../endpoints/post-token-validate.md) endpoint to check whether the [DII](../ref-info/glossary-uid.md#gl-dii) that you are sending through [POST /token/generate](../endpoints/post-token-generate.md) is valid. `POST /token/validate` is used primarily for testing purposes.

For details, see [Using POST /token/validate to Test](../endpoints/post-token-validate.md#using-post-tokenvalidate-to-test).

#### Do I need to decrypt tokens?

No, publishers do not need to decrypt [UID2 tokens](../ref-info/glossary-uid.md#gl-uid2-token). However, if you want to get access to [raw UID2s](../ref-info/glossary-uid.md#gl-raw-uid2) for internal use only, please work with UID2 support to gain access.

#### How will I be notified of user opt-out?

If the user has opted out, the API response notifies you in either of these cases:
- When you generate the UID2 token by a call to the [POST /token/generate](../endpoints/post-token-generate.md) endpoint, either directly or via one of the UID2 SDKs, using the required `optout_check` parameter with a value of `1`.
- When you refresh the UID2 token by a call to the [POST /token/refresh](../endpoints/post-token-refresh.md) endpoint, either directly or via one of the UID2 SDKs.

#### Where should I make token generation calls&#8212;from the server side or the client side?

UID2 tokens must be generated only on the server side after authentication. In other words, to ensure that the API key used to access the service remains secret, the [POST /token/generate](../endpoints/post-token-generate.md) endpoint must be called only from the server side.

#### Can I make token refresh calls from the client side?

Yes. The [POST /token/refresh](../endpoints/post-token-refresh.md) can be called from the client side (for example, a browser or a mobile app) because it does not require using an API key.

#### How can I test the refresh token workflow?

You can use the `optout@email.com` email address or the `+00000000000` phone number to test your token refresh workflow. Using either parameter value in a request always generates an identity response with a `refresh_token` that results in a logout response.

The procedure is a little different depending on whether or not you are using an SDK.

##### With SDK:

1. Depending on whether the DII is an email address or a phone number, send a [POST /token/generate](../endpoints/post-token-generate.md) request using one of the following values:
    - The `optout@email.com` as the `email` value.
    - The hash of `optout@email.com` as the `email_hash` value. 
    - The `+00000000000` as the `phone` value.
    - The hash of `+00000000000` as the `phone_hash` value.
2. Wait until the SDK's [background auto-refresh](../sdks/client-side-identity.md#background-token-auto-refresh) attempts to refresh the advertising token (this can take several hours) and observe the refresh attempt fail with the `OPTOUT` status. At this point the SDK also clears the first-party cookie.

##### Without SDK:

1. Depending on whether the [DII](../ref-info/glossary-uid.md#gl-dii) is an email address or a phone number, send a [POST /token/generate](../endpoints/post-token-generate.md) request using one of the following values:
    - The `optout@email.com` as the `email` value.
    - The hash of `optout@email.com` as the `email_hash` value. 
    - The `+00000000000` as the `phone` value.
    - The hash of `+00000000000` as the `phone_hash` value.
2. Store the returned `refresh_token` for use in the following step.
3. Send a [POST /token/refresh](../endpoints/post-token-refresh.md) request with the `refresh_token` (saved in step 2) as the `token` value.<br/>The body response should be empty, and the `status` value should be set to `optout` because the `optout@email.com` email and the `+00000000000` phone number always result in a logged out user.

#### What is the uniqueness and rotation policy for UID2 tokens?

The UID2 service encrypts UID2 tokens using random initialization vectors. The UID2 token is unique for a given user as the user browses the internet. This means that every time a UID2 token is generated, the token is always different, even for the same underlying raw UID2. Every time the token is refreshed, a new token is generated and encrypted. This mechanism helps ensure that untrusted parties cannot track a user's identity.

#### What does a UID2 token look like in the bid stream?

There are many ways to approach UID2 implementation. Here is one example of a code snippet showing how a UID2 token is passed in the bid stream:

```javascript
{
  "user":{
    "ext":{
      "eids":[
        {
          "source":"uidapi.com",
          "uids":[
            {
              "id":"AgAAAHcy2ka1tSweERARV/wgwM+zM5wK98b9XItZGVgHaU23Eh0XOmAixO6VBcMd3k2ir/TGHLf7O7kQGLyeRPC5/VBSPmugOblMlzgy0B1ZfHQ7ccVurbyzgL1ZZOZ5cBvPDrvfR9MsKqPgWvrIKRkKVTYyUkG5YRAc++xRKfbL/ZSYxQ==",
              "rtiPartner":"UID2"
            }
          ]
        }
      ]
    }
  }
}
```

## FAQs for Advertisers and Data Providers

Here are some frequently asked questions for advertisers and data providers using the UID2 framework.

   - [How do I know when to refresh the UID2 due to salt bucket rotation?](#how-do-i-know-when-to-refresh-the-uid2-due-to-salt-bucket-rotation)
   - [Do refreshed emails get assigned to the same bucket with which they were previously associated?](#do-refreshed-emails-get-assigned-to-the-same-bucket-with-which-they-were-previously-associated)
   - [How often should UID2s be refreshed for incremental updates?](#how-often-should-uid2s-be-refreshed-for-incremental-updates)
   - [How should I generate the SHA-256 of DII for mapping?](#how-should-i-generate-the-sha-256-of-dii-for-mapping)
   - [Should I store large volumes of email address, phone number, or their hash mappings? ](#should-i-store-large-volumes-of-email-address-phone-number-or-their-hash-mappings)
   - [How should I handle user optouts?](#how-should-i-handle-user-optouts)

#### How do I know when to refresh the UID2 due to salt bucket rotation?
<!-- FAQ_19 ADP -->
Metadata supplied with the UID2 generation request indicates the salt bucket used for generating the UID2. Salt buckets persist and correspond to the underlying [DII](../ref-info/glossary-uid.md#gl-dii) used to generate a UID2. Use the  [POST /identity/buckets](../endpoints/post-identity-buckets.md) endpoint to return which salt buckets rotated since a given timestamp. The returned rotated salt buckets inform you which UID2s to refresh.

#### Do refreshed emails get assigned to the same bucket with which they were previously associated?
<!-- FAQ_20 ADP -->
Not necessarily. After you remap emails associated with a particular bucket ID, the emails might be assigned to a different bucket ID. To check the bucket ID, [call the mapping function](../guides/advertiser-dataprovider-guide.md#retrieve-a-raw-uid2-for-dii-using-the-identity-map-endpoints) and save the returned UID2 and bucket ID again.

>IMPORTANT: When mapping and remapping emails, be sure not to make any assumptions of the number of buckets, their specific rotation dates, or to which bucket an email gets assigned. 

#### How often should UID2s be refreshed for incremental updates?

The recommended cadence for updating audiences is daily.

Even though each salt bucket is updated roughly once a year, individual bucket updates are spread over the year. This means that about 1/365th of all buckets are rotated daily. If fidelity is critical, consider calling the [POST /identity/buckets](../endpoints/post-identity-buckets.md) endpoint more frequently, for example, hourly.

#### How should I generate the SHA-256 of DII for mapping?
<!-- FAQ_22 ADP -->
The system should follow the [email normalization rules](gs-normalization-encoding.md#email-address-normalization) and hash without salting.

#### Should I store large volumes of email address, phone number, or their hash mappings? 
<!-- FAQ_23 ADP -->
Yes. Not storing mappings may increase processing time drastically when you have to map millions of email addresses or phone numbers. Recalculating only those mappings that actually need to be updated, however, reduces the total processing time because only about 1/365th of UID2s need to be updated daily.

>IMPORTANT: Unless you are using a private operator, you must map email addresses, phone numbers, or hashes consecutively, using a single HTTP connection, in batches of 5,000 emails at a time. In other words, do your mapping without creating multiple parallel connections. 

#### How should I handle user optouts?
<!-- FAQ_24 ADP -->
When a user opts out of UID2-based targeted advertising through the [Transparency and Control Portal](https://www.transparentadvertising.com/), the optout signal is sent to DSPs and publishers, which handle optouts at bid time. As an advertiser or data provider, you do not need to check for UID2 optout in this scenario.

If a user opts out through your website, you should follow your internal procedures for handling the optout, for example, you might choose not to generate a UID2 for that user.

## FAQs for DSPs

Here are some frequently asked questions for demand-side platforms (DSPs).

   - [How do I know which decryption key to apply to a UID2?](#how-do-i-know-which-decryption-key-to-apply-to-a-uid2)
   - [Where do I get the decryption keys?](#where-do-i-get-the-decryption-keys)
   - [How do I know if/when the salt bucket has rotated?](#how-do-i-know-ifwhen-the-salt-bucket-has-rotated)
   - [Should the DSP be concerned with latency?](#should-the-dsp-be-concerned-with-latency)
   - [How should the DSP maintain proper frequency capping with UID2?](#how-should-the-dsp-maintain-proper-frequency-capping-with-uid2)
   - [Will all user opt-out traffic be sent to the DSP?](#will-all-user-opt-out-traffic-be-sent-to-the-dsp)
   - [Is the DSP expected to handle opt-out signals only for the UID2s that they already store?](#is-the-dsp-expected-to-handle-opt-out-signals-only-for-the-uid2s-that-they-already-store)
   - [How long should the DSP keep the opt-out list?](#how-long-should-the-dsp-keep-the-opt-out-list)
   - [Is the UID2 of an opted-out user sent to the opt-out endpoint in an encrypted form?](#is-the-uid2-of-an-opted-out-user-sent-to-the-opt-out-endpoint-in-an-encrypted-form)
   - [What request type do opt-outs use?](#what-request-type-do-opt-outs-use)
   - [How strict are the requirements for honoring opt-outs?](#how-strict-are-the-requirements-for-honoring-opt-outs)
   - [How many decryption keys may be present in memory at any point?](#how-many-decryption-keys-may-be-present-in-memory-at-any-point)
   - [How do SDK errors impact the DSP's ability to respond to a bid?](#how-do-sdk-errors-impact-the-dsps-ability-to-respond-to-a-bid)

#### How do I know which decryption key to apply to a UID2?
<!-- FAQ_25 DSP -->
Each of the server-side SDKs  (see [SDKs](../sdks/summary-sdks.md)) updates decryption keys automatically. Metadata supplied with the UID2 token discloses the IDs of the decryption keys to use. 

#### Where do I get the decryption keys?
<!-- FAQ_26 DSP -->
You can use one of the server-side SDKs (see [SDKs](../sdks/summary-sdks.md)) to communicate with the UID2 service and fetch the latest keys. To make sure that the keys remain up-to-date, it is recommended to fetch them periodically, for example, once every hour. 

#### How do I know if/when the salt bucket has rotated?
<!-- FAQ_27 DSP -->
The DSP is not privy to when the UID2 salt bucket rotates. This is similar to a DSP being unaware if users cleared their cookies. Salt bucket rotation has no significant impact on the DSP.  

#### Should the DSP be concerned with latency?
<!-- FAQ_28 DSP -->
The UID2 service does not introduce latency into the bidding process. Any latency experienced can be attributed to the network, not the UID2 service.

#### How should the DSP maintain proper frequency capping with UID2?
<!-- FAQ_29 DSP -->
The UID2 has the same chance as a cookie of becoming stale. Hence, the DSP can adapt the same infrastructure currently used for cookie or deviceID-based frequency capping for UID2. For details, see [How do I know when to refresh the UID2 due to salt bucket rotation?](#how-do-i-know-when-to-refresh-the-uid2-due-to-salt-bucket-rotation)

#### Will all user opt-out traffic be sent to the DSP?
<!-- FAQ_30 DSP -->
Yes, all opt-outs from the UID2 [Transparency and Control Portal](https://www.transparentadvertising.com/) hit the opt-out endpoint, which the DSP must configure to [honor user opt-outs](../guides/dsp-guide.md#honor-user-opt-outs).

#### Is the DSP expected to handle opt-out signals only for the UID2s that they already store?
<!-- FAQ_31 DSP -->
In some cases a DSP may receive a UID2 token for a newly-stored UID2 where the token is generated before the opt-out timestamp. The DSP is not allowed to bid on such tokens. It is therefore recommended to store all opt-out signals regardless of whether the corresponding UID2 is currently stored by the DSP or not. For details, see the diagram in [Bidding Opt-Out Logic](../guides/dsp-guide.md#bidding-opt-out-logic).

#### How long should the DSP keep the opt-out list?
<!-- FAQ_32 DSP -->
We recommend that you keep the opt-out information indefinitely.

#### Is the UID2 of an opted-out user sent to the opt-out endpoint in an encrypted form?
<!-- FAQ_33 DSP -->
No. It is sent as an unencrypted (raw) UID2.

#### What request type do opt-outs use? 
<!-- FAQ_34 DSP -->
Typically GET requests, but different DSPs may use different types.

#### How strict are the requirements for honoring opt-outs? 
<!-- FAQ_35 DSP -->
Opt-outs must be always respected. It may take some time for an opt-out request to propagate through the system during which time it is expected that some bids may not honor the opt-out.

#### How many decryption keys may be present in memory at any point?
<!-- FAQ_36 DSP -->
There may be thousands of decryption keys present in the system at any given point.

#### How do SDK errors impact the DSP's ability to respond to a bid?
<!-- FAQ_37 DSP -->
If there is an error, the SDK will not decrypt the UID2 token into a UID2.
