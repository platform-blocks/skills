# Platform Blocks Forms — patterns

Complete, copy-paste form screens. All imports are from the package root except `validationRules`/`validateValue` (`@platform-blocks/ui/Input`) and the `FormLayout` family (`@platform-blocks/ui/FormLayout`).

## App root setup (keyboard)

```tsx
import { PlatformBlocksProvider, KeyboardManagerProvider } from '@platform-blocks/ui';

export default function App() {
  return (
    <PlatformBlocksProvider>
      <KeyboardManagerProvider>
        {/* navigation / screens */}
      </KeyboardManagerProvider>
    </PlatformBlocksProvider>
  );
}
```

## 1. Sign-up form with plain React state (recommended baseline)

Controlled inputs, validation on submit + on blur, error strings passed straight into each input's `error` prop.

```tsx
import { useState } from 'react';
import {
  KeyboardAwareLayout, Column, Title, Text, Input, PasswordInput, Checkbox, Button,
} from '@platform-blocks/ui';
import { validationRules, validateValue } from '@platform-blocks/ui/Input';

const schema = {
  name: [validationRules.required('Name is required')],
  email: [validationRules.required('Email is required'), validationRules.email()],
  password: [
    validationRules.required('Password is required'),
    validationRules.minLength(8),
  ],
  terms: [validationRules.custom((v) => v === true, 'You must accept the terms')],
};

type Values = { name: string; email: string; password: string; terms: boolean };

export function SignUpScreen() {
  const [values, setValues] = useState<Values>({ name: '', email: '', password: '', terms: false });
  const [errors, setErrors] = useState<Partial<Record<keyof Values, string>>>({});
  const [submitting, setSubmitting] = useState(false);

  const setField = <K extends keyof Values>(key: K, value: Values[K]) => {
    setValues((prev) => ({ ...prev, [key]: value }));
    if (errors[key]) setErrors((prev) => ({ ...prev, [key]: undefined })); // clear on edit
  };

  const validateField = async (key: keyof Values) => {
    const messages = await validateValue(values[key], schema[key], values);
    setErrors((prev) => ({ ...prev, [key]: messages[0] }));
  };

  const handleSubmit = async () => {
    const nextErrors: typeof errors = {};
    for (const key of Object.keys(schema) as (keyof Values)[]) {
      const messages = await validateValue(values[key], schema[key], values);
      if (messages.length) nextErrors[key] = messages[0];
    }
    setErrors(nextErrors);
    if (Object.keys(nextErrors).length > 0) return;

    setSubmitting(true);
    try {
      await api.signUp(values);
    } catch (e) {
      setErrors({ email: 'An account with this email already exists' }); // server error
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <KeyboardAwareLayout contentContainerStyle={{ padding: 20 }}>
      <Column gap="lg" style={{ maxWidth: 480, alignSelf: 'center', width: '100%' }}>
        <Column gap="sm">
          <Title order={1}>Create account</Title>
          <Text colorVariant="secondary">It takes less than a minute.</Text>
        </Column>

        <Input
          label="Full name"
          placeholder="Ada Lovelace"
          value={values.name}
          onChangeText={(t) => setField('name', t)}
          onBlur={() => validateField('name')}
          error={errors.name}
          required
          withAsterisk
          autoCapitalize="words"
          returnKeyType="next"
          testID="signup-name"
        />

        <Input
          type="email"
          label="Email"
          placeholder="ada@example.com"
          helperText="We never share your email."
          value={values.email}
          onChangeText={(t) => setField('email', t)}
          onBlur={() => validateField('email')}
          error={errors.email}
          required
          withAsterisk
          returnKeyType="next"
        />

        <PasswordInput
          label="Password"
          description="At least 8 characters"
          value={values.password}
          onChangeText={(t) => setField('password', t)}
          onBlur={() => validateField('password')}
          error={errors.password}
          required
          withAsterisk
          showStrengthIndicator
          strengthValidation={[{
            type: 'passwordStrength',
            requirements: { minLength: 8, requireUppercase: true, requireNumbers: true },
            message: 'Add an uppercase letter and a number',
          }]}
          returnKeyType="done"
          onEnter={handleSubmit}
        />

        <Checkbox
          label="I agree to the Terms of Service"
          checked={values.terms}
          onChange={(checked) => setField('terms', checked)}
          error={errors.terms}
        />

        <Button title="Create account" onPress={handleSubmit} loading={submitting} fullWidth />
      </Column>
    </KeyboardAwareLayout>
  );
}
```

## 2. Built-in `Form` compound (context-managed state)

Text fields bind automatically through `Form.Field` + `Form.Input`. Non-text controls (Select, Checkbox, ...) are bound manually with `useFormContext()` — `getFieldProps` speaks `value`/`onChangeText` and does not fit them.

```tsx
import {
  Form, useFormContext, Select, Checkbox, Column, Title, KeyboardAwareLayout,
} from '@platform-blocks/ui';

// Custom-bound field: read values/errors from context, write with setFieldValue.
function RoleField() {
  const { values, errors, touched, setFieldValue, setFieldTouched } = useFormContext();
  return (
    <Select
      label="Role"
      placeholder="Choose a role"
      options={[
        { label: 'Engineer', value: 'engineer' },
        { label: 'Designer', value: 'designer' },
        { label: 'Product', value: 'product', description: 'PM / product ops' },
      ]}
      value={values.role ?? null}
      onChange={(value) => {
        setFieldValue('role', value);
        setFieldTouched('role', true);
      }}
      error={touched.role ? errors.role : undefined}
    />
  );
}

function TermsField() {
  const { values, errors, touched, setFieldValue, setFieldTouched } = useFormContext();
  return (
    <Checkbox
      label="I agree to the Terms of Service"
      checked={values.terms === true}
      onChange={(checked) => {
        setFieldValue('terms', checked);
        setFieldTouched('terms', true);
      }}
      error={touched.terms ? errors.terms : undefined}
    />
  );
}

export function InviteScreen() {
  return (
    <KeyboardAwareLayout contentContainerStyle={{ padding: 20 }}>
      <Form
        initialValues={{ name: '', email: '', role: null, terms: false }}
        validationSchema={{
          name: [{ type: 'required', message: 'Name is required' }],
          email: [
            { type: 'required', message: 'Email is required' },
            { type: 'pattern', value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/, message: 'Enter a valid email' },
          ],
          role: [{ type: 'required', message: 'Pick a role' }],
          terms: [{ type: 'custom', validator: (v) => v === true, message: 'Required' }],
        }}
        onSubmit={async (values) => {
          await api.invite(values); // reached only when every rule passes
        }}
      >
        <Column gap="lg" style={{ maxWidth: 480, alignSelf: 'center', width: '100%' }}>
          <Title order={2}>Invite a teammate</Title>

          {/* Form.Field injects name into Form.Input; binding = value/onChangeText/onBlur/error */}
          <Form.Field name="name">
            <Form.Input label="Full name" placeholder="Ada Lovelace" />
          </Form.Field>

          <Form.Field name="email">
            <Form.Input type="email" label="Email" placeholder="ada@example.com" />
          </Form.Field>

          <RoleField />
          <TermsField />

          {/* Renders a Button; disabled while submitting/invalid; string children only */}
          <Form.Submit>Send invite</Form.Submit>
        </Column>
      </Form>
    </KeyboardAwareLayout>
  );
}
```

Conditional fields with `dependsOn` (implemented; `validation`/`validateWhen` on `Form.Field` are not):

```tsx
<Form.Field
  name="companyName"
  dependsOn={[{ field: 'accountType', condition: (v) => v === 'business', action: 'show' }]}
>
  <Form.Input label="Company name" />
</Form.Field>
```

## 3. Settings screen with ControlField.Group

```tsx
import { useState } from 'react';
import { ScrollView } from 'react-native';
import { ControlField, Column, Title } from '@platform-blocks/ui';

export function NotificationSettings() {
  const [push, setPush] = useState(true);
  const [emailDigest, setEmailDigest] = useState(false);
  const [marketing, setMarketing] = useState(false);

  return (
    <ScrollView contentContainerStyle={{ padding: 20 }}>
      <Column gap="lg">
        <Title order={2}>Notifications</Title>

        <ControlField.Group
          title="Alerts"
          footer="You can change these at any time."
          variant="bordered"
          insetDividers
        >
          <ControlField
            label="Push notifications"
            description="Mentions, replies, and direct messages"
            isSelected={push}
            onSelectedChange={setPush}
          />
          <ControlField
            label="Weekly email digest"
            isSelected={emailDigest}
            onSelectedChange={setEmailDigest}
          />
          <ControlField
            label="Product updates"
            description="Occasional feature announcements"
            isSelected={marketing}
            onSelectedChange={setMarketing}
          />
        </ControlField.Group>
      </Column>
    </ScrollView>
  );
}
```

Consent row with checkbox indicator + validation state:

```tsx
<ControlField
  variant="checkbox"
  indicatorPosition="left"
  label="I agree to the Terms of Service"
  description="You must accept before continuing"
  isRequired
  isSelected={agreed}
  onSelectedChange={setAgreed}
  isInvalid={showErrors && !agreed}
  error={showErrors && !agreed ? 'This field is required' : undefined}
/>
```

## 4. Mixed-control profile form (plain state, one field per control family)

```tsx
import { useState } from 'react';
import {
  KeyboardAwareLayout, Column, Title, Input, TextArea, NumberInput, Select, RadioGroup,
  Slider, PhoneInput, DatePickerInput, ColorInput, Button,
} from '@platform-blocks/ui';

export function ProfileScreen() {
  const [displayName, setDisplayName] = useState('');
  const [bio, setBio] = useState('');
  const [teamSize, setTeamSize] = useState<number | undefined>(undefined);
  const [country, setCountry] = useState<string | null>(null);
  const [plan, setPlan] = useState('free');
  const [experience, setExperience] = useState(3);
  const [phoneE164, setPhoneE164] = useState('');
  const [birthday, setBirthday] = useState<Date | null>(null);
  const [accent, setAccent] = useState('#4263eb');

  return (
    <KeyboardAwareLayout contentContainerStyle={{ padding: 20 }} extraScrollHeight={24}>
      <Column gap="lg" style={{ maxWidth: 560, alignSelf: 'center', width: '100%' }}>
        <Title order={2}>Profile</Title>

        <Input label="Display name" value={displayName} onChangeText={setDisplayName} />

        <TextArea
          label="Bio"
          description="Shown on your public profile"
          value={bio}
          onChangeText={setBio}
          autoResize
          minRows={3}
          maxRows={8}
          maxLength={280}
          showCharCounter
        />

        <NumberInput
          label="Team size"
          value={teamSize}
          onChange={setTeamSize}
          min={1}
          max={500}
          step={1}
          withControls
          allowEmpty
        />

        <Select
          label="Country"
          placeholder="Select a country"
          searchable
          clearable
          options={[
            { label: 'United States', value: 'US' },
            { label: 'United Kingdom', value: 'GB' },
            { label: 'Germany', value: 'DE' },
          ]}
          value={country}
          onChange={(value) => setCountry(value)}
        />

        <RadioGroup
          label="Plan"
          variant="card"
          options={[
            { label: 'Free', value: 'free', description: 'For individuals' },
            { label: 'Pro', value: 'pro', description: 'For teams' },
          ]}
          value={plan}
          onChange={setPlan}
        />

        <Slider
          label="Years of experience"
          value={experience}
          onChange={setExperience}
          min={0}
          max={20}
          step={1}
          valueLabelAlwaysOn
        />

        <PhoneInput
          label="Phone"
          defaultCountry="US"
          selectableCountry
          onChange={(raw, formatted, meta) => setPhoneE164(meta.e164)}
        />

        <DatePickerInput
          label="Birthday"
          type="single"
          value={birthday}
          onChange={(value) => setBirthday(value as Date | null)}
          clearable
          closeOnSelect
        />

        <ColorInput label="Accent color" value={accent} onChange={setAccent} />

        <Button title="Save profile" onPress={() => api.save({ displayName, bio, teamSize, country, plan, experience, phone: phoneE164, birthday, accent })} />
      </Column>
    </KeyboardAwareLayout>
  );
}
```

## 5. OTP verification with PinInput

```tsx
import { useState } from 'react';
import { Column, Title, Text, PinInput, Button } from '@platform-blocks/ui';

export function VerifyCodeScreen() {
  const [code, setCode] = useState('');
  const [error, setError] = useState<string | undefined>();

  const verify = async (pin: string) => {
    setError(undefined);
    const ok = await api.verify(pin);
    if (!ok) setError('That code is incorrect. Try again.');
  };

  return (
    <Column gap="lg" p="xl" align="center">
      <Title order={2}>Enter the 6-digit code</Title>
      <Text colorVariant="secondary">We sent it to your phone.</Text>
      <PinInput
        length={6}
        type="numeric"
        oneTimeCode           // enables OS one-time-code autofill
        autoFocus
        value={code}
        onChange={(pin) => { setCode(pin); setError(undefined); }}
        onComplete={verify}   // fires when all cells are filled
        error={error}
      />
      <Button title="Resend code" variant="outline" onPress={() => api.resend()} />
    </Column>
  );
}
```

## 6. Structured layout with the FormLayout family (subpath import)

`FormLayout`/`FormSection`/`FormGroup`/`FormField` are layout-only (no state). Use `FormField` to add a label/error shell around controls that lack their own, or for left-aligned label columns.

```tsx
import { useState } from 'react';
import { ScrollView } from 'react-native';
import { Input, Switch, Button } from '@platform-blocks/ui';
import { FormLayout, FormSection, FormGroup, FormField } from '@platform-blocks/ui/FormLayout';

export function BillingSettings() {
  const [first, setFirst] = useState('');
  const [last, setLast] = useState('');
  const [autoRenew, setAutoRenew] = useState(true);

  return (
    <ScrollView contentContainerStyle={{ padding: 20 }}>
      <FormLayout maxWidth={640} variant="card" spacing="lg">
        <FormSection title="Billing contact" description="Who receives invoices">
          <FormGroup direction="row" columns={2} spacing="md">
            <Input label="First name" value={first} onChangeText={setFirst} />
            <Input label="Last name" value={last} onChangeText={setLast} />
          </FormGroup>
        </FormSection>

        <FormSection title="Renewal" collapsible defaultCollapsed={false}>
          <FormField
            label="Auto-renew"
            description="Charge the card on file each cycle"
            labelPosition="left"
          >
            <Switch checked={autoRenew} onChange={setAutoRenew} accessibilityLabel="Auto-renew" />
          </FormField>
        </FormSection>

        <Button title="Save" onPress={() => api.saveBilling({ first, last, autoRenew })} />
      </FormLayout>
    </ScrollView>
  );
}
```

## 7. Cross-field validation (confirm password)

With the built-in `Form`, `custom` validators receive `formValues` as the second argument:

```tsx
<Form
  initialValues={{ password: '', confirm: '' }}
  validationSchema={{
    password: [
      { type: 'required', message: 'Password is required' },
      { type: 'minLength', value: 8, message: 'Must be at least 8 characters' },
    ],
    confirm: [{
      type: 'custom',
      validator: (value, formValues) => value === formValues?.password,
      message: 'Passwords do not match',
    }],
  }}
  onSubmit={(values) => api.setPassword(values.password)}
>
  <Form.Field name="password">
    <Form.Input type="password" label="New password" />
  </Form.Field>
  <Form.Field name="confirm">
    <Form.Input type="password" label="Confirm password" />
  </Form.Field>
  <Form.Submit>Update password</Form.Submit>
</Form>
```

With plain state, pass `values` as `validateValue`'s third argument so `custom` rules can read siblings: `await validateValue(values.confirm, schema.confirm, values)`.
