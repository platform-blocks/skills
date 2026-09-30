---
name: plocks-forms
description: Build forms with the @plocks/ui React Native library. Use when composing form screens with Form/Form.Field/Form.Input/Form.Submit or FormLayout/ControlField, wiring input components (Input, PasswordInput, TextArea, NumberInput, Select, AutoComplete, Checkbox, Switch, RadioGroup, SegmentedControl, Slider, PinInput, PhoneInput, DatePickerInput, TimePickerInput, ColorInput, FileInput), implementing validation (validationSchema, validationRules, validateValue, on-submit/on-blur with React state), choosing controlled vs uncontrolled inputs (value/onChangeText vs onChange, defaultValue/defaultChecked), or handling keyboards (KeyboardAwareLayout, KeyboardManagerProvider) and form accessibility.
---

# plocks Forms

Form building with `@plocks/ui` for React Native (iOS/Android/Web). Everything below imports from the package root unless noted:

```tsx
import { Form, Input, Select, Checkbox, Button, KeyboardAwareLayout } from '@plocks/ui';
```

Four reference files sit alongside this one:

- `references/api.md` — the curated API: how the pieces compose, the union
  types, the defaults that bite. **Read this first.**
- `references/props.md` — generated, exhaustive prop tables for every component
  in this skill. Look here for the complete surface of a single prop.
- `references/icons.md` — generated, every `name` the built-in `Icon` registry
  accepts. Check it before writing `<Icon name="…">`; unlisted names render
  nothing.
- `references/patterns.md` — complete copy-paste screens.

## The form model — two valid approaches

**1. Plain React state (recommended default).** Every input is a controlled component. Hold values and errors in `useState`, validate on submit and/or blur, pass `error` strings back into the inputs. This works uniformly across all control types.

**2. Built-in `Form` compound.** `Form` is a real (but thin) state container: a context provider (`FormProvider`) holding `values`, `errors`, `touched`, `isSubmitting`, plus a plain `<View>`. It is geared toward **text inputs** — its field binding speaks `value`/`onChangeText`/`onBlur`.

```tsx
<Form
  initialValues={{ email: '' }}
  validationSchema={{ email: [
    { type: 'required', message: 'Email is required' },
    { type: 'pattern', value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/, message: 'Invalid email' },
  ]}}
  onSubmit={async (values) => { /* called only when validation passes */ }}
>
  <Form.Field name="email">
    <Form.Input label="Email" placeholder="ada@example.com" />
  </Form.Field>
  <Form.Submit>Create account</Form.Submit>
</Form>
```

`FormProps`: `initialValues`, `validationSchema` (`{ [field]: ValidationRule[] }`), `onSubmit(values)`, `validate(values) => errorsMap` (sync or async), `disabled`, `validateOnChange` (default `true`), `validateOnBlur` (default `true`).

Submission (`Form.Submit` press or `submitForm()`): marks all schema fields touched, runs `validate` + the schema, stores errors, and calls `onSubmit(values)` **only if there are no errors**.

### Form gotchas (verified against source)

- `Form.Field` injects only `name`/`disabled`/`required`, and only into `Form.Input`, `Form.Label`, `Form.Error` children. A plain `<Input>` inside `Form.Field` renders but is **not bound** to form state — use `Form.Input`, or bind manually with `useFormContext().getFieldProps(name)`.
- `getFieldProps(name)` returns `{ value, onChangeText, onBlur, error, name }` — `error` is only exposed once the field is `touched`, and `value` falls back to `''`. That contract fits `Input`/`PasswordInput`/`TextArea`/`AutoComplete`. Other controls use different prop names (below), so wire them with `values`, `setFieldValue`, `setFieldTouched` from `useFormContext()` instead.
- `Form.Submit` renders a `Button` with `title` = its children (string only; anything else renders literal "Submit"). It disables itself while `isSubmitting`, when the form is `disabled`, or when `isValid` is false.
- `Form.Field`'s `dependsOn` (show/hide/enable/disable/require based on other fields) works; its `validation` and `validateWhen` props exist in the types but are **not wired up** — put rules in `validationSchema`.
- The `validation` prop on `InputProps` is accepted but currently **inert** on `Input` — validation lives at the Form level or in your own state logic.
- A `required` rule passes for `false` (it only rejects `undefined`/`null`/`''`), so validate checkboxes with a `custom` rule: `{ type: 'custom', validator: (v) => v === true, message: '...' }`.

## Field anatomy: labels, help text, errors

Text-family inputs (`Input`, `PasswordInput`, `TextArea`, `NumberInput`, `Slider`, `PinInput`, `PhoneInput`, `DatePickerInput`, `TimePickerInput`, `FileInput` — all extend `BaseInputProps`) share:

- `label` — rendered above the field; `required` + `withAsterisk` add the `*` indicator
- `description` — short text directly under the label (above the field)
- `helperText` — text below the field; on `Input` it is hidden while an error shows
- `error` — **string** message below the field; also switches the field into error styling
- `labelProps` / `descriptionProps` — style overrides for the label/description `<Text>`

`Select` and `AutoComplete` have their own props but the same `label`/`description`/`helperText`/`error`/`required` names. `Checkbox`, `Switch`, `Radio`, `RadioGroup`, `ControlField`, `ColorInput` have `label`, `description`, and `error` (no `helperText`).

## Controlled vs uncontrolled + prop-name cheat sheet

Prop names differ per control — React Native text inputs use `onChangeText`, most others use `onChange`. Omit `value`/`checked` and use the `default*` prop for uncontrolled usage where listed.

| Component | Value prop | Change handler | Uncontrolled |
| --- | --- | --- | --- |
| `Input`, `PasswordInput`, `TextArea`, `AutoComplete` | `value: string` | `onChangeText(text)` | `defaultValue` (TextArea only) |
| `NumberInput` | `value?: number` | `onChange(value: number \| undefined)` | — |
| `Select<T>` | `value?: T \| null` | `onChange(value, option)` | `defaultValue` |
| `Checkbox`, `Switch` | `checked: boolean` | `onChange(checked)` | `defaultChecked` |
| `ControlField` | `isSelected` (alias `checked`) | `onSelectedChange` (alias `onChange`) | `defaultSelected` |
| `Radio`, `RadioGroup`, `SegmentedControl` | `value: string` | `onChange(value)` | `defaultValue` (SegmentedControl) |
| `Slider` / `RangeSlider` | `value: number` / `[number, number]` | `onChange(value)` | `defaultValue` |
| `PinInput` | `value: string` | `onChange(pin)` + `onComplete(pin)` | `defaultValue` |
| `PhoneInput` | `value: string` (national digits) | `onChange(raw, formatted, meta)` — `meta.e164` is submittable | `defaultValue` |
| `DatePickerInput` | `value: CalendarValue` (`Date \| Date[] \| [Date\|null, Date\|null] \| null`) | `onChange(value)` | `defaultValue` |
| `MonthPickerInput`, `YearPickerInput` | `value?: Date \| null` | `onChange(value)` | `defaultValue` |
| `TimePickerInput` | `value?: TimePickerValue \| null` (`{hours, minutes, seconds?}`) | `onChange(next)` | `defaultValue` |
| `ColorInput` | `value: string` (hex) | `onChange(color)` | `defaultValue` |
| `FileInput` | — (internal list) | `onFilesChange(files: FileInputFile[])` | n/a |

## Validation

`ValidationRule`: `{ type: 'required' | 'minLength' | 'maxLength' | 'pattern' | 'custom' | 'passwordStrength', value?, message, validator? }`.

Rule builders and the runner live on the `Input` subpath (not the package root):

```tsx
import { validationRules, validateValue } from '@plocks/ui/Input';

const rules = [validationRules.required(), validationRules.email()];
const errors = await validateValue(email, rules); // Promise<string[]> of failed messages
```

Use them inside `validationSchema` (the `Form` runs `validateValue` per field: on change, on blur, and on submit — errors surface once a field is touched) or call `validateValue` yourself in plain-state forms. `validationRules` presets: `required`, `minLength`, `maxLength`, `email`, `url`, `number`, `passwordStrength`, `custom`. `PasswordInput` adds `showStrengthIndicator`, `showVisibilityToggle`, and `strengthValidation` (uses `calculatePasswordStrength`).

## Layout components

- **`ControlField` / `ControlField.Group`** (root export) — a pressable row combining label + description + a checkbox/radio/switch indicator (`variant`, default `'switch'`; `indicatorPosition` default `'right'`). `ControlField.Group` wraps rows in an iOS-settings-style surface with dividers, `title`, and `footer`. Compose `ControlField.Label/.Description/.Indicator/.Error` for full control.
- **`FormLayout` family** — `FormLayout`, `FormSection`, and `FormGroup` are root exports. The layout `FormField` is available from `@plocks/ui/FormLayout`; it is distinct from `Form.Field`. `FormLayout` centers a `maxWidth` column, `FormSection` groups fields under a title, and `FormGroup` arranges rows or columns.

## Keyboard handling

- Wrap form screens in **`KeyboardAwareLayout`** (root export): `KeyboardAvoidingView` + `ScrollView` with keyboard-height padding. Defaults: `scrollable` `true`, `keyboardShouldPersistTaps="handled"`, `enabled` `true`; tune with `behavior`, `keyboardVerticalOffset`, `extraScrollHeight`, `contentContainerStyle`, `scrollRef`.
- Mount **`KeyboardManagerProvider`** near the app root to enable keyboard state tracking. `useKeyboardManager()` gives `isKeyboardVisible`, `keyboardHeight`, `dismissKeyboard()`, and `refocus(componentId)`; inputs opt in to focus restoration via their `keyboardFocusId` prop.
- Per-input keyboard control: `keyboardType`/`inputMode`, `returnKeyType`/`enterKeyHint`, `blurOnSubmit`, `autoFocus`, `showSoftInputOnFocus`, and `onEnter` (fired on submit/enter). `Select` has `keyboardAvoidance` for dropdown positioning.
- `Input type="email" | "password" | "tel" | "number" | "search"` presets keyboard type, secure entry, capitalization, and autocomplete.

## Accessibility

- All `BaseInputProps` inputs accept `accessibilityLabel` (falls back to the string `label`) and `accessibilityHint` (falls back to `helperText`). `Input` announces the label + "required" on focus, renders errors with `role="alert"` + a live region, and announces them assertively on blur.
- `Checkbox`, `Switch`, and `ControlField` accept `accessibilityLabel` for when there is no visible text label. `ControlField` makes the whole row a single pressable/accessible target.
- Always set `testID` on fields you need to reach in tests.

## Anything this skill does not cover

This skill covers form composition, input components, validation, and keyboard
handling. plocks is much larger — 100+ core UI components, 24 chart types, and
48 documented hooks. Do not guess an API for something outside this scope;
fetch the generated docs instead:

| What you need | Where |
| --- | --- |
| Index of every page, one line each | `https://plocks.dev/llms.txt` |
| One component or chart | `https://plocks.dev/llms/components/<Name>.md` |
| One hook | `https://plocks.dev/llms/hooks/<useName>.md` |
| Guides | `https://plocks.dev/llms/guides/{getting-started,accessibility,localization}.md` |
| Everything in one file (~1.3 MB) | `https://plocks.dev/llms-full.txt` |

`<Name>` is the exact PascalCase export name — `.../llms/components/DataTable.md`,
`.../llms/components/AreaChart.md`. Each page carries the component's full prop
table (type, required, default, description) plus runnable examples, generated
from the source, so it is authoritative where memory is not. When you are unsure
whether something exists or what it is called, read `llms.txt` first — it lists
every page with a one-line summary.

Import components from their owning package: `@plocks/ui` for core UI,
`@plocks/charts`, `@plocks/dates`, `@plocks/code`, `@plocks/media`,
`@plocks/qrcode`, and `@plocks/spotlight` for extensions. `FormLayout` is
also exported from `@plocks/ui`; the `Navigation` module uses the
`@plocks/ui/Navigation` subpath. Check the package barrel before importing.

Notably outside this skill:

- **Reporting submit results** — `Toast` (via `useToast`), `Alert`,
  `LoadingOverlay`, `Dialog`. A form screen almost always needs one of these.
- **Date/time *display* components** — only the `*Input` variants are covered
  here; `Calendar`, `MiniCalendar`, `DatePicker`, `TimePicker`, `MonthPicker` and
  `YearPicker` have their own pages.
- **Tables and lists** for showing submitted data — `DataTable`, `DataList`,
  `Table`.
- **Install and provider wiring** → the `plocks-setup` skill. **Screen
  layout** → `plocks-layout`. **Theme tokens** →
  `plocks-theming`.
