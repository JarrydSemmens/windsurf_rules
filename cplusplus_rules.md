Version: 1.0
Date Created: 5:31 PM 24/08/2025
Last Edit Date: 5:45 PM 24/08/2025
Owner: "Jarryd Adaens"

# Unreal C++ Coding Rules (Workspace)

> Target: UE 5.x C++ projects. Align with Epic’s standard. 

## Scope
- Applies to all C++ in this workspace (engine modules, game modules, plugins).
- Goal: readable, consistent UE-idiomatic code that compiles fast and plays well with UHT/UBT.

## Language & Types
- Prefer UE fixed-width types where size matters: `int32`, `uint8`, `int64`, etc. Use plain `int` only when width is irrelevant.
- Use `bool` for booleans. Do not assume size of `bool`.
- Use `TCHAR` for character when interacting with UE text APIs.
- Use `TEXT("...")` for string literals intended for UE types (`FString`, `FName`).  
- Standard Library: allowed when superior. Avoid mixing UE and STL idioms in the same API. Keep consistency within an API surface.

## Naming
- PascalCase for types, methods, and most identifiers.
- UE type prefixes:
  - `U` = `UObject` derived (`UActorComponent`)
  - `A` = `AActor` derived
  - `S` = Slate widget
  - `F` = struct or non-UObject class
  - `I` = pure interface
  - `T` = template class
  - `E` = enum type
- Variables:
  - Booleans start with `b` (`bIsVisible`, `bPendingDestroy`).
  - Use descriptive nouns for data. Avoid cryptic acronyms.
  - One declaration per line.
- Functions:
  - Boolean returns ask a question: `IsVisible()`, `ShouldClearBuffer()`.
  - Procedures use strong verb + object: `ResetCache()`, `ApplySettings()`.
  - Prefer explicit, descriptive names over abbreviations.
- Parameters:
  - Use `camelCase`.
  - If a parameter is passed by non-const reference and is written to, prefix with `Out`: `OutResult`. If also boolean, `bOutSucceeded`.
  - Optional team convention: input parameters may use the `in_` prefix (e.g., `int32 in_Count`). Note: this diverges from Epic's stock style; use consistently across your codebase if adopted.
- Templates:
  - Template type parameters may use `In` prefix to disambiguate (`template<typename InElementType>` then `using ElementType = InElementType;`).
- Macros: ALL_CAPS with underscores, prefix with your project/module tag (e.g., `PROJECT_`, `YOURMODULE_`). Avoid `UE_`, which is reserved for Epic/engine.

## Class Organization
- Public interface first, then protected, then private.
- Enforce encapsulation. Keep members private unless part of the intended API. Provide protected accessors if needed.
- `final` for types not intended for inheritance.

## Namespaces
- Do **not** use namespaces for `UCLASS`, `USTRUCT`, `UENUM` declarations (UHT limitation).
- Non-UObject public APIs may live in `YourProject::YourDomain::...`. Internal-only details may use `YourProject::YourDomain::Private::...`.
- No `using` in global scope. `using` is OK within a namespace or function body.

## Files & Includes
- File names match their primary type names, including UE prefixes where applicable: e.g., `USceneComponent.h/.cpp` for `USceneComponent`.
- All headers use `#pragma once`.
- IWYU mindset:
  - Forward-declare where possible in headers; include in `.cpp`.
  - Include only what you use; avoid umbrella headers (don’t include `Core.h`).
  - Don’t rely on transitive includes. Include everything you use directly.
- Order:
  - `.cpp`: include its own header first, then other includes.
  - `.h`: `#include "ThisHeader.generated.h"` is the **last** include.
- Modules: put public headers in `Public/`, internals in `Private/`.

## Formatting
- Tabs for indentation. Tab width 4. Spaces only for alignment after non-tab characters.
- Allman braces: opening `{` on its own line for functions, control blocks, and class/structs.
- Always use braces, even for single-statement blocks.
- One statement per line.
- Pointer/reference spacing: `Type* Ptr`, `Type& Ref` (one space to the right of `*` or `&`).
- Leave a single blank line at end of file.

## Control Flow
- `if/else`: braces on all branches. `else if` aligned with `if`.
- `switch`: every case breaks/returns or has an explicit `// falls through`. Always provide `default:`.
- Minimize dependency distance. Initialize close to use. Split large functions into well-named helpers.

## Strings & Text
- Wrap UE string literals with `TEXT()`.
- Prefer named constants over magic literals in calls. Example: `Trigger(ObjectName, CooldownSeconds, bCanInterrupt)`.

## Logging & Asserts
- Use `UE_LOG(Category, Verbosity, TEXT("..."))` for logging.
- Use standard UE asserts (`check`, `ensureMsgf`) per project policy. Keep messages actionable.

## UHT/Reflection Requirements
- Use the correct UE prefixes (`U`, `A`, `F`, etc.) so UHT can recognize reflected types.
- Reflection macros:
  - Class/struct/enum declarations use `UCLASS`, `USTRUCT`, `UENUM`.
  - Members exposed to UE use `UPROPERTY` with metadata where needed.
  - Functions exposed to UE/Blueprint use `UFUNCTION` and, if needed, `UPARAM`.
- Generated header include (`*.generated.h`) must be last in headers when working with Unreal Engine code.

## Comments & Docs
- U.S. English for code and comments.
- Keep comments concise and accurate. Update when behavior changes.
- Place variable documentation above the declaration. Optional blank lines to group variables.

## Examples

```cpp
// Example UObject class
// Copyright Your Studio. All Rights Reserved.
#pragma once

#include "Containers/Array.h"
#include "Components/ActorComponent.h"
#include "BatteryStatusComponent.generated.h"

UCLASS(ClassGroup=(Input), meta=(BlueprintSpawnableComponent))
class YOURMODULE_API UBatteryStatusComponent : public UActorComponent
{
	GENERATED_BODY()

public:
	UBatteryStatusComponent();

	// Returns true if the battery is considered low.
	UFUNCTION(BlueprintPure, Category="Battery")
	bool IsBatteryLow() const;

	// Writes threshold value. Out param is ref and will be set.
	void GetLowBatteryThreshold(int32& OutThreshold) const;

    int32 GetSize(int32 in_sizeOfExample) const;

private:
	// Percent 0..100
	UPROPERTY(EditAnywhere, Category="Battery")
	int32 BatteryPercent = 100;

	// Threshold below which IsBatteryLow() returns true
	UPROPERTY(EditAnywhere, Category="Battery")
	int32 LowThreshold = 20;

	// State flag, UE boolean prefix
	bool bIsCharging = false;

	// Example backing data used by GetSize()
	TArray<int32> Cache;
	int32 Mode = 0;
};

// Formatting and control flow
int32 UBatteryStatusComponent::GetSize(int32 in_sizeOfExample) const
{
	const bool bHasCache = Cache.Num() > in_sizeOfExample;
	if (bHasCache)
	{
		return Cache.Last();
	}

	switch (Mode)
	{
		case 0:
			// falls through
		case 1:
			return 1;
		default:
			return 0;
	}
}
