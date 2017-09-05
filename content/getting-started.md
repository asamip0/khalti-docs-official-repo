There are four steps for integrating Khalti payment to a merchant system.

- [1. Signup as merchant](#signup-as-merchant)
- [2. Understand how khalti payment works](#understand-how-khalti-payment-works)
- [3. Integration](#integration)
- [3.1. Client side integration](#client-side-integration)
- [3.2. Server side integration](#server-side-integration)

## 1. Signup as merchant
First of all you will need a merchant and a consumer accounts.
Merchant is an online business service like QFX, Kaymu, etc.
Consumer is an end user who uses Khalti to purchase products or services from merchants.

Please follow links below to create a merchant and a consumer accounts if you have not already.

- [Create a merchant account](https://khalti.com/join/merchant/)
- [Create a consumer account](https://khalti.com/join/)

## 2. Understand Khalti payment process

![Khalti payment overview](./img/khalti-payment-overview.jpg)

## 3. Integration
Now that you know how Khalti payment works. Its time to integrate it into your system.
Khalti must to be integrated at client and server.

### 3.1. Client side integration
For now there is only one way to integrate Khalti at client side, through SDKs.
We have developed SDKs for every major plaforms and we call it `Checkout`.

Checkouts provide all the necessary UIs and perform necessary processes to initiate and confirm payment.

- [Web](./checkout/web.md)
- [Android](./checkout/android.md)
- [iOS](./checkout/ios.md) (Comming soon)

### 3.1. Server side integration
After user confirms payment, it has to be verified by Khalti.
Fund from user's account is move to merchant only if verification succeeds.
Verification must be done by the merchant server using a secret key.

- [Verification api](./api/verification.md)
- [Transaction api](./api/transaction.md)