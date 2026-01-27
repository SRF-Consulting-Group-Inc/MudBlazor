# MudBlazor Converter Breaking Changes - Past Few Months

## Summary

**YES**, there were significant breaking changes to the converter system in MudBlazor recently. The major converter rework was introduced in **PR #12177** and released in **v9.0.0-preview.1** (December 20, 2025).

## Timeline

- **November 27, 2025**: PR #12177 "Converters: Rework conversion system" opened
- **December 3, 2025**: PR merged into dev branch
- **December 20, 2025**: Released in v9.0.0-preview.1
- **January 14, 2026**: Included in v9.0.0-preview.2

## What Changed?

### Overview

The converter system underwent a **comprehensive rework**, transitioning from a class-based `Converter` approach to a modern interface-based architecture. This represents one of the most significant breaking changes in MudBlazor's form component infrastructure.

### Key Breaking Changes

#### 1. **Interface-Based Architecture**

**Before (v8.x):**
- Class-based `Converter<T>` with `Set`/`Get` methods
- Direct parameter manipulation

**After (v9.x):**
- New interfaces: `IConverter<T>`, `IReversibleConverter<T>`, `ICultureAwareConverter<T>`
- Methods renamed: `Convert`/`ConvertBack` instead of `Set`/`Get`
- Covariant/contravariant type parameters for better type safety

#### 2. **Parameter Handling**

**Breaking:** You can no longer directly write to these parameters:
- `Format`
- `Converter`
- `Culture`
- `ToStringFunc`

These parameters are now read-only and must be set declaratively in the component markup, not programmatically.

#### 3. **Error Handling**

**New approach:**
- Structured exception handling with `ConversionException` carrying localization keys
- New `ConversionResult<T>` struct for non-throwing conversion attempts
- Better error messages and localization support

#### 4. **Converter Composition**

**New features:**
- `ConverterChain` for composing conversion pipelines
- Type dispatchers (`TypeDispatcher`, `ReversibleTypeDispatcher`) for polymorphic conversion
- Fluent API for building complex converters

#### 5. **Range Handling**

**Extracted utility:**
- Range string manipulation moved to dedicated `RangeUtility` class
- Canonical format: `[start;end]`

#### 6. **Component API Changes**

**MudFormComponent changes:**
- Updated to use `IConverter` with new error handling via `ConversionResult`
- New method: `GetDefaultConverter()`
- Methods renamed to async versions (from PR #12147)

**MudBaseInput changes:**
- Culture/format injection via delegates to culture-aware converters
- Removal of `TextUpdateSuppression` (PR #12306)
- Better integration with ParameterState

## Impact on Custom Converters

If you implemented custom converters, you **MUST** update them:

### Old Pattern (v8.x)
```csharp
public class MyConverter : Converter<int>
{
    public override int Get(string value)
    {
        return int.Parse(value);
    }
    
    public override string Set(int value)
    {
        return value.ToString();
    }
}
```

### New Pattern (v9.x)
```csharp
public class MyConverter : IReversibleConverter<int, string>
{
    public string Convert(int value)
    {
        return value.ToString();
    }
    
    public int ConvertBack(string value)
    {
        if (!int.TryParse(value, out var result))
            throw new ConversionException("Invalid number");
        return result;
    }
}
```

### Culture-Aware Converters

For culture-specific formatting:

```csharp
public class MyConverter : IReversibleConverter<int, string>, ICultureAwareConverter
{
    public CultureInfo Culture { get; set; }
    
    public string Convert(int value)
    {
        return value.ToString(Culture);
    }
    
    public int ConvertBack(string value)
    {
        return int.Parse(value, Culture);
    }
}
```

## Migration Checklist

- [ ] **Review all custom converters** - Update to implement new interfaces
- [ ] **Update method calls** - Change `Set`/`Get` to `Convert`/`ConvertBack`
- [ ] **Remove parameter mutations** - Don't write to `Format`, `Converter`, `Culture`, `ToStringFunc`
- [ ] **Update error handling** - Use `ConversionException` instead of generic exceptions
- [ ] **Test all form inputs** - Verify data binding, conversion, and validation still work
- [ ] **Check culture-specific formatting** - Ensure date/number formatting works correctly
- [ ] **Review validation logic** - Converter errors now integrate better with validation

## Components Affected

All form components that use converters are affected:
- `MudTextField<T>`
- `MudNumericField<T>`
- `MudDatePicker`
- `MudTimePicker`
- `MudSelect<T>`
- `MudAutocomplete<T>`
- `MudRangeInput<T>`
- Any component deriving from `MudFormComponent<T, U>`

## Additional Breaking Changes in v9.0.0-preview.1

The converter rework was part of a larger set of breaking changes in v9.0.0-preview.1:

1. **ParameterState refactoring** - Improved parameter handling across components
2. **MudFormComponent method renames** - Many methods now async
3. **MudBaseInput refactoring** - `TextUpdateSuppression` removed, better ParameterState integration
4. **MudGlobal cleanup** - Deprecated properties removed
5. **Popover changes** - Modal behavior changes
6. **Obsolete API removal** - All v8 deprecations removed

## Resources

- **PR #12177**: https://github.com/MudBlazor/MudBlazor/pull/12177
- **Release v9.0.0-preview.1**: https://github.com/MudBlazor/MudBlazor/releases/tag/v9.0.0-preview.1
- **Release v9.0.0-preview.2**: https://github.com/MudBlazor/MudBlazor/releases/tag/v9.0.0-preview.2
- **Converter Documentation**: https://www.mudblazor.com/features/converters
- **Changes**: 97 files changed, 5,359 additions, 2,351 deletions

## Recommendations

### For Production Applications (v8.x)
- **Stay on v8.x** until v9.0.0 stable is released
- Latest stable: **v8.15.0** (November 23, 2025)
- No converter breaking changes in v8.x

### For Early Adopters (v9.x preview)
- Review the full changelog before upgrading
- Test thoroughly in non-production environments
- Prepare for additional breaking changes before v9.0.0 stable
- Follow migration guide above for converters

### For New Projects
- Consider starting with v9.0.0-preview.2 if you want latest features
- Be aware you'll need to update to v9.0.0 stable when released
- Less migration work since you'll use the new APIs from the start

## Known Issues

After the converter rework, some issues were discovered and fixed:

- **Issue #11958** (October 2025): `ResetAsync()` not clearing `ConverterError` - Fixed in v9 development
- **PR #12365** (January 2026): Additional cleanup and `GetDefaultConverter()` improvements

## Conclusion

The converter system underwent a major, breaking rework in v9.0.0-preview.1 (released December 20, 2025). This change improves:
- **Type safety** with interface-based contracts
- **Composability** with converter chains
- **Error handling** with structured exceptions
- **Maintainability** with cleaner architecture

However, it requires code changes for anyone using custom converters or directly manipulating converter-related parameters.

## Questions?

If you have questions about migrating to the new converter system:
1. Check the [MudBlazor documentation](https://www.mudblazor.com/features/converters)
2. Review PR #12177 for implementation details
3. Ask in [MudBlazor Discussions](https://github.com/MudBlazor/MudBlazor/discussions)
4. Open an issue if you find bugs
