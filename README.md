# {settpay-sdk}

## SettPay Plugin SDK (SP-SDK)

Flutter SDK for building SettPay Plugins (SPs).
SDK for integrating blockchain network as a plugin.

# Building a settpay_plugin dart package.

## 1. Create your plugin repo

create a dart package
```shell
flutter create --template=package sp_polkadot
cd sp_polkadot
```
add dependencies in pubspec.yaml
```yaml
dependencies:
  settpay_sdk: ^1.0.0
```
and install the dependencies.
```shell
flutter pub get
```

## 2. Build your polkadot-js wrapper

The App use a `polkadot-js/api` instance running in a hidden webView
to connect to remote node.

Example:
 - SettPay.JS: [https://github.com/setheum-js/settpay.js](https://github.com/setheum-js/settpay.js)

## 3. Implement your plugin class

Modify the plugin entry file(eg. sp_polkadot.dart),
create a `PluginFoo` class extending `SettPayPlugin`:
```dart
class PluginPolkadot extends SettPayPlugin {
  /// define your own plugin
}
```

#### 3.1. override `SettPayPlugin.basic`
```dart
  @override
  final basic = PluginBasicData(
    name: 'polkadot',
    ss58: 0,
    primaryColor: Colors.deepPurple,
    gradientColor: Colors.blue,
    // The `bg.png` will be displayed as texture on a block with theme color,
    // so it should have a transparent or dark background color.
    backgroundImage: AssetImage('packages/sp_polkadot/assets/images/bg.png'),
    icon:
        Image.asset('packages/sp_polkadot/assets/images/logo.png'),
    // The `logo_gray.png` should have a gray color `#9e9e9e`.
    iconDisabled: Image.asset(
        'packages/sp_polkadot/assets/images/logo_gray.png'),
    isTestNet: false,
  );
```

#### 3.2. override `SettPayPlugin.tokenIcons`
Define the icon widgets so the SettPay App can display tokens
of your para-chain with token icons.
```dart
  @override
  final Map<String, Widget> tokenIcons = {
    'KSM': Image.asset(
        'packages/sp_polkadot/assets/images/tokens/KSM.png'),
    'DOT': Image.asset(
        'packages/sp_polkadot/assets/images/tokens/DOT.png'),
  };
```

#### 3.3. override `SettPayPlugin.nodeList`

```dart
const node_list = [
  {
    'name': 'Kusama (Polkadot Canary, hosted by Parity)',
    'ss58': 2,
    'endpoint': 'wss://kusama-rpc.polkadot.io',
  },
];
```
```dart
  @override
  List<NetworkParams> get nodeList {
    return node_list.map((e) => NetworkParams.fromJson(e)).toList();
  }
```

#### 3.4. override `SettPayPlugin.getNavItems(BuildContext, Keyring)`
Define your custom navigation-item in `BottomNavigationBar` of SettPay App.
The `HomeNavItem.content` is the page content widget displayed while your navItem was selected.
```dart
  @override
  List<HomeNavItem> getNavItems(BuildContext context, Keyring keyring) {
    return [
      HomeNavItem(
        text: 'Polkadot',
        icon: SvgPicture.asset(
          'packages/sp_polkadot/assets/images/logo.svg',
          color: Theme.of(context).disabledColor,
        ),
        iconActive: SvgPicture.asset(
            'packages/sp_polkadot/assets/images/logo.svg'),
        content: PolkadotEntry(this, keyring),
      )
    ];
  }
```

#### 3.5. override `SettPayPlugin.getRoutes(Keyring)`
Define navigation route for your plugin pages.
```dart
  @override
  Map<String, WidgetBuilder> getRoutes(Keyring keyring) {
    return {
      TxConfirmPage.route: (_) =>
          TxConfirmPage(this, keyring, _service.getPassword),
      CurrencySelectPage.route: (_) => CurrencySelectPage(this),
      AccountQrCodePage.route: (_) => AccountQrCodePage(this, keyring),

      TokenDetailPage.route: (_) => TokenDetailPage(this, keyring),
      TransferPage.route: (_) => TransferPage(this, keyring),

      // other pages
      // ...
    };
  }
```

#### 3.6. override `SettPayPlugin.loadJSCode()` method
Load the `polkadot-js/api` wrapper you built in step 2.
```dart
  @override
  Future<String> loadJSCode() => rootBundle.loadString(
      'packages/sp_polkadot/lib/js_service_polkadot/dist/main.js');
```

#### 3.7. override plugin life-circle methods
 - onWillStart(), you may want to prepare your plugin state data here.
 - onStarted(), remote node connected, you may fetch data from network.
 - onAccountChanged(), user just changed account, you may clear
 cache of the prev account and query data for new account.

// TODO: Add Example: Examples:
//  - [sp_polkadot](https://github.com/SettPay/sp_polkadot/blob/master/lib////sp_polkadot.dart)
