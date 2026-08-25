# Platform Blocks Forms — API reference

All components import from `@platform-blocks/ui` unless a subpath is noted. Every prop below is verified against `packages/ui/src`.

## Form (compound)

`Form` = `FormBase` + attached sub-components: `Form.Field`, `Form.Input`, `Form.Label`, `Form.Error`, `Form.Submit`. `FormBase` renders a `FormProvider` (React context + `useState`) around a plain `<View>` — there is no native `<form>` element or submit-on-enter plumbing.

### FormProps

| Prop | Type | Default | Description |
| --- | --- | --- | --- |
| `initialValues` | `Record<string, any>` | `{}` | Initial field values keyed by field name |
| `validationSchema` | `ValidationSchema` = `{ [field]: ValidationRule[] }` | `{}` | Declarative per-field rules, run through `validateValue` |
| `onSubmit` | `(values) => void \| Promise<void>` | — | Called with all values, only when validation produces no errors |
| `validate` | `(values) => Record<string, string> \| Promise<...>` | — | Custom whole-form validation; merged with schema errors on submit |
| `disabled` | `boolean` | `false` | Blocks submission; propagated to context |
| `validateOnChange` | `boolean` | `true` | Run the field's schema rules on every `setFieldValue` |
| `validateOnBlur` | `boolean` | `true` | Run the field's schema rules on blur (via `getFieldProps().onBlur`) |
| `children` | `ReactNode` | required | Form content |

### FormContextValue (`useFormContext()` / `useOptionalFormContext()`)

| Member | Type | Notes |
| --- | --- | --- |
| `values` | `Record<string, any>` | Current values |
| `errors` | `Record<string, string>` | Empty string = no error |
| `touched` | `Record<string, boolean>` | Set on blur; all schema fields set on submit |
| `disabled` / `isSubmitting` / `isValid` | `boolean` | `isValid` = every error string empty (true before any validation ran) |
| `validateField(name, value)` | `=> Promise<string \| null>` | Runs that field's schema rules |
| `setFieldValue(name, value)` | | Also validates when `validateOnChange` |
| `setFieldError(name, error)` | | Manual error injection (e.g. server errors) |
| `setFieldTouched(name, touched)` | | |
| `getFieldProps(name)` | `=> { value, onChangeText, onBlur, error, name }` | `value` falls back to `''`; `error` only when `touched[name]` — spread onto `Input`-family components |
| `submitForm()` | `=> Promise<void>` | Touch-all → validate → `onSubmit` if clean |
| `resetForm()` | | Back to `initialValues`, clears errors/touched |

`useFormContext()` throws outside `<Form>`; `useOptionalFormContext()` returns `null`.

### Form.Field (FormFieldProps)

| Prop | Type | Notes |
| --- | --- | --- |
| `name` | `string` | Injected into `Form.Input`/`Form.Label`/`Form.Error` children (child's own props win) |
| `dependsOn` | `Array<{ field, condition(value, formValues), action: 'show'\|'hide'\|'enable'\|'disable'\|'require' }>` | Conditional visibility/enablement/required based on other fields — implemented |
| `validateWhen`, `validation` | — | Present in the type but **not implemented** by `FormField`; use `validationSchema` |

Injection targets are matched by displayName/reference (`FormInput`, `FormError`, `FormLabel`); other children (including plain `Input`) pass through untouched — they get no value binding.

### Form.Input (FormInputProps)

`{ name?: string } & InputProps` (index signature). Inside a form context with a `name`, renders `<Input {...inputProps} {...getFieldProps(name)} />` — field props are spread **last**, so the form's `value`/`onChangeText`/`onBlur`/`error` override yours. Outside a context (or without `name`) it degrades to a plain `Input`. Forwards ref to the underlying `TextInput`.

### Form.Label (FormLabelProps)

`htmlFor?: string` (unused for a11y currently), `required?: boolean` (renders red ` *`), `children`. Static 14px semibold `<Text>`.

### Form.Error (FormErrorProps)

`name?: string`, `error?: string`. Shows the explicit `error` if given, otherwise `formContext.errors[name]`; renders nothing when clear. Red 12px `<Text>`.

### Form.Submit (FormSubmitProps)

`children` (string → the Button `title`; non-strings render literal `"Submit"`), `disabled?`, plus any `Button` props. Press calls `submitForm()`. Auto-disabled when `isSubmitting`, form `disabled`, or `!isValid`.

## Validation utilities — `import { ... } from '@platform-blocks/ui/Input'`

Not exported from the package root; use the `/Input` subpath. Also exports types `ValidationRule`, `PasswordStrengthRule`, `BaseInputProps`.

```ts
interface ValidationRule {
  type: 'required' | 'minLength' | 'maxLength' | 'pattern' | 'custom' | 'passwordStrength';
  value?: any;              // length for min/maxLength, RegExp or string for pattern
  message: string;
  validator?: (value: any, formValues?: Record<string, any>) => boolean | Promise<boolean>; // for 'custom'
}
interface PasswordStrengthRule extends Omit<ValidationRule, 'type'> {
  type: 'passwordStrength';
  requirements: { minLength?; requireUppercase?; requireLowercase?; requireNumbers?; requireSymbols? };
}
```

- `validateValue(value, rules, formValues?) => Promise<string[]>` — messages of failed rules. `required` rejects only `undefined`/`null`/`''` (so `false` passes — use a `custom` rule for must-be-checked).
- `validationRules` presets: `required(msg?)`, `minLength(n, msg?)`, `maxLength(n, msg?)`, `email(msg?)`, `url(msg?)`, `number(msg?)`, `passwordStrength(requirements, msg?)`, `custom(validator, msg)`.
- `calculatePasswordStrength(password, rules?)` — used by `PasswordInput`'s strength meter.

## BaseInputProps (shared by Input, PasswordInput, TextArea, NumberInput, Slider, PinInput, PhoneInput, DatePickerInput, TimePickerInput, FileInput)

Also extends spacing (`p`, `m`, ...), layout, and border-radius system props.

| Prop | Type | Description |
| --- | --- | --- |
| `variant` | `'default' \| 'filled' \| 'outline' \| 'unstyled'` | Field shell styling |
| `value` / `onChangeText` | `string` / `(text) => void` | Controlled value (pickers/number/pin/phone/slider override these with typed `value`/`onChange`) |
| `label` | `ReactNode` | Label above the field |
| `description` | `string` | Short text under the label, above the field |
| `helperText` | `string` | Text below the field (hidden while `error` shows on Input) |
| `error` | `string` | Error message below the field + error styling |
| `required` / `withAsterisk` | `boolean` | Required state / show the `*` indicator |
| `placeholder`, `placeholderTextColor` | `string` | |
| `disabled` | `boolean` | |
| `size` | `SizeValue` (`'xs'...'xl'`) | |
| `name` | `string` | Form integration id |
| `startSection` / `endSection` | `ReactNode` | Adornments (with `startSectionProps`/`endSectionProps`) |
| `clearable`, `clearButtonLabel`, `onClear` | | Built-in clear button |
| `onFocus`, `onBlur`, `onEnter` | `() => void` | |
| `debounceMs` | `number` | Validation debounce |
| `keyboardFocusId` | `string` | Id used with `KeyboardManagerProvider` refocus |
| `labelProps` / `descriptionProps` | `Omit<TextProps,'children'>` | Label/description text overrides |
| `accessibilityLabel`, `accessibilityHint`, `testID` | `string` | A11y label falls back to `label`; hint falls back to `helperText` |

## Text inputs

### Input (InputProps extends BaseInputProps)

`type` (`'text' \| 'password' \| 'email' \| 'tel' \| 'number' \| 'search'` — presets keyboardType/secureTextEntry/autoCapitalize/autoComplete), `autoComplete`, `keyboardType`, `multiline`, `numberOfLines`, `minLines`, `maxLines`, `maxLength`, `secureTextEntry`, `textInputProps` (raw RN TextInput props + `onKeyDown`/`onKeyUp`), `inputRef`, plus native passthroughs: `autoCapitalize`, `autoCorrect`, `autoFocus`, `returnKeyType`, `blurOnSubmit`, `selectTextOnFocus`, `textContentType`, `textAlign`, `spellCheck`, `inputMode`, `enterKeyHint`, `selectionColor`, `showSoftInputOnFocus`, `editable`. The `validation?: ValidationRule[]` prop exists but is currently ignored by the component. `type="password"` gets a built-in visibility toggle.

### PasswordInput

`Omit<InputProps, 'type' | 'secureTextEntry'>` + `showStrengthIndicator` (default `false`), `showVisibilityToggle` (default `true`), `strengthValidation?: PasswordStrengthRule[]`.

### TextArea (extends BaseInputProps)

`defaultValue`, `rows`, `minRows`, `maxRows`, `autoResize`, `maxLength`, `showCharCounter`, `h` (fixed height), `resize` (`'none' \| 'vertical' \| 'horizontal' \| 'both'`), `textInputProps`, `scrollEnabled`, plus the same native passthroughs as Input.

### NumberInput (BaseInputProps minus value/onChangeText)

`value?: number`, `onChange?: (value: number | undefined) => void`, `min`, `max`, `step`, `precision`, `allowDecimal`, `allowNegative`, `allowLeadingZeros`, `decimalSeparator`, `allowedDecimalSeparators`, `decimalScale`, `fixedDecimalScale`, `thousandSeparator` (string or `true`), `thousandsGroupStyle`, `prefix`, `suffix`, `format` (`'integer' \| 'decimal' \| 'currency' \| 'percentage'`), `currency`, `isAllowed(values)`, `startValue`, `stepHoldDelay`, `stepHoldInterval`, `shiftMultiplier`, `withKeyboardEvents`, `withControls`, `withSideButtons`, `hideControlsOnMobile`, `withDragGesture` (+ `dragAxis`, `dragStepDistance`, `dragStepMultiplier`, `onDragStateChange`), `formatter`, `parser`, `clampBehavior` (`'strict' \| 'blur' \| 'none'`), `allowEmpty`.

## Choice controls

### Select&lt;T&gt;

`options: SelectOption<T>[]` (`{ label, value, description?, disabled? }`), `value?: T | null`, `defaultValue`, `onChange?: (value: T | null, option?: SelectOption<T> | null) => void`, `placeholder`, `label`, `description`, `helperText`, `error` (string), `searchable`, `clearable` (+ `clearButtonLabel`, `onClear`), `disabled`, `size`, `radius`, `fullWidth`, `maxH`, `closeOnSelect`, `refocusAfterSelect`, `keyboardAvoidance`, `renderOption(opt, active, selected)`, `variant` (`InputVariant`), `labelProps`, `descriptionProps`.

### AutoComplete

`value?: string`, `onChangeText`, `data?: AutoCompleteOption[]` (`{ label, value, group?, disabled?, data? }`), `onSearch?: (query) => Promise<AutoCompleteOption[]>`, `minSearchLength`, `searchDelay`, `onSelect(item)`, `renderItem`, `renderValue`, `allowCustomValue`, `maxSuggestions`, `label`, `description`, `helperText`, `required`, `error`, `placeholder`, `disabled`, `size`, `radius`, `clearable`/`clearButtonLabel`/`onClear`, multi-select support via chips (`ChipProps`-based).

### Checkbox

`checked`, `defaultChecked`, `onChange?: (checked: boolean) => void`, `indeterminate`, `indeterminateIcon`, `label` (ReactNode), `description`, `error` (string), `labelPosition` (`'left' \| 'right' \| 'top' \| 'bottom'`), `color`, `colorVariant` (`'primary' \| 'secondary' \| 'success' \| 'error' \| 'warning'`), `size`, `disabled`, `required`, `children` (alternative to label), `labelProps`, `descriptionProps`, `accessibilityLabel`, `testID`, spacing props.

### Switch

`checked`, `defaultChecked`, `onChange?: (checked: boolean) => void`, `label`, `description`, `error` (string), `labelPosition` (`'left' \| 'right' \| 'top' \| 'bottom'`), `size`, `disabled`, `children`, `labelProps`, `descriptionProps`, `accessibilityLabel`, spacing props.

### Radio / RadioGroup

`Radio`: `value: string` (required), `checked`, `onChange?: (value: string) => void`, `name`, `label`, `description`, `error`, `labelPosition` (`'left' \| 'right'`), `icon`, `size`, `color`, `disabled`, `required`, `transitionDuration`.
`RadioGroup`: `options: Array<{ label, value, disabled?, description?, icon? }>` (required), `value`, `onChange(value)`, `name`, `orientation` (`'vertical' \| 'horizontal'`), `variant` (`'default' \| 'card' \| 'segmented' \| 'chip'`), `label`, `description`, `error`, `gap`, `labelPosition`, `size`, `color`, `disabled`, `required`.

### SegmentedControl

`data: SegmentedControlData[]` (`{ value, label, ... }`), `value`, `defaultValue`, `onChange(value)`, `label`, `description`.

### Slider / RangeSlider

Slider extends `BaseInputProps` (minus value/onChangeText/label/variant) so it has `error`/`helperText`/`description`. `value?: number`, `defaultValue`, `onChange(value)`, `min`, `max`, `step`, `orientation`, `variant` (`'default' \| 'filled' \| 'outline' \| 'minimal' \| 'segmented' \| 'unstyled'`), `label` (ReactNode), `valueLabel` (formatter or `null`), `valueLabelAlwaysOn`, `valueLabelPosition`/`Offset`/`Style`/`Props`/`AsCard`, `showMarks`, `ticks: SliderTick[]`, `showTicks`, `restrictToTicks`, `colorScheme`, `trackColor`/`activeTrackColor`/`thumbColor`, `trackSize`/`thumbSize`, style overrides. `RangeSlider`: `value?: [number, number]`, `onChange?: ([lo, hi])`.

## Specialty inputs

### PinInput

`length`, `value`, `defaultValue`, `onChange?: (pin: string) => void`, `onComplete?: (pin: string) => void`, `mask`, `maskChar`, `manageFocus`, `enforceOrderInitialOnly`, `type` (`'alphanumeric' \| 'numeric'`), `placeholder` (per cell), `allowPaste`, `oneTimeCode` (OTP autofill), `spacing`, `borderRadius`, `keyboardFocusId`, `autoFocus`, `textContentType`, plus BaseInputProps label/error/etc.

### PhoneInput

`value?: string` (national digits only), `defaultValue`, `onChange?: (raw, formatted, meta: PhoneChangeMeta) => void` where meta = `{ country, dialCode, e164, isComplete }` — submit `meta.e164`. `country` / `defaultCountry` (default `'US'`) / `onCountryChange`, `selectableCountry` (dial-code dropdown), `autoDetect` (switch country on typed/pasted `+<code>`), `showCountryCode`, `mask` (custom, `0` = digit), `textInputProps`, plus BaseInputProps.

### DatePickerInput (and friends)

`value?: CalendarValue` (`Date | Date[] | [Date | null, Date | null] | null`), `defaultValue`, `onChange(value)`, `type` (`'single' \| 'multiple' \| 'range'`), `calendarProps` (pass-through to `Calendar`), `placeholder`, `displayFormat`, `clearable`, `size`, `disabled`, `withAsterisk`, `dropdownType` (`'modal' \| 'popover'`), `closeOnSelect`, `onOpen`/`onClose`/`onFocus`/`onBlur`, plus BaseInputProps label/error/etc.
`MonthPickerInput` / `YearPickerInput`: `value?: Date | null`, `onChange(value)`, `formatValue(date)`. `TimePickerInput`: `value?: TimePickerValue | null` (`{ hours: 0-23, minutes, seconds? }`), `defaultValue`, `onChange(next)`, `label`, `description`, `error`, `helperText`. Inline pickers: `DatePicker`, `Calendar`, `TimePicker`.

### ColorInput

`value?: string` (hex), `defaultValue`, `onChange?: (color: string) => void`, `label`, `placeholder`, `disabled`, `required`, `error` (string), `description`, `size`, `variant` (`'default' \| 'filled' \| 'unstyled'`), spacing/layout/radius props.

### FileInput

Extends BaseInputProps (label/error/helperText...). `variant` (`'standard' \| 'dropzone' \| 'compact'`), `accept: string[]`, `multiple`, `maxSize` (bytes), `maxFiles`, `onFilesChange?: (files: FileInputFile[]) => void`, `onFileRemove(fileId)`, `onUpload?: (files) => Promise<void>`, `onProgress(fileId, progress)`, `validateFile(file) => string | null`, `showFileList`, `enableDragDrop`, `PreviewComponent`, `imagePreview`, `uploadSettings` (`{ url, method, headers, fieldName, formData }`). `FileInputFile` = `{ file, id, name, size, type, uri?, previewUrl?, progress?, status?, error? }`.

## ControlField / ControlField.Group

Root exports: `ControlField`, `ControlFieldGroup`, `useControlField()`, `useControlFieldContext()`, `useControlFieldGroup()`.

`ControlFieldProps`: `isSelected` (alias `checked`), `defaultSelected`, `onSelectedChange` (alias `onChange`) — `(selected: boolean) => void`, `isDisabled`/`disabled`, `isInvalid` (defaults true when `error` set), `isRequired`/`required`, `variant` (`'checkbox' \| 'radio' \| 'switch'`, default `'switch'`), `label`, `description`, `error` (string), `color`, `colorVariant`, `size`, `indicatorPosition` (`'left' \| 'right'`, default `'right'`), `control` (custom element; `checked`/`disabled` injected), `labelProps`, `descriptionProps`, `children` (compound mode), `accessibilityLabel`, `style`, `testID`, spacing props.

Compound parts: `ControlField.Label`, `ControlField.Description`, `ControlField.Indicator` (`variant?`, `children?` custom control), `ControlField.Error`, `ControlField.Group`. `useControlField()` exposes `{ isSelected, onSelectedChange, isDisabled, isInvalid, isRequired, size, color, colorVariant, variant }` for custom controls.

`ControlFieldGroupProps`: `children`, `variant` (`'default' \| 'bordered' \| 'flush'`), `dividers` (default `true`), `insetDividers`, `radius` (`'sm' \| 'md' \| 'lg'` or number, default `'md'`), `size` (applied to all child fields), `title`, `footer`, `style`, `testID`, spacing props.

## FormLayout family — `import { ... } from '@platform-blocks/ui/FormLayout'`

Not in the root barrel; subpath import only.

| Component | Props |
| --- | --- |
| `FormLayout` | `maxWidth` (default `600`), `spacing` (`'sm' \| 'md' \| 'lg' \| 'xl'`, default `'lg'`), `variant` (`'default' \| 'card' \| 'modal'`) — centered column with gap |
| `FormSection` | `title`, `description`, `spacing` (`'sm' \| 'md' \| 'lg'`), `collapsible`, `defaultCollapsed` |
| `FormGroup` | `direction` (`'row' \| 'column'`), `columns` (`2 \| 3 \| 4`), `spacing` (`'xs'...'lg'`), `align` (`'start' \| 'center' \| 'end' \| 'stretch'`) |
| `FormField` | `label`, `description`, `error` (string), `required` (default `false`), `width` (`'auto' \| 'full' \| number`, default `'full'`), `labelPosition` (`'top' \| 'left' \| 'right'`, default `'top'`) — layout-only label/error wrapper, renders error below children |

## Keyboard

### KeyboardAwareLayout (root export)

`KeyboardAvoidingView` (+ optional inner `ScrollView`). Props: `behavior` (platform default when omitted), `keyboardVerticalOffset` (default `0`), `enabled` (default `true`), `scrollable` (default `true`), `extraScrollHeight` (extra padding beyond keyboard height), `style`, `contentContainerStyle`, `keyboardShouldPersistTaps` (`'always' \| 'never' \| 'handled'`, default `'handled'`), `scrollRef`, `scrollViewProps`, plus ScrollView passthroughs (`scrollEnabled`, `bounces`, `onScroll`, `scrollEventThrottle`, `showsVerticalScrollIndicator`, `refreshControl`, ...), and spacing props.

### KeyboardManagerProvider (root export)

Props: `children`, `disabled?`. `useKeyboardManager()` (throws outside provider) / `useKeyboardManagerOptional()` return `KeyboardManagerContextValue`:

| Member | Description |
| --- | --- |
| `isKeyboardVisible`, `keyboardHeight` | Live keyboard state |
| `keyboardEndCoordinates`, `keyboardAnimationDuration`, `keyboardAnimationEasing` | From the last native keyboard event |
| `dismissKeyboard()` | Imperative dismiss |
| `setFocusTarget(id \| null)` / `consumeFocusTarget(id)` / `pendingFocusTarget` | Focus-restoration handshake |
| `refocus(componentId, { dismiss? })` | Record a focus target so the input with that `keyboardFocusId` restores focus |
