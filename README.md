# redux-persist-expo-securestore

Storage engine for [redux-persist](https://github.com/rt2zz/redux-persist) for use with [Expo's SecureStorage ](https://docs.expo.io/versions/latest/sdk/securestore.html).

> iOS: Values are stored using the keychain services as kSecClassGenericPassword. iOS has the additional option of being able to set the value’s kSecAttrAccessible attribute, which controls when the value is available to be fetched.
>
> Android: Values are stored in SharedPreferences, encrypted with Android’s Keystore system.

## Installation

Requires Expo SDK (automatically used when using [Expo](https://expo.io/) or [create-react-native-app](https://github.com/react-community/create-react-native-app)).

```bash
yarn add redux-persist-expo-securestore
# or
npm install --save redux-persist-expo-securestore
```

## Usage

Use as a `redux-persist` global storage engine:

```js
import createSecureStore from "redux-persist-expo-securestore";

import { createStore } from "redux";
import { persistStore, persistCombineReducers } from "redux-persist";
import reducers from "./reducers";

// Secure storage
const storage = createSecureStore();

const config = {
  key: "root",
  storage
};

const reducer = persistCombineReducers(config, reducers);

function configureStore() {
  // ...
  const store = createStore(reducer);
  const persistor = persistStore(store);

  return { persistor, store };
}
```

Use as an engine for only a reducer:

```js
import createSecureStore from "redux-persist-expo-securestore";

import { combineReducers } from "redux";
import { persistReducer } from "redux-persist";
import AsyncStorage from "redux-persist/lib/storage";

import { mainReducer, secureReducer } from "./reducers";

// Secure storage
const secureStorage = createSecureStore();

const securePersistConfig = {
  key: "secure",
  storage: secureStorage
};

// Non-secure (AsyncStorage) storage
const mainPersistConfig = {
  key: "main",
  storage: AsyncStorage
};

// Combine them together
const rootReducer = combineReducers({
  main: persistReducer(mainPersistConfig, mainReducer),
  secure: persistReducer(securePersistConfig, secureReducer)
});

function configureStore() {
  // ...
  let store = createStore(rootReducer);
  let persistor = persistStore(store);

  return { persistor, store };
}
```

## Jest integration

You will need to update [`transformIgnorePatterns`](https://jestjs.io/docs/en/configuration.html#transformignorepatterns-array-string) to exclude this module, as it exports untranspiled code.

As an example, the following is the suggested `transformIgnorePatterns` from the [Expo docs](https://docs.expo.io/versions/latest/guides/testing-with-jest/#jest-configuration) with `redux-persist-expo-securestore` also whitelisted:

```
node_modules/(?!((jest-)?react-native|react-clone-referenced-element|expo(nent)?|@expo(nent)?/.*|react-navigation|@react-navigation/.*|sentry-expo|native-base|redux-persist-expo-securestore))
```

## API

### `createSecureStore([options])`

#### `[options]`: `object`

Options to pass to [Expo's SecureStore](https://docs.expo.io/versions/latest/sdk/securestore/):

##### `keychainService`: `string`

iOS: The item’s service, equivalent to kSecAttrService

Android: Equivalent of the public/private key pair Alias

##### `keychainAccessible`: `enum`

iOS only: Specifies when the stored entry is accessible, using iOS’s kSecAttrAccessible property. See Apple’s documentation on keychain item accessibility. The available options are:

> Expo.SecureStore.WHEN_UNLOCKED: The data in the keychain item can be accessed only while the device is unlocked by the user.

> Expo.SecureStore.AFTER_FIRST_UNLOCK: The data in the keychain item cannot be accessed after a restart until the device has been unlocked once by the user. This may be useful if you need to access the item when the phone is locked.

> Expo.SecureStore.ALWAYS: The data in the keychain item can always be accessed regardless of whether the device is locked. This is the least secure option.

> Expo.SecureStore.WHEN_UNLOCKED_THIS_DEVICE_ONLY: Similar to WHEN_UNLOCKED, except the entry is not migrated to a new device when restoring from a backup.

> Expo.SecureStore.WHEN_PASSCODE_SET_THIS_DEVICE_ONLY: Similar to WHEN_UNLOCKED_THIS_DEVICE_ONLY, except the user must have set a passcode in order to store an entry. If the user removes their passcode, the entry will be deleted.

> Expo.SecureStore.AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY: Similar to AFTER_FIRST_UNLOCK, except the entry is not migrated to a new device when restoring from a backup.

> Expo.SecureStore.ALWAYS_THIS_DEVICE_ONLY: Similar to ALWAYS, except the entry is not migrated to a new device when restoring from a backup.

redux-persist-expo-securestore specific options:

##### `replaceCharacter`: `string`

Default: `_`

See [caveat](#caveat).

##### `replacer`: `function(key: string, replaceCharacter: string): string`

Default: replace all illegal characters by `replaceCharacter`

See [caveat](#caveat).

## Caveat

Keys for SecureStorage only support `[A-Za-z0-9.-_]`, meaning all other characters are replaced by `options.replaceCharacter` (defaults to `_`).

You may change this character by replacing `options.replaceCharacter`.

You may also change the default key transformer by replacing `options.replacer`.

## Note

Inspired by [redux-persist-sensitive-storage](https://github.com/CodingZeal/redux-persist-sensitive-storage) (which required a native module).
