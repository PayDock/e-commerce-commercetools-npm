# Paydock Commercetools

Paydock commercetools provides you with the building blocks to create a checkout experience for your shoppers, allowing
them to pay using the payment method of their choice.

# Installation

## Get paydock commercetools

### npm or yarn (recommended)

```bash
npm install @paydock-commercetools/paydock
```

```bash
yarn add @paydock-commercetools/paydock
```

Import paydock into your application. You can add your own styling by overriding the rules in the CSS file.

```javascript
import PaydockCommercetoolWidget from '@paydock-commercetools/paydock';
import '@paydock-commercetools/paydock/dist/widget.css';
```

You also need to copy the icons to a public directory so that they are available for download from the browser.
we recommend placing them so that they are available via the default URL "images/paydock/*". One way to do this is to
add the following to the package.json file:

```json
{
  "name": "your-awesome-application-name",
  "version": "*.*.*",
  "scripts": {
    "postinstall": "npm run copy-images-paydock && npm run copy-logo-paydock",
    "copy-images-paydock": "copyfiles -u 5 node_modules/@paydock-commercetools/paydock/dist/assets/icons/* public/images/paydock/",
    "copy-logo-paydock": "copyfiles -u 5 node_modules/@paydock-commercetools/paydock/dist/assets/* public/images/paydock/"
  },
  "dependencies": {
    "copyfiles": "^2.4.1"
  }
}
```

Of course, you can specify a different path to your icons by passing the corresponding parameter to the constructor:

```javascript
new PaydockCommercetoolWidget({iconPath: 'path/to/paydock/plugin/icons/dir'});
```

### Embed script and stylesheet

Embed the Paydock Commercetools script element above any other JavaScript in your checkout page.

```html

<script src="paydock-commercetools/widget.js"></script>
```

Embed the Paydock Commercetools stylesheet. You can add your own styling by overriding the rules in the CSS fil

```html

<link rel="stylesheet" href="paydock-commercetools/widget.css">
```

You also need to copy the icons to a public directory so that they are available for download from the browser.
we recommend placing them so that they are available via the default URL "images/paydock/*".

Of course, you can specify a different path to your icons by passing the corresponding parameter to the constructor:
```javascript
new PaydockCommercetoolWidget({iconPath: 'path/to/paydock/plugin/icons/dir'});
```

## Create a DOM element for Drop-in

Create a DOM container element on your checkout page where you want Drop-in to be rendered and give it a descriptive id.

```html

<div id="paydock-widget-container">
    <!-- Paydock Checkout will be mounted here -->
</div>
```

# Set up Drop-in

## Create store for Drop-in

Сreate a global store where the properties of each payment method will be written when the widget is initialized.

## Initialize the payment session. Example on Vue.js

Create an instance of Drop-in and mount it to the container element you created.

### 1. Load paydock script

Load the paydock script (Make sure that all subsequent logic related to the widget and widget initialization happens
after the file is loaded), for example do something like this:

```javascript
import {loadScript} from "vue-plugin-load-script";

loadScript('https://widget.paydock.com/sdk/latest/widget.umd.js').then(() => {
    initPayment();
})
```

### 2. Set configuration data. Example:

```javascript
const config = {
    api: 'https://api.europe-west1.gcp.commercetools.com',
    auth: {
        host: 'https://auth.europe-west1.gcp.commercetools.com',
        projectKey: 'paydock',
        credentials: {
            clientId: 'some-client-id',
            clientSecret: 'some-client-secret',
            scope: 'all_neaded scopes for work yor store example "manage_orders:paydock manage_customers:paydock"'
        },
    }
}
```

### 3. Get paydock payment configuration

First of all you need to get information about the cart, customer, and methods for working with the cart

Here in the response we get the configuration, the available payment methods and the unique payment ID.

```javascript
import axios from "axios";

const getPaydockPaymentsConfiguration = async () => {
    // Fetch payment methods using a POST request       
    let response = await axios.post(`${config.api}/${config.auth.projectKey}/payments/`, {
        amountPlanned: {
            currencyCode: 'AUD',
            centAmount: 12415 //integer, amount in cents
        },
        paymentMethodInfo: {
            paymentInterface: 'Mock',
            method: 'paydock-pay',
            name: {
                en: 'Paydock'
            }
        },
        custom: {
            type: {
                typeId: 'type',
                key: "paydock-components-payment-type"
            },
            fields: {
                commercetoolsProjectKey: config.auth.projectKey,
                PaymentExtensionRequest: JSON.stringify({
                    action: "getPaymentMethodsRequest",
                    request: {}
                })
            }
        },
        transactions: [
            {
                type: "Charge",
                amount: {
                    currencyCode: "AUD",
                    centAmount: 12415 //integer, amount in cents
                },
                state: "Initial"
            }
        ]
    }, {
        headers: {
            'Content-Type': 'application/json',
            authorization: `Bearer ${await this.getAuthToken()}`
        }
    });
}
```

### 4. Add function initialize paydock checkout.

Function is responsible for

- Get the parameters
- Creating a widget
- Display of payment methods
- Widget storage and event handling

Create a new widget

```javascript
function initPaydockCheckout(paymentMethod, paydockStore, configuration, PaydockCommercetoolWidget) {
    configuration.api_commercetools = {
        url: `${config.ct.api}/${config.ct.auth.projectKey}/payments/`,
        token: localStorage.getItem(ACCESS_TOKEN)
    }
    let widget = new PaydockCommercetoolWidget({
        selector: '#' + paymentMethod.name,
        type: paymentMethod.type,
        configuration: configuration,
        userId: commerceToolCustomerId,
        paymentButtonSelector: '#paymentButton',
        radioGroupName: 'payment_method',
    });
}
```

Handle specific logic for saving card details. Сheck if the user is logged and widget.isSaveCardEnable() equal true, use
widget render methods.

```javascript
widget.renderSaveCardCheckbox();
widget.renderCredentialsSelect();

```

Set amount and currency for the widget based on the cart data

```javascript
widget.setAmount(totalPrice);
widget.setCurrency(currencyCode);
```

Display payment methods on the widget

```javascript
widget.displayPaymentMethods(paymentMethod);
``` 

Load the widget (card, bank)

```javascript
widget.loadWidget()
```

init wallets and apm`s buttons

```javascript
widget.initAPIMSButtons(type)
widget.initWalletButtons(type)
```

Get widget

```javascript
widget.widget
```

### 5. Component mount

```javascript
onMounted(async () => {
    Object.values(configuration.payment_methods).forEach(paymentMethod => {
        Object.entries(paymentMethod.config).forEach(([key, value]) => {
            if (key.includes('use_on_checkout') && value === 'Yes') {
                initPaydockCheckout(paymentMethod, paydockStore, configuration, PaydockCommercetoolWidget);
            }
        });
    });
})
```

### 6. Add a custom hook for handling Paydock payments

**During order placement, if the checkout form is valid and the payment method is paydock, we will provide the function
of creating an order through paydock in which:**

1. We receive the value of a one-time OTT token from the widget

```javascript
input[name = "payment_source_card_token"]
```

2. We collect the required data and transfer it to the widget

```javascript
widget.setAmount(totalPrice)
widget.setCurrency(currencyCode)
widget.currencyCode.setPaymentSource(paymentSource)
widget.setAdditionalInfo(additionalInfo)
```

For wallets you must set form validation state:

```javascript
widget.setIsValidForm(true);
```

3. Get vault token for the payment

```javascript
widget.getVaultToken()
```

4. We create a payment by updating the existing commercetools api "makePaymentRequest"

Create payment using the collected data

```javascript
createPayment({...}) 
```

In the createPayment function, you must perform 3 actions:

- First. Create payment, send a request to create a payment in the Paydock system - through the extension, using the
  custom field makePaymentRequest where "widget.paymentId" its unique payment ID from step 3.

```javascript
//example for "Google Pay"

let paydockResponse = document.querySelector('[name="paydock-pay-google-pay"]').value;
let chargeId = paydockResponse.data.id;
let paymentType = 'Google Pay';
let currentPaymentUrl = `${config.ct.api}/${config.ct.auth.projectKey}/payments/${widget.paymentId}`;

paydockResponse = JSON.parse(paydockResponse);

if (paydockResponse.data.status === "inreview") {
    status = 'paydock-pending'
} else {
    status = paydockResponse.data.status === 'pending' ? 'paydock-authorize' : 'paydock-paid';
}

let response = await fetchWithToken(currentPaymentUrl, {
    method: 'GET',
    headers: headers
});
let currentPayment = await response.json();

const updateData = {
    version: currentPayment.version,
    actions: [
        {
            action: "setCustomField",
            name: "makePaymentRequest",
            value: JSON.stringify({
                orderId: widget.paymentId,
                paymentId: widget.paymentId,
                amount: {
                    currency: currencyCode,
                    value: centAmount
                },
                PaydockTransactionId: paymentSource,
                PaydockPaymentStatus: status,
                PaydockPaymentType: paymentType,
                CommerceToolsUserId: commerceToolCustomerId,
                SaveCard: saveCard,
                VaultToken: vaultToken,
                AdditionalInfo: additionalInfo
            })
        }
    ]
};
const response = await fetchWithToken(currentPaymentUrl, {
    method: 'POST',
    headers: headers,
    body: JSON.stringify(updateData)
});
let payment = await response.json();
let paydockWidgetCardServerErrorBlock = document.getElementById('paydock-widget-card-server-error');
let paymentExtensionResponse = payment?.custom?.fields?.PaymentExtensionResponse ?? null
if (paymentExtensionResponse) {
    paymentExtensionResponse = JSON.parse(paymentExtensionResponse);
    if (paymentExtensionResponse.status === "Failure") {
        paydockWidgetCardServerErrorBlock.innerText = paymentExtensionResponse.message;
        paydockWidgetCardServerErrorBlock.classList.remove("hide");
        return Promise.reject(paymentExtensionResponse.message);
    }
}

```

- Second. Add payment to cart and update cart required info

```javascript
let orderPaymentStatus = 'Pending'
let orderStatus = 'Open'
let paymentExtensionResponse = payment?.custom?.fields?.PaymentExtensionResponse ?? null

if(paymentExtensionResponse){
  orderPaymentStatus = paymentExtensionResponse.orderPaymentStatus
  orderStatus = paymentExtensionResponse.orderStatus
}

response = await fetchWithToken(`${config.ct.api}/${config.ct.auth.projectKey}/carts/${cartId}`, {
    method: 'GET',
    headers: headers
});
let currentCart = await response.json();
response = await fetchWithToken(`${config.ct.api}/${config.ct.auth.projectKey}/carts/${cartId}`, {
    method: 'POST',
    headers: headers,
    body: JSON.stringify({
        version: currentCart.version,
        actions: [
            {
                action: "addPayment",
                payment: {
                    typeId: "payment",
                    id: payment.id
                }
            }
        ]
    }),
});

currentCart = await response.json();
await fetchWithToken(`${config.ct.api}/${config.ct.auth.projectKey}/orders`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    id: currentCart.id,
    orderNumber: reference,
    version: currentCart.version,
    orderState: orderStatus,
    paymentState: orderPaymentStatus
  }),
});
```

- Third. Process order

```javascript
currentCart = await response.json();
await fetchWithToken(`${config.ct.api}/${config.ct.auth.projectKey}/orders`, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        id: currentCart.id,
        orderNumber: reference,
        version: currentCart.version
    }),
});
```

5. Link the payment to the order and create the order

### 7. Transfer information to widget

To pass payment information, pass the objects of this structure to the setBillingInfo() and setShippingInfo() methods.

```javascript
setBillingInfo({
    first_name: "first_name", //string
    last_name: "last_name"l, //string
    email: "email", //string
    phone: "phone"l, //string
    address_line1: "address_line1", //string
    address_line2: "address_line2", //string
    address_city: "address_city", //string
    address_state: "address_state", //string
    address_country: "address_country", //string
    address_postcode: "address_postcode", //integer
});
```

for cart item information pass the object of this structure to the setCartItems method.

```javascript
setCartItems([
    {
        name: "name of product", // string
        type: "type", // string  (type or category name of product)
        quantity: 10, // int
        item_uri: "https://some.domain/path/to/product", // string
        image_uri: "https://cdn.some.domain/path/to/product/image",
        amount: 123.45 // float (price with two digits after the decimal point)
    }
]);
```

### 8. Thank You page

After making the payment, it redirect to the Thank you page and the text is displayed depending on the completion of the
payment.

If status pending 'yes', we use text

```json
{
  "thankYouOrderProcessed": "Your order is being processed. We’ll get back to you shortly."
} 
```

If status pending 'no', we use text

```json
{
  "thankYouOrderReceived": "Thank you. Your order has been received."
}
```

Use the function redirectToThankYouPage to receive a text status about the order.

```javascript
async function redirectToThankYouPage(router) {
    let currentPaymentUrl = `${config.ct.api}/${config.ct.auth.projectKey}/payments/${paydockStore?.paymentId}`;
    const response = await fetchWithToken(currentPaymentUrl, {
        method: 'GET',
        headers: {'Content-Type': 'application/json'}
    });

    let orderStatusInReview = currentPayment.custom.fields.PaydockPaymentStatus === 'paydock-pending' ? 'yes' : 'no'
}
```

## See also

- [Paydock website](https://paydock.com/)

## License

This repository is available under the [MIT license](LICENSE).
