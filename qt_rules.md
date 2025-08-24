Version: 1.0
Date Created: 6:17 PM 24/08/2025
Last Edit Date: 6:18 PM 24/08/2025
Owner: "Jarryd Adaens"

# Windsurf Rules — Qt C++ (Windows, OpenRGB Plugin)

## Scope and priority
- Target: Qt Widgets–based C++ on Windows 11, MSVC toolchain, OpenRGB plugin code.
- Applies to: Source, tests, build files (CMake/qmake).
- Precedence: Project rules > Global rules > Tool defaults.

## Build system
- Prefer Qt 6 + CMake. Use `Qt6::` targets and `AUTOMOC`, `AUTORCC`, `AUTOUIC`.
- If the project is on Qt 5 or upstream requires qmake, keep a thin `.pro` that mirrors CMake options.
- Example CMake (library/plugin):
  ```cmake
  cmake_minimum_required(VERSION 3.21)
  project(OpenRgbRogBattery LANGUAGES CXX)
  set(CMAKE_CXX_STANDARD 20)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)

  find_package(Qt6 REQUIRED COMPONENTS Core Widgets)
  add_library(rog_battery_plugin SHARED
      src/Plugin.cpp
      src/PluginUi.cpp
      include/Plugin.hpp
      resources/resources.qrc)
  target_link_libraries(rog_battery_plugin PRIVATE Qt6::Core Qt6::Widgets)
  target_include_directories(rog_battery_plugin PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
  target_compile_definitions(rog_battery_plugin PRIVATE QT_NO_KEYWORDS)
  
  # Windows-specific settings
  if(WIN32)
      set_target_properties(rog_battery_plugin PROPERTIES
          CXX_EXTENSIONS OFF
          COMPILE_OPTIONS "/permissive-;/Zc:__cplusplus")
  endif()
  ```
- Windows defaults: MSVC v143, Ninja generator, `/permissive-` `/Zc:__cplusplus`.

## Tooling

- **Formatting**: Use Qt's `_clang-format` as base. If absent, approximate: K&R, 4 spaces, 100 col width, attached braces except for function/class bodies.
- **Static analysis**: Enable `/W4` `/WX-` and clang-tidy (checks: `-*,cppcoreguidelines-*,modernize-*,-modernize-use-trailing-return-type,performance-*`).
- **IDEs**: Qt Creator or VS + "Qt VS Tools" are both allowed.

## Language and API rules (Qt-conformant)

- No exceptions. No RTTI (`dynamic_cast`, `typeid`) in production code.
- Prefer C++ casts: `static_cast`, `const_cast`, `reinterpret_cast`.
- Each `QObject` subclass must include `Q_OBJECT`.
- Public headers: stable API, verbose parameter names, use `#pragma once`; avoid global objects with non-trivial constructors.

## Ownership and lifetime

- Use parent–child ownership for `QObject` trees. Never delete children manually; prefer `deleteLater()` across threads.
- Do not combine smart-pointer ownership with `QObject` parenting. For non-`QObject` resources, use RAII (`std::unique_ptr`/`std::shared_ptr`) as usual.

## Signals/slots

- Use function-pointer syntax, not `SIGNAL`/`SLOT` strings:
  ```cpp
  connect(source, &Source::updated, this, &Target::onUpdated);
  ```
- For overloads, use `qOverload` or `static_cast`.
- Use `Qt::UniqueConnection` where idempotence is required.
- Cross-thread: rely on `Qt::AutoConnection` semantics; it becomes queued when crossing threads.

## Threading

- UI lives on the GUI thread. Offload polling or I/O to worker `QObjects` in `QThreads`.
- Change thread affinity via `moveToThread`. Don't access a `QObject` from a foreign thread; signal it instead.
- Use `QMetaObject::invokeMethod` with `Qt::QueuedConnection` for cross-thread calls.

## Containers

- Qt 6: `QVector` is an alias of `QList`. Use `QList` by default; prefer STL containers in pure algorithmic code where appropriate.
- Use `QStringList` for string collections, `QHash` for key-value pairs, `QSet` for unique collections.

## Strings

- Store and pass strings as `QString` or `QStringView` for non-owning parameters.
- Prefer compile-time string creation: `u"Text"_s` (Qt::Literals) or `QStringLiteral("Text")`.
- Use translator macros for UI text: `tr("Battery")`.
- For formatting, prefer `QString::arg()` over `sprintf`-style functions.

## Formatting and naming

- Indent 4 spaces. Max line 100.
- Attached braces for control flow; brace on new line for function/class bodies only.
- Types and classes: `UpperCamelCase`. Methods/variables: `lowerCamelCase`. Acronyms CamelCased (`QXmlStreamReader`).
- One declaration per line. Avoid abbreviations.
- Private members: prefix with `m_` (e.g., `m_batteryLevel`).

## Build artifacts and defines

- Define `QT_NO_KEYWORDS` to avoid `emit`/`slots`/`signals` macro collisions.
- Public headers include form: `#include <QtCore/qstring.h>` when shipping public API; otherwise normal module includes are fine.
- Use `Q_DECLARE_METATYPE` for custom types used in signals/slots or `QVariant`.

## Plugin-specific (OpenRGB)

- Don't block the UI with battery polling. Use `QTimer` or a worker object in a `QThread`.
- Emit a debounced signal for state changes to avoid spamming RGB updates.
- Keep any Windows API calls behind a narrow adapter class with clear lifetime and thread ownership.

## Testing

- Use `QtTest` for unit tests. Avoid GUI dependencies in tests; mock `QObject` emitters.
- Use `QSignalSpy` for testing signal emissions.
- Test files should follow the pattern `tst_classname.cpp`.

## File layout

```
/include/...   public headers
/src/...       implementation  
/resources/    .qrc, icons
/tests/        QtTest-based tests
/cmake/        CMake modules (if any)
```

## Editor configs

- Add `.editorconfig` with 4 spaces and 100 col.
- Add project `.clang-format` aligned to Qt's presets if you can't vendor Qt's `_clang-format`.

## Examples

### Connect with overload resolution
```cpp
connect(combo, qOverload<int>(&QComboBox::currentIndexChanged),
        this, &Controller::onIndexChanged);
```

### QObject lifetime
```cpp
auto worker = new Worker;              // no parent; will be moved
worker->moveToThread(thread);
connect(thread, &QThread::finished, worker, &QObject::deleteLater);
```

### Efficient strings
```cpp
using namespace Qt::Literals::StringLiterals;
label->setText(u"Charging"_s);
```

### Custom type registration
```cpp
Q_DECLARE_METATYPE(BatteryState)

// In implementation:
qRegisterMetaType<BatteryState>("BatteryState");
```

## Enforcement

- CI checks: build on MSVC Release+Debug, run clang-tidy, run QtTest.
- PRs touching public headers must include usage examples and rationale.
- Use `Q_DISABLE_COPY_MOVE` for classes that shouldn't be copied/moved.
