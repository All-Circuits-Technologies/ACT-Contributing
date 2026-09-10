<!--
SPDX-FileCopyrightText: 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Flutter coding standards <!-- omit in toc -->

Dart, Flutter and ACT-architecture conventions, reusable across ACT projects. Each rule has a stable
id (`RDx` for Dart, `RFLx` for Flutter and ACT architecture) so it can be cited in code review.
Project-specific conventions (the concrete manager list, app folder tree, routes) belong in a
per-project document that links here.

This standard overloads the global standards:
[global standards](coding-standards-global.md). First read
[coding standards](coding-standards.md) to understand how the standards apply and the override
mechanism.

This guide does **not** restate what the linter enforces. Follow standard
[Effective Dart](https://dart.dev/effective-dart) and
[Flutter style](https://docs.flutter.dev/) conventions, plus the `analysis_options.yaml` shared by
ACT-Flutter-Packages - configure them and let CI enforce them. The rules below cover only what a
linter cannot see: architectural decisions, ACT package usage, and conventions with more than one
reasonable option.

## Table of content <!-- omit in toc -->

- [Naming and files](#naming-and-files)
- [Types, immutability, equality](#types-immutability-equality)
- [Null-safety and error handling](#null-safety-and-error-handling)
- [Async, streams, resources](#async-streams-resources)
- [Documentation and generated code](#documentation-and-generated-code)
- [Widgets and composition](#widgets-and-composition)
- [State management (BLoC)](#state-management-bloc)
- [Application architecture](#application-architecture)
- [Configuration and logging](#configuration-and-logging)
- [Routing](#routing)
- [Internationalization](#internationalization)
- [Assets and themes](#assets-and-themes)
- [Security](#security)
- [Performance](#performance)
- [Sources](#sources)

## Naming and files

### RD1 - Prefix project types with a project code

Prefix project-specific public types with a short project code (`Nls`, `Hewo`) to avoid collisions
once several ACT apps' packages are imported side by side.

```dart
class NlsConfigManager extends AbsUsualConfigManager { /* ... */ }
class NlsRoutesManager { /* ... */ }
```

### RD2 - Import only via `package:`

Use `package:` imports exclusively, never relative imports (`../..`). Import order within
`package:` imports is alphabetical and enforced by the linter (`directives_ordering`) - do not
hand-group imports by category.

### RD3 - Expose a barrel file, hide `src/`

A package/app exposes a single barrel file (`lib/<name>.dart`) that `export`s its public API;
implementation lives under a private `src/` folder not imported directly by consumers.

## Types, immutability, equality

### RD4 - Make models immutable and `Equatable`

Data models are `const` constructible, hold `final` fields, and extend `Equatable` for value-based
comparison.

```dart
class Product extends Equatable {
  final int id;
  final String name;

  const Product({required this.id, required this.name});

  @override
  List<Object?> get props => [id, name];
}
```

### RD5 - Keep `copyWith` parameters nullable

`copyWith` parameters are nullable and default to the current value. Because a nullable parameter
cannot distinguish "keep the current value" from "set it to `null`", expose a dedicated method
when a field must actually be cleared:

```dart
// copyWith cannot clear `product`: `product ?? this.product` keeps the old value.
SessionState copyWith({Product? product}) =>
    SessionState(product: product ?? this.product);

// A dedicated method expresses the "clear" intent explicitly.
SessionState clearProduct() => SessionState(product: null);
```

### RD6 - Use `enum` for closed sets

Use an `enum` for any closed set of values instead of `String`/`int` constants.

### RD7 - Implement `toString()` on debuggable models

Implement `toString()` on models that can appear in logs or debug output.

## Null-safety and error handling

### RD8 - Return a `Result` instead of throwing

Fallible operations that can naturally fail (not a programming error) return a `Result` type from
`act_dart_result` (`Result`, `ResultWithStatus`, `ResultWithRequiredValue`) rather than throwing.

```dart
Future<ResultWithRequiredValue<RuntimeErrorCode, List<Product>>> getProducts() async =>
    _pharmacyService.getAvailableProducts();
```

### RD9 - List `ok` first in status enums

A status enum used with a `Result` lists `ok` as its first value, so the enum's default (index 0)
is already a valid, successful state:

```dart
enum RuntimeErrorCode { ok, timeout, hardwareUnavailable }
```

### RD10 - Use `late final` for deferred initialization

Use `late final` for fields whose value is known only after `initLifeCycle` (or an equivalent
async init step), not for values that could be computed eagerly.

### RD11 - Reserve exceptions for the throwing boundary

Errors are propagated as `Result`, never through exceptions used for control flow. `try`/`catch`
is reserved for the boundary where a third-party library or SDK actually throws: catch narrowly,
log, and convert to a `Result`/status there. An explicit `throw` is acceptable only to fail fast
during development (an unrecoverable invariant violation), not as an error channel for expected
failure cases.

### RD12 - Compose `Result`s with early returns

Compose chains of `Result`-returning calls with early returns on failure rather than nested `if`s:

```dart
final defaultProductIdResult = _sessionService.getDefaultProductId();
if (defaultProductIdResult.isError) {
  return ResultWithStatus(status: defaultProductIdResult.status);
}
```

## Async, streams, resources

### RD13 - Never use `async void`

Never use `async void`, except when a signature must literally match `VoidCallback` (e.g.
`onPressed: () async { ... }`). Everywhere else, return `Future<void>` so the caller can `await`
and handle failures:

```dart
// Avoid - the caller can neither await nor catch a failure.
void loadData() async { await repo.fetch(); }

// Prefer
Future<void> loadData() async { await repo.fetch(); }
```

### RD14 - Use `ValueKeeperWithAndOnStream` for value + stream

Use `ValueKeeperWithAndOnStream` (or an equivalent `act_dart_value_keeper` type) when a value
needs both a current-value getter and a change stream, instead of hand-rolling a
`StreamController` plus a separate field.

### RD15 - Choose stream cardinality deliberately

Choose stream cardinality deliberately: a single-subscription stream for a one-consumer pipeline
(e.g. a service feeding one BLoC), a broadcast stream when several independent listeners must
observe the same events.

## Documentation and generated code

### RD16 - Document every class and member, private ones included

Every class, method, getter, setter and field carries a `///` doc comment - **including private
(`_foo`) members and `@protected` ones**. A private helper still needs to state what it does and
why; visibility does not exempt it from documentation.

```dart
/// Resolve the runtime status code for [productId] into a typed [Product].
///
/// Returns [RuntimeErrorCode.ok] with the product when the id is known.
Future<ResultWithStatus<RuntimeErrorCode, Product>> _resolveProduct(int productId) async {
  // ...
}
```

### RD17 - Document with an imperative verb

Document public APIs with `///` doc comments starting with an imperative verb (`Manager for...`,
`Get the...`), following [Effective Dart](https://dart.dev/effective-dart/documentation).

### RD18 - Reuse doc blocks with `{@template}`/`{@macro}`

Use `{@template <id>}...{@endtemplate}` for a doc block reused verbatim, and `{@macro <id>}` at
each reuse site, instead of copy-pasting the same comment:

```dart
/// {@macro act_life_cycle.MixinWithLifeCycle.initLifeCycle}
@override
Future<void> initLifeCycle() async { /* ... */ }
```

### RD19 - Keep generated code out of version control

All generated code (localization, FFI bindings, protobuf/gRPC stubs, JSON serialization, ...) is
emitted under a dedicated `generated/` folder, is never hand-edited, and is excluded from version
control (`.gitignore`).

## Widgets and composition

### RFL1 - Default to `StatelessWidget`

Default to `StatelessWidget`; reach for `StatefulWidget` only when the widget owns mutable,
non-BLoC state (animation controllers, text controllers, focus nodes).

### RFL2 - Extract sub-widgets, not builder methods

Extract a sub-widget rather than a private method returning `Widget`:

```dart
// Avoid - re-executed and re-allocated on every parent rebuild, no isolation.
Widget _buildHeader() => Text(title);

// Prefer - independently rebuildable, can be const, its own inspector node.
class _Header extends StatelessWidget {
  const _Header({required this.title});
  final String title;

  @override
  Widget build(BuildContext context) => Text(title);
}
```

### RFL3 - Split a `BlocProvider` entry widget from its `_View`

When a widget wraps a `BlocProvider`, split the public entry widget from a private
`_<WidgetName>View` that reads the BLoC and does the actual rendering:

```dart
class HmiWidget extends StatelessWidget {
  const HmiWidget({super.key});

  @override
  Widget build(BuildContext context) => BlocProvider(
    create: (_) => HmiBloc(...),
    child: const _HmiWidgetView(),
  );
}

class _HmiWidgetView extends StatelessWidget {
  const _HmiWidgetView();
  // ...
}
```

### RFL4 - Pass a `Key` only when identity matters

Pass a `Key` when a widget's identity must survive reordering or a rebuild that would otherwise
recreate its state (list items, `IndexedStack` children, `AnimatedList`). Declaring `super.key` on
public widget constructors is already enforced by the linter (`use_key_in_widget_constructors`) -
this rule is about when to actually pass one.

### RFL5 - Initialize `ScreenUtilInit` once, use `.w`/`.h`/`.sp` in widgets

For responsive layouts, initialize `ScreenUtilInit` once at the app root (see `MainAppUi`) and use
its `.w`/`.h`/`.sp` extensions (or `LayoutBuilder` for structural breakpoints) in widgets; do not
hardcode logical pixel sizes.

## State management (BLoC)

### RFL6 - Do not cache `InheritedWidget` lookups

Do not cache the result of an `InheritedWidget` lookup (`Theme.of`, `MediaQuery.of`, `Tr.of`,
`context.read<T>()`) in a field or closure that outlives the current `build`; re-read it on every
build so changes (theme, locale, screen size) are picked up.

### RFL7 - Extend the ACT BLoC/state/event base classes

BLoCs extend `BlocForMixin<S>` from `act_flutter_utility` (not the raw `Bloc<Event, S>`), states
extend `BlocStateForMixin<S>`, events extend `BlocEventForMixin`.

### RFL8 - Model events as a sealed hierarchy

Model events as a sealed hierarchy of small, intention-revealing classes (one class per
user/system action) rather than a single event carrying a discriminant field:

```dart
sealed class HmiEvent extends BlocEventForMixin {
  const HmiEvent();
}

class ButtonPressedEvent extends HmiEvent {
  const ButtonPressedEvent({required this.keyId});
  final KeyId keyId;

  @override
  List<Object?> get props => [...super.props, keyId];
}
```

### RFL9 - Initialize state with a named `init` constructor

State is immutable, exposes a **named `init` constructor** (`const` when possible) for its
initial value, and a `copyWith` for updates. Prefer `init` over a `factory .initial()` - it
matches this codebase's current convention:

```dart
class MainAppState extends BlocStateForMixin<MainAppState> {
  const MainAppState.init() : wantedLocale = null;
  // ...
}
```

### RFL10 - Register events via `registerMixinEvents`

Register event handlers by overriding `registerMixinEvents()` and calling `super` first (the base
class calls it from its constructor):

```dart
@override
void registerMixinEvents() {
  super.registerMixinEvents();
  on<ButtonPressedEvent>(_onButtonPressedEvent);
}
```

### RFL11 - Initialize a BLoC asynchronously with `MixinAsyncInitBloc`

For asynchronous initialization of a BLoC, mix in `MixinAsyncInitBloc<S>` and override
`initAsyncBloc({required emit})` (calling `super.initAsyncBloc()` first) instead of doing async
work in the constructor. Re-`add(const AsyncInitEvent())` to retry initialization if needed.

### RFL12 - Bridge external streams into `add(event)`

To bridge an external stream (a service or manager) into a BLoC, subscribe inside
`registerMixinEvents()` and translate each emission into `add(SomeEvent(...))` - never mutate
state directly from the listener callback. Track the subscription and cancel it in
`disposeLifeCycle`/`close`:

```dart
@override
void registerMixinEvents() {
  super.registerMixinEvents();
  on<InitHmiEvent>(_onInitHmiEvent);
  _subscriptions.add(_keysService.stream.listen((e) => add(SomeEvent(e))));
  add(const InitHmiEvent());
}
```

### RFL13 - Factor cross-cutting behaviour into a mixin pair

Factor a cross-cutting BLoC behaviour into a pair of mixins - one for the event-handling side, one
for the state side - and compose them onto the concrete BLoC/state with `with`. Each mixin
registers its own events via `registerMixinEvents` and calls `super`:

```dart
mixin MixinPageWithOverlay<S extends BlocStateForMixin<S>> on BlocForMixin<S> {
  @override
  void registerMixinEvents() {
    super.registerMixinEvents();
    on<OverlayShownEvent>(_onOverlayShown);
  }
}

mixin MixinPageWithOverlayState<S extends MixinPageWithOverlayState<S>>
    on BlocStateForMixin<S> {
  bool get overlayDisplayed;
}

class MyPageBloc extends BlocForMixin<MyPageState> with MixinPageWithOverlay<MyPageState> {
  MyPageBloc() : super(const MyPageState.init());
}
```

### RFL14 - Keep `createState()` free of logic

Keep `createState()` a one-line return of the `State` subclass; no logic (data fetching,
computation) belongs there.

## Application architecture

### RFL15 - Implement the three-phase manager lifecycle

Every manager extends `AbsWithLifeCycle` and implements the three-phase lifecycle:
`initLifeCycle()` (create services, load resources), `initAfterView(BuildContext)` (rare, needs a
`BuildContext`), `disposeLifeCycle()` (release resources). Always call `super` first/last as
appropriate.

### RFL16 - Define a manager's builder in the same file

Define a manager's builder (`class XxxBuilder extends AbsLifeCycleFactory<Xxx>`) in the same file
as the manager, right above it.

### RFL17 - Declare init ordering with `dependsOn()`

Declare inter-manager init ordering by overriding `dependsOn()` on the builder rather than relying
on registration order alone:

```dart
class PharmacyManagerBuilder extends AbsLifeCycleFactory<PharmacyManager> {
  const PharmacyManagerBuilder() : super(PharmacyManager.new);

  @override
  Iterable<Type> dependsOn() => [LoggerManager, RuntimeManager];
}
```

### RFL18 - Decompose a manager into services

Decompose a manager into **services** (one per sub-domain) rather than growing a single class.
Services extend a domain-specific abstract base (itself extending `AbsWithLifeCycle`) and receive
a sub-logger from their owning manager. Other managers reach a service through the owning
manager's DI-registered instance, not by registering the service itself:

```dart
class RuntimeManager extends AbsWithLifeCycle {
  late final RuntimeServicePharmacy pharmacyService;
  // ...
}

// Elsewhere, another manager reaches the service through RuntimeManager:
final pharmacyService = globalGetIt().get<RuntimeManager>().pharmacyService;
```

### RFL19 - Retrieve managers via `globalGetIt()`

Retrieve any manager via `globalGetIt().get<ManagerType>()`; never call `GetIt.instance` directly.

### RFL20 - Start the app with `AbsUiGlobalManager` + `runActApp()`

The app's global manager extends `AbsUiGlobalManager`, registers every manager inside
`registerManagers()` using `registerManagerAsync<T>(const XxxBuilder())` **in dependency order**,
and is started from `main()` with a single call:

```dart
class NeedlelessGlobalManager extends AbsUiGlobalManager {
  @override
  Future<void> registerManagers() async {
    registerManagerAsync<NlsConfigManager>(const NlsConfigBuilder());
    registerManagerAsync<LoggerManager>(ExtDefaultLoggerBuilder<NlsConfigManager>());
    // ... one registerManagerAsync per manager, dependencies first
  }
}

Future<void> main() async => NeedlelessGlobalManager.instance.runActApp(const MainAppUi());
```

### RFL21 - Organize `lib/` by concern

Organize `lib/` by concern, with these canonical top-level folders (omit unused ones):

```text
lib/
├── main.dart
├── constants/   # Theme, assets, business constants
├── generated/   # Auto-generated - do not edit (see RD19)
├── l10n/        # ARB translation source files
├── managers/    # Managers + their builders + services
├── models/      # Data models (Equatable)
├── types/       # Enums, exceptions, type aliases
└── ui/
    ├── main_app/  # MaterialApp root widget
    ├── pages/     # Page widgets, organized by feature
    └── widgets/   # Reusable widgets
```

### RFL22 - Colocate a BLoC's files with its widget

A widget built around a BLoC keeps its BLoC, event, state and widget files colocated in the same
feature folder: `<feature>_bloc.dart`, `<feature>_event.dart`, `<feature>_state.dart`,
`<feature>_widget.dart` (or `_page.dart`/`_ui.dart` for a page).

### RFL23 - Use consistent role suffixes for artifact names

Use these role suffixes consistently for artifact names (casing itself follows standard
Dart/Flutter rules):

| Role            | Suffix                                       | Example                 |
| --------------- | -------------------------------------------- | ----------------------- |
| Manager builder | `<Manager>Builder`                           | `RuntimeManagerBuilder` |
| Service         | `<Domain>Service`                            | `RuntimeServiceCore`    |
| BLoC            | `<Feature>Bloc`                              | `HmiBloc`               |
| State           | `<Feature>State`                             | `HmiState`              |
| Event           | `<Action>Event`                              | `ButtonPressedEvent`    |
| Page            | `<Feature>Page` in `<feature>_page.dart`     | `SettingsPage`          |
| Widget          | `<Feature>Widget` in `<feature>_widget.dart` | `HmiWidget`             |

### RFL39 - Show a fatal error page via `UiFatalErrorManager`

Register `UiFatalErrorManager` (from `act_global_manager`) with a page builder so `runActApp()`
shows that page instead of crashing on a startup failure or an uncaught error. Register it right
after the logger it depends on, so it also covers the managers that follow:

```dart
registerManagerAsync<UiFatalErrorManager>(
  UiFatalErrorBuilder((error) => FatalErrorPage(error: error)),
);
```

The page runs before any other manager is ready, so keep it self-contained (its own `MaterialApp`,
no routes or translations). Trigger it manually with `displayFatalErrorPage(error)`.

## Configuration and logging

### RFL24 - Never hardcode configuration values

No hardcoded configuration values in the codebase. Configuration extends
`AbsUsualConfigManager` and exposes each value as a `const ConfigVar<T>("path.to.value")`:

```dart
class NlsConfigManager extends AbsUsualConfigManager with MixinLocaleConfig {
  final serverHostname = const ConfigVar<String>("server.hostname");
}
```

### RFL25 - Register `LoggerManager` right after config

Register `LoggerManager` immediately after the config manager in `registerManagers()` - every
manager registered afterwards can depend on it.

### RFL26 - One `LogsHelper` per class, built into a hierarchy

Each class that logs owns one `LogsHelper`, built from a `static const _loggerCategory` constant.
Build a logger hierarchy with `createASubLogsHelper(...)` when a manager hands a sub-logger to an
owned service, instead of giving every service a flat top-level logger.

### RFL27 - Respect log level intent

Respect log level intent: `d` for development-only detail, `i` for normal lifecycle events, `w`
for recoverable/unexpected situations, `e` for failures. Never use `print`.

## Routing

### RFL28 - Declare routes as `enum with MixinRoute`

Routes are declared as an `enum with MixinRoute`, carrying `parent`, `transition`, and
`screenOrientation` per route:

```dart
enum NlsRoute with MixinRoute {
  error,
  mainMenu,
  settings;
  // parent / transition / screenOrientation fields + constructor
}
```

### RFL29 - Map routes to pages in a `RoutesNameHelper`

Map routes to pages in a `RoutesNameHelper extends AbstractRoutesHelper<T>`, registering each
route with `onPage(route, _createXxxPage)` in the constructor.

### RFL30 - Retrieve route extras with `checkAndCastExtra<T>`

Retrieve route extras with `checkAndCastExtra<T>(state)` instead of an unchecked cast, so a
wrong/missing extra fails with a clear error:

```dart
final pageInfo = checkAndCastExtra<PrimingInjectionPageInfo>(state);
```

### RFL31 - Navigate only through the routes manager

Navigate exclusively through the app's routes manager
(`globalGetIt().get<XxxRoutesManager>().push(...)`/`.pop()`), never by constructing a route/path
manually and never by calling Flutter's `Navigator` directly (`Navigator.of(context)`,
`Navigator.push(...)`, `Navigator.pop(...)`). The routes manager is the single place that knows
how routes translate to pages, transitions, and orientation - bypassing it breaks that guarantee.

```dart
// Avoid - bypasses the routes manager.
Navigator.of(context).pushNamed('/settings');

// Prefer
globalGetIt().get<NlsRoutesManager>().push(NlsRoute.settings);
```

## Internationalization

### RFL32 - Keep user-facing strings in ARB files

All user-facing strings live in ARB files (`l10n/intl_<locale>.arb`); no literal user-visible
string in Dart code.

### RFL33 - Register all four localization delegates

Register the four localization delegates (`Tr.delegate`, `GlobalMaterialLocalizations.delegate`,
`GlobalWidgetsLocalizations.delegate`, `GlobalCupertinoLocalizations.delegate`) and
`supportedLocales` on the root `MaterialApp`.

## Assets and themes

### RFL34 - Define the theme with a namespace-aliased import

Define the theme in `constants/theme_constants.dart` (or similar) as an `ActThemeModel`, imported
elsewhere with a namespace alias rather than star-imported, to keep call sites explicit about
origin:

```dart
import 'package:needleless_ui/constants/theme_constants.dart' as theme_constants;

final theme = theme_constants.appTheme;
```

## Security

### RFL35 - Route secrets through ACT auth/config packages, never local storage

Secrets, tokens, and other sensitive data never go through `SharedPreferences` or
`act_local_storage_manager`. Configuration values use `AbsUsualConfigManager` + `ConfigVar<T>`
(RFL24); authentication material uses `act_shared_auth` + `act_shared_auth_local_storage`,
`act_amplify_cognito`, `act_oauth2_core`/`act_oauth2_google`, or `act_jwt_utilities` as
appropriate. Call the relevant `clear()` explicitly on logout.

## Performance

### RFL36 - Cache values outside `build()`

Compute or cache values outside `build()` (e.g. in `initState`/BLoC state), not inline inside the
widget tree on every rebuild.

### RFL37 - Wrap expensive repaint subtrees in `RepaintBoundary`

Wrap a subtree that repaints independently and expensively (custom painters, animations) in a
`RepaintBoundary` so it does not force siblings to repaint.

### RFL38 - Use `ListView.builder` for non-trivial lists

Use `ListView.builder` (or another lazy builder) for any list whose length is not small and
fixed; never eagerly build a `Column` of many children inside a scroll view.

## Sources

- [Effective Dart](https://dart.dev/effective-dart)
- [Flutter documentation](https://docs.flutter.dev/)
- [flutter_bloc documentation](https://bloclibrary.dev/)
- [OWASP Mobile Application Security Verification Standard (MASVS)](https://mas.owasp.org/MASVS/)
- ACT-Flutter-Packages `analysis_options.yaml` and `act_flutter_utility`, `act_life_cycle`,
  `act_global_manager`, `act_dart_result`, `act_router_manager`, `act_config_manager`,
  `act_themes_manager` source.
