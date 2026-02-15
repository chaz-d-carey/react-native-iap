# react-native-iap

[![npm version](https://badge.fury.io/js/react-native-iap.svg)](https://badge.fury.io/js/react-native-iap)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

React Native In-App Purchase module for iOS, Android, and Amazon. This library provides a unified API to handle in-app purchases and subscriptions across different platforms.

## Features

- ✅ **Cross-platform support**: iOS (StoreKit 1 & 2), Android (Play Store), and Amazon
- ✅ **TypeScript support**: Full TypeScript definitions included
- ✅ **React Hooks**: Easy-to-use `useIAP` hook for React components
- ✅ **Purchase management**: Handle consumables, non-consumables, and subscriptions
- ✅ **Purchase history**: Retrieve purchase history and available purchases
- ✅ **Error handling**: Comprehensive error handling with typed errors
- ✅ **Event listeners**: Listen to purchase updates and errors
- ✅ **StoreKit 2 support**: Modern StoreKit 2 API support for iOS 15+

## Requirements

- `react` >= 16.13.1
- `react-native` >= 0.65.1
- iOS 12.0+ (for StoreKit 1) or iOS 15.0+ (for StoreKit 2)
- Android API 24+ (Android 7.0+)

## Installation

```bash
npm install react-native-iap
# or
yarn add react-native-iap
```

### iOS Setup

```bash
cd ios && pod install && cd ..
```

> **Note**: For iOS 12.x, add SwiftUI.framework in Xcode:
> Build Phases → Link Binary With Libraries → + → SwiftUI.framework (Optional)

### Android Setup

#### Configure Payment Provider

You can support either Play Store, Amazon, or both.

**Play Store only** - Add to `android/app/build.gradle`:

```gradle
defaultConfig {
    ...
    missingDimensionStrategy "store", "play"
}
```

**Both Play Store and Amazon** - Add to `android/app/build.gradle`:

```gradle
android {
    ...
    flavorDimensions "appstore"

    productFlavors {
        googlePlay {
            dimension "appstore"
            missingDimensionStrategy "store", "play"
        }

        amazon {
            dimension "appstore"
            missingDimensionStrategy "store", "amazon"
        }
    }
}
```

#### AndroidX Support

Add to `android/build.gradle`:

```gradle
buildscript {
    ext {
        ...
        androidXAnnotation = "1.1.0"
        androidXBrowser = "1.0.0"
        minSdkVersion = 24
        kotlinVersion = "1.8.0"
    }
    dependencies {
        ...
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion"
    }
}
```

## Quick Start

### Basic Usage

```tsx
import React, {useEffect, useState} from 'react';
import {View, Button, Text} from 'react-native';
import {
  initConnection,
  getProducts,
  requestPurchase,
  finishTransaction,
  Product,
  Purchase,
  purchaseErrorListener,
  purchaseUpdatedListener,
} from 'react-native-iap';

const App = () => {
  const [products, setProducts] = useState<Product[]>([]);
  const [purchases, setPurchases] = useState<Purchase[]>([]);

  useEffect(() => {
    // Initialize connection
    initConnection();

    // Get products
    const loadProducts = async () => {
      const items = await getProducts({
        skus: ['com.example.product1', 'com.example.product2'],
      });
      setProducts(items);
    };

    loadProducts();

    // Listen to purchase updates
    const purchaseUpdateSubscription = purchaseUpdatedListener(
      async (purchase: Purchase) => {
        console.log('Purchase successful:', purchase);
        // Finish the transaction after delivering the product
        await finishTransaction({purchase});
      },
    );

    // Listen to purchase errors
    const purchaseErrorSubscription = purchaseErrorListener((error) => {
      console.log('Purchase error:', error);
    });

    return () => {
      purchaseUpdateSubscription.remove();
      purchaseErrorSubscription.remove();
    };
  }, []);

  const handlePurchase = async (productId: string) => {
    try {
      await requestPurchase({sku: productId});
    } catch (error) {
      console.error('Purchase request failed:', error);
    }
  };

  return (
    <View>
      {products.map((product) => (
        <View key={product.productId}>
          <Text>{product.title}</Text>
          <Text>{product.localizedPrice}</Text>
          <Button
            title="Buy"
            onPress={() => handlePurchase(product.productId)}
          />
        </View>
      ))}
    </View>
  );
};
```

### Using React Hooks

```tsx
import React, {useEffect} from 'react';
import {View, Button, Text} from 'react-native';
import {useIAP, withIAPContext, requestPurchase} from 'react-native-iap';

const App = () => {
  const {
    connected,
    products,
    subscriptions,
    getProducts,
    getSubscriptions,
    currentPurchase,
    currentPurchaseError,
  } = useIAP();

  useEffect(() => {
    if (connected) {
      getProducts({skus: ['com.example.product1']});
      getSubscriptions({skus: ['com.example.subscription1']});
    }
  }, [connected]);

  useEffect(() => {
    if (currentPurchase) {
      console.log('Purchase completed:', currentPurchase);
    }
  }, [currentPurchase]);

  useEffect(() => {
    if (currentPurchaseError) {
      console.log('Purchase error:', currentPurchaseError);
    }
  }, [currentPurchaseError]);

  return (
    <View>
      {products.map((product) => (
        <Button
          key={product.productId}
          title={`Buy ${product.title}`}
          onPress={() => requestPurchase({sku: product.productId})}
        />
      ))}
    </View>
  );
};

// Wrap your app with IAP context
export default withIAPContext(App);
```

## StoreKit 2 Configuration (iOS)

You can configure StoreKit mode for iOS:

```tsx
import {setup} from 'react-native-iap';

// Before calling initConnection
setup({
  storekitMode: 'STOREKIT1_MODE', // or 'STOREKIT2_MODE' or 'STOREKIT_HYBRID_MODE'
});
```

- `STOREKIT1_MODE`: Uses StoreKit 1 only (default, supports iOS 12+)
- `STOREKIT2_MODE`: Uses StoreKit 2 only (requires iOS 15+)
- `STOREKIT_HYBRID_MODE`: Uses StoreKit 2 on iOS 15+ devices, falls back to StoreKit 1 on older devices

## API Overview

### Core Methods

- `initConnection()` - Initialize the IAP connection (required on Android)
- `endConnection()` - Close the IAP connection
- `getProducts({skus})` - Get product information
- `getSubscriptions({skus})` - Get subscription information
- `requestPurchase({sku})` - Request a purchase
- `requestSubscription({sku})` - Request a subscription
- `finishTransaction({purchase})` - Finish a transaction
- `getAvailablePurchases()` - Get all available (unfinished) purchases
- `getPurchaseHistory()` - Get purchase history

### Event Listeners

- `purchaseUpdatedListener(callback)` - Listen to successful purchases
- `purchaseErrorListener(callback)` - Listen to purchase errors

### React Hooks

- `useIAP()` - React hook for IAP functionality
- `withIAPContext(Component)` - HOC to provide IAP context

## Important Notes

⚠️ **Server-side validation**: This library handles client-side purchases, but you **must** implement server-side receipt validation for production apps. This is critical for security and preventing fraud.

⚠️ **Transaction finishing**: Always call `finishTransaction()` after successfully delivering the purchased content to the user. This is required to complete the purchase flow.

## Documentation

For detailed documentation, examples, and API reference, visit:

- [Full Documentation](https://github.com/dooboolab-community/react-native-iap)
- [API Reference](https://github.com/dooboolab-community/react-native-iap#readme)
- [Example App](./IapExample)

## Contributing

Contributions are welcome! Please read our contributing guidelines and submit pull requests to the repository.

## License

MIT © [dooboolab-community](https://github.com/dooboolab-community)

## Support

- [GitHub Issues](https://github.com/dooboolab-community/react-native-iap/issues)
- [GitHub Discussions](https://github.com/dooboolab-community/react-native-iap/discussions)

## Contributors

- [hyochan](https://github.com/hyochan)
- [Andres Aguilar](https://github.com/andresesfm)
- [Jérémy Barbet](https://github.com/jeremybarbet)

---

Made with ❤️ by the react-native-iap community
