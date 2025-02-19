---
sidebar_position: 2
---

# Guides

From a technical integration angle, the primary use of Paycrest's contracts and APIs is to enable off-ramp transactions. This can be done using the Pay Crest contract directly or through the Sender's API, which has a slightly different UX to accommodate third-party apps.

1) **Gateway contract**: The gateway contract is an entry contract that allows connected wallets on dapps to authorize transactions to enable off-ramp to a fiat account.

2) **Sender API**: This allows third-party apps to enable their users to off-ramp using a receiving address. This means they don't have to connect their wallet to authorize the transaction with the gas fee. Instead, the order is initiated using an API, while the user completes it by sending crypto to a receiving address and their fiat account is funded.