# Building Folio for PS5

This fork enables the Folio and FolioCore runtime modules for PS5. The editor module
remains a host module. No SDK headers, libraries, tools or console binaries are
distributed with the plugin; use your own SDK and an engine with the PS5 platform.

From the plugin root, with CMake, Ninja and Git on PATH:

```powershell
$env:SCE_PROSPERO_SDK_DIR = 'X:/SDK/Prospero'
cmake -S cmake/ue -B build/ue-PS5-Release -G Ninja `
  -DCMAKE_TOOLCHAIN_FILE="$PWD/cmake/ps5-toolchain.cmake" `
  -DCMAKE_BUILD_TYPE=Release -DFOLIO_UE_PLATFORM=PS5 `
  -DFOLIO_ENGINE_DIR='X:/UnrealEngine/Engine'
cmake --build build/ue-PS5-Release --parallel 12
```

The engine must supply the matching FreeType library. The default follows the
platform extension's `Platforms/PS5/Source/ThirdParty/FreeType2` layout. Override
`FOLIO_FREETYPE_LIB` if your engine stores it elsewhere. HarfBuzz uses the engine's
module as it does on desktop.

CMake stages the pinned, patched RmlUi and msdfgen libraries under
`Source/ThirdParty/*/lib/PS5`. They are ignored by Git. To stage outside the plugin,
set `FOLIO_STAGE_ROOT`; copy the resulting PS5 library directories, including the
`RMLUI_PATCHES` stamp, into the plugin after comparing the generated headers with
the headers used by the host build. Do not mix dependency revisions between targets.

## Allocator alignment

The CMake UE build sets `-fnew-alignment=16` for PS5, matching Unreal's Sony
toolchain and global `operator new`. Omitting it allowed the compiler to generate
32-byte aligned AVX stores to an allocation only guaranteed to be 16-byte aligned.
This caused an early SIGBUS in `Rml::Detail::InitializeElementInstancerPools`.
The corrected object code uses unaligned stores at that location. Do not solve
this by disabling AVX or changing the shared engine allocator.

## Validation

Tested with a UE 5.8.3 PS5 platform: RmlUi/msdfgen cross compilation, Development
link, full project cook, and inspection of the corrected initialization code.
The project editor modules also compiled with Unity and with the game module
compiled separately. The corrected PS5 binary still requires a console run;
successful compilation is not runtime validation.
