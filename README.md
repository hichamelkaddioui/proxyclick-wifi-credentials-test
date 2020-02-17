# Proxyclick Wifi Credentials Test

> The sales team is forwarding the following customer request to you:
>
> - Our client will have multiple Proxyclick company accounts (around 5)
> - Upon check-in the visitor will receive a text message with the > credentials to use the guest Wi-Fi
> - The credentials are valid for 30 days so that if the visitor visits another office of our customer, they will be able to use to same credentials
>
> Please describe the solution you will put in place using our available building blocks (see https://api.proxyclick.com/v1/docs)

## Introduction

Let's suppose the following configuration.

- The client "ACME" has 5 distinct companies / sites, all connected to the Proxyclick API with some cutting-edge Aila interactive kiosks
- The client "ACME" has a wifi provider that allows authentication and user management through an API
- Proxyclick uses a SMS provider, say Twilio (someone's been listening)

The solution would be a **Proxyclick wifi micro-service** that would have the following responsibilities:

- Create `credentials` given a `visitor` object
- Register these `credentials` to the client's wifi provider
- Communicate the `credentials` the the `visitor`
- Revoke these `credentials` to the client's wifi provider 30 days after their creation

This micro-service could expose a _webhook_ URL accepting a `visitor`, and could be [written in Node.js](https://github.com/proxyclick/interview-wifi-credentials).

![Overview](./img/overview.png)

## Credentials creation

Upon arriving, a `visitor` checks in to the kiosk. This triggers an ID check against Proxyclick's database. Assuming the `visitor` is expected and legit, the API responds to the company and triggers the said webhook.

The wifi micro-service then generates `credentials` and asks the wifi provider to register them. If everything goes smoothly, it sends the `visitor` its new `credentials`

![Credentials creation](./img/creation.png)

## Credentials usage

The `visitor`, having received its credentials by SMS, can now use the client's wifi in any company / site!

![Credentials creation](./img/usage.png)

## Credentials revocation

30 days after their creation, the wifi micro-service asks the client's wifi provider do revoke the `credentials`, through a CRON job for instance.

![Credentials creation](./img/revocation.png)

## Notes

As you can see, I didn't specifically use the Proxyclick API. This is because the API and backend do only two things in this scenario:

1. Check a `visitor`, which is already implemented in embedded kiosk apps
2. Call a webhook, which [seems already possible](https://help.proxyclick.com/visitor-management/webhooks/) and easy enough for Customer Success to do.

Having such a micro-service enables to reuse it for other clients that might encounter the same need, rather than having a piece of software making specific calls.
