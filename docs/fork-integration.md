# Standalone fork integration

This branch merges `feature/optional-fork-features` (`909d566`) into
BrunoLevy/ImGuiColorTextEdit_geogram `master` (`062408b`). The original Bruno
branch is the merge's first parent; its history is retained without rebasing.
The incoming implementation includes goossens master `43a3759`, with its newer
layout, autocomplete and modular language registry.

Bruno's Emscripten cursor-member names (`is_main`, `main_index`), Apple macro
guard and transaction-parameter warning fixes are retained in the modern core.
ALT packaging is maintained on a separate branch.

## Geogram

`ENABLE_GEOGRAM` defaults to `ON` on this Bruno integration branch (the general
optional-features branch defaults to `OFF`). When enabled, the public CMake target exports
`__GEOGRAM__` to consumers; the library and its consumers must agree on this
option because it changes the public class layout.

Features from BrunoLevy/ImGuiColorTextEdit_geogram (`062408b`) are adapted to
the current document/visual-position architecture:

* `set_callback` and run (F5), save (F2), stop (Ctrl-C without selection),
  completion (Tab) and text-change events. Native autocomplete retains its
  special keys while active. Read-only mode does not request completion.
* Context under the pointer for application tooltips and member completion.
  Existing marker tooltips and upstream hover handlers take priority.
* Selection, insertion, cursor and find-window compatibility helpers.
* Optional find-window font and close-button label supplied by the host.

The library does **not** include Geogram headers, reference `GEO::icon_UTF8`,
or use `GEOGRAM_GFX_API`. ImGui remains its only external library dependency.
The application owns fonts, icons and action handlers. The application can use
the modern language registry (`LookupLanguage("Lua")`) instead of obsolete
`Language::Lua()`. DLL ownership is controlled by `TEXTEDITOR_API`, independently
of the application's `IMGUI_API` definition.

## BalazsJako audit

The full master/dev histories were examined, including the common ancestor
`0a88824` and the 2025 full rewrite in the main upstream. The four master
commits not reachable from goossens master (`2803948`, `4335e74`, `7454c1f`,
`ca2f9f1`) are warning fixes and updates to obsolete ImGui calls, not new
editor features. The main upstream already uses the current ImGui API.

The older editor's features have modern equivalents: error/breakpoint display
uses markers and line decorators; identifier descriptions use hover callbacks;
language keywords and custom tokenization use the modular language definitions;
UTF-8, undo/redo, palettes, read-only mode and indentation are already supported.
The old regex-based tokenizer is an implementation, not an additional user
feature. No second legacy editor or duplicate feature implementation is carried.

The historical dev commit `8cbe584` (2019-08-15) does have one distinct feature:
the choice between a short tab arrow and an arrow spanning the full tab width.
Upstream renders only a short arrow. This choice is retained under `__BALAZS__`,
enabled by `ENABLE_BALAZS` (default `OFF`). When enabled, the historical default
is a full-width arrow; `SetShowShortTabGlyphs(true)` selects a short arrow.
Tab layout, text contents and indentation are unaffected. Like the Geogram flag,
the option is propagated to consumers and must match the library build.

## Building

Provide an existing `imgui` / `imgui::imgui` target or an installed `imgui`
CMake package, then configure this project normally. Both must use the same
ImGui configuration. `BUILD_SHARED_LIBS` defaults to `ON`; `BUILD_TESTING`
defaults to `OFF`. For regression checks enable `BUILD_TESTING` and run CTest.
Installed consumers use `find_package(ImGuiColorTextEdit CONFIG REQUIRED)` and
link `ImGuiColorTextEdit::ImGuiColorTextEdit`. The installed target propagates
the dependency, include directories, C++17 requirement and compatibility flag.
