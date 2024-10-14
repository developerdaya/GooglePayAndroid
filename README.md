# GooglePayAndroid
To integrate **Google Pay** in your Android app, you can follow these steps. Google Pay allows users to pay quickly and securely using their Google accounts, and it’s a great addition to apps for handling payments. Below is a guide on how to integrate **Google Pay** for payments using the **Google Pay API** in Android.

### Steps to Integrate Google Pay:

### 1. **Add Required Dependencies**:
You need to add Google Play services dependency in your `build.gradle` (Module: app) file.

```gradle
dependencies {
    // Google Play services dependency for Google Pay API
    implementation 'com.google.android.gms:play-services-wallet:19.1.0'
}
```

### 2. **Request Google Pay API Access**:
You need to request Google Pay access from your Google Cloud Console.

- Go to the [Google Pay Business Console](https://pay.google.com/business/console/) and set up a new account.
- Follow the steps to register your app to access the Google Pay API.

### 3. **Create the Payment Request**:
Google Pay API requires you to create a payment request using a JSON structure. Here is an example of how to build the request.

#### JSON Structure for Payment Request:
```json
{
  "apiVersion": 2,
  "apiVersionMinor": 0,
  "allowedPaymentMethods": [
    {
      "type": "CARD",
      "parameters": {
        "allowedAuthMethods": ["PAN_ONLY", "CRYPTOGRAM_3DS"],
        "allowedCardNetworks": ["AMEX", "DISCOVER", "JCB", "MASTERCARD", "VISA"]
      },
      "tokenizationSpecification": {
        "type": "PAYMENT_GATEWAY",
        "parameters": {
          "gateway": "example",
          "gatewayMerchantId": "exampleMerchantId"
        }
      }
    }
  ],
  "transactionInfo": {
    "totalPriceStatus": "FINAL",
    "totalPrice": "12.34",
    "currencyCode": "USD"
  },
  "merchantInfo": {
    "merchantName": "Example Merchant"
  }
}
```

### 4. **Create the PaymentRequest JSON in Kotlin**:
You can build this JSON in Kotlin using the `JSONObject` class.

```kotlin
import org.json.JSONObject

fun createPaymentRequest(): JSONObject {
    val baseRequest = JSONObject()
    baseRequest.put("apiVersion", 2)
    baseRequest.put("apiVersionMinor", 0)

    val allowedPaymentMethods = JSONArray()
    val cardPaymentMethod = JSONObject()
    val cardParams = JSONObject()

    cardParams.put("allowedAuthMethods", JSONArray(listOf("PAN_ONLY", "CRYPTOGRAM_3DS")))
    cardParams.put("allowedCardNetworks", JSONArray(listOf("AMEX", "DISCOVER", "JCB", "MASTERCARD", "VISA")))

    cardPaymentMethod.put("type", "CARD")
    cardPaymentMethod.put("parameters", cardParams)

    val tokenizationSpecification = JSONObject()
    tokenizationSpecification.put("type", "PAYMENT_GATEWAY")
    val tokenizationParams = JSONObject()
    tokenizationParams.put("gateway", "example") // Replace with your payment gateway
    tokenizationParams.put("gatewayMerchantId", "exampleMerchantId")
    tokenizationSpecification.put("parameters", tokenizationParams)

    cardPaymentMethod.put("tokenizationSpecification", tokenizationSpecification)
    allowedPaymentMethods.put(cardPaymentMethod)

    baseRequest.put("allowedPaymentMethods", allowedPaymentMethods)

    val transactionInfo = JSONObject()
    transactionInfo.put("totalPriceStatus", "FINAL")
    transactionInfo.put("totalPrice", "12.34") // Replace with the actual price
    transactionInfo.put("currencyCode", "USD") // Replace with the actual currency

    baseRequest.put("transactionInfo", transactionInfo)

    val merchantInfo = JSONObject()
    merchantInfo.put("merchantName", "Example Merchant") // Replace with your merchant info
    baseRequest.put("merchantInfo", merchantInfo)

    return baseRequest
}
```

### 5. **Check if Google Pay is Available**:
Before presenting the payment sheet to the user, you should check if the Google Pay API is available on the device.

```kotlin
import com.google.android.gms.wallet.PaymentsClient
import com.google.android.gms.wallet.Wallet
import com.google.android.gms.wallet.WalletConstants

fun createPaymentsClient(activity: Activity): PaymentsClient {
    val walletOptions = Wallet.WalletOptions.Builder()
        .setEnvironment(WalletConstants.ENVIRONMENT_TEST) // Use ENVIRONMENT_PRODUCTION for live app
        .build()
    return Wallet.getPaymentsClient(activity, walletOptions)
}

fun isGooglePayAvailable(paymentsClient: PaymentsClient, readyToPayJson: JSONObject) {
    val request = IsReadyToPayRequest.fromJson(readyToPayJson.toString())
    paymentsClient.isReadyToPay(request).addOnCompleteListener { task ->
        try {
            val result = task.getResult(ApiException::class.java)
            if (result == true) {
                // Google Pay is available, proceed with showing the payment sheet
            } else {
                // Google Pay is not available
            }
        } catch (e: ApiException) {
            e.printStackTrace()
        }
    }
}
```

### 6. **Show the Google Pay Payment Sheet**:
Once you confirm that Google Pay is available, you can show the payment sheet.

```kotlin
import android.content.Intent
import com.google.android.gms.wallet.PaymentDataRequest
import com.google.android.gms.wallet.PaymentsClient

fun requestPayment(paymentsClient: PaymentsClient, paymentDataRequestJson: JSONObject, activity: Activity) {
    val paymentDataRequest = PaymentDataRequest.fromJson(paymentDataRequestJson.toString())
    if (paymentDataRequest != null) {
        val requestCode = 999
        val task = paymentsClient.loadPaymentData(paymentDataRequest)
        AutoResolveHelper.resolveTask(task, activity, requestCode)
    }
}

override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    when (requestCode) {
        999 -> {
            when (resultCode) {
                Activity.RESULT_OK -> {
                    val paymentData = PaymentData.getFromIntent(data!!)
                    // Handle the payment information
                }
                Activity.RESULT_CANCELED -> {
                    // Handle cancellation of payment
                }
                AutoResolveHelper.RESULT_ERROR -> {
                    val status = AutoResolveHelper.getStatusFromIntent(data)
                    // Handle error in payment
                }
            }
        }
    }
}
```

### 7. **Handle Payment Data**:
Once the user completes the payment, you will receive the payment data in `onActivityResult`. You can then process the payment response and confirm the transaction with your backend server.

### 8. **Testing**:
- To test the integration, you can use **Google Pay Test Cards** available in [Google's documentation](https://developers.google.com/pay/api/android/guides/test-and-deploy/integration-checklist).
- Make sure to switch from `WalletConstants.ENVIRONMENT_TEST` to `WalletConstants.ENVIRONMENT_PRODUCTION` when deploying the app to production.

### Conclusion:
- Integrating **Google Pay** simplifies the payment process for users, allowing them to make payments securely and quickly.
- You can handle multiple payment methods, including credit and debit cards.
- For live deployment, ensure you properly configure your **merchant account** and switch to the production environment.
