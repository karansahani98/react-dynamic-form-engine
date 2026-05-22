# Dynamic Form Engine

> A schema-driven, fully dynamic form engine built in React + TypeScript — designed for large-scale healthcare module systems (OPD, IPD, HR, Hospital Settings, etc.) where forms are complex, conditional, and highly reusable.

---

## What this is

A standalone engine that renders any form from a JSON/TypeScript schema — no hardcoding, no repetitive form boilerplate. Pass in a schema, get a fully functional form with validation, conditional logic, computed fields, dependent dropdowns, draft saving, and API submission.

Built as part of a Hospital Management System but designed to be domain-agnostic.

---

## Architecture overview

```
Schema Input (JSON/TS)
        ↓
   formSchemaService → schemaParser → useFormSchema
        ↓
   Core Components
   ┌─────────────────────────────────────────────┐
   │  DynamicForm → FormStepper → FormSection    │
   │                    → FieldWrapper           │
   └─────────────────────────────────────────────┘
        ↓
   25+ Field Types
   (Text, Select, Autocomplete, Date, Currency,
    Computed, File, Image, Signature, Rating…)
        ↓
   ┌──────────────────┬──────────────────────────┐
   │   State Hooks    │   Utils & Validators      │
   │ useDynamicForm   │ conditionalEngine         │
   │ useConditional   │ validationEngine          │
   │ useDependent     │ schemaParser              │
   │ useComputed      │ formTransformer           │
   │ useFieldValidation│ fieldMapper              │
   │ useFormDraft     │ cacheManager              │
   │ useFormSubmit    │ Validators (8 types)      │
   │ useFieldDataSource│                          │
   └──────────────────┴──────────────────────────┘
        ↓
   Services (API Layer)
   formSchemaService · formSubmitService
   formDraftService  · masterDataService
        ↓
   Rendered Output
   FormPreview · FormActions → Submitted Payload
```

---

## Folder structure

```
form-engine/
├── components/          # Core UI: DynamicForm, FormStepper, FormSection,
│                        #   FieldWrapper, FormActions, FormPreview
├── fields/              # 25+ field components
│   ├── TextField.tsx
│   ├── SelectField.tsx
│   ├── AutocompleteField.tsx
│   ├── DateField.tsx / DateTimeField.tsx
│   ├── ComputedField.tsx
│   ├── CurrencyField.tsx / PercentageField.tsx
│   ├── FileUploadField.tsx / ImageUploadField.tsx
│   ├── SignatureField.tsx
│   ├── SliderField.tsx / RatingField.tsx
│   ├── MultiSelectField.tsx / CheckboxGroupField.tsx
│   ├── AddressField.tsx / PhoneField.tsx
│   └── ... (ColorPickerField, RadioField, HiddenField, etc.)
│
├── hooks/               # Business logic, one concern per hook
│   ├── useDynamicForm.ts         — master form state orchestrator
│   ├── useConditionalLogic.ts    — show/hide fields based on rules
│   ├── useDependentFields.ts     — cascading dropdown chains
│   ├── useComputedFields.ts      — auto-calculated field values
│   ├── useFieldValidation.ts     — per-field validation runner
│   ├── useFormDraft.ts           — save/restore in-progress forms
│   ├── useFormSubmit.ts          — submission lifecycle
│   ├── useFormSchema.ts          — schema loading & parsing
│   └── useFieldDataSource.ts     — async data fetching for fields
│
├── services/            # API calls, abstracted per concern
│   ├── formSchemaService.ts      — fetch form schema from server
│   ├── formSubmitService.ts      — submit form payload
│   ├── formDraftService.ts       — persist/load drafts
│   └── masterDataService.ts      — fetch master data for select/autocomplete
│
├── utils/               # Pure engine logic (no React deps)
│   ├── conditionalEngine.ts      — evaluates show/hide rules
│   ├── validationEngine.ts       — runs all validators
│   ├── schemaParser.ts           — normalises incoming schema
│   ├── formTransformer.ts        — transforms form values for API
│   ├── fieldMapper.ts            — maps schema field types → components
│   ├── cacheManager.ts           — caches master data lookups
│   └── defaultValues.ts          — generates default values from schema
│
├── validators/          # Pure validation functions
│   ├── required.ts
│   ├── email.ts
│   ├── phone.ts
│   ├── date.ts
│   ├── minMax.ts
│   ├── pattern.ts
│   └── custom.ts
│
├── types/               # TypeScript interfaces
│   ├── form.types.ts
│   ├── field.types.ts
│   ├── schema.types.ts
│   ├── validation.types.ts
│   └── common.types.ts
│
├── constants/           # Enums and constants
│   ├── fieldTypes.ts
│   ├── operatorTypes.ts
│   └── validationTypes.ts
│
└── styles/              # Theme tokens and form styles
    ├── formStyles.ts
    └── theme.ts
```

---

## Key design decisions

### 1. Schema-driven
Forms are defined as data (JSON or TypeScript objects), not as JSX. The engine reads the schema and renders the entire form — sections, fields, layout, validation rules, conditional logic — without any imperative code in the consuming module.

### 2. Conditional logic engine
`conditionalEngine.ts` evaluates operator-based rules (equals, not-equals, contains, greater-than, etc.) to show/hide or enable/disable fields at runtime. Rules are declared in the schema, not in component code.

### 3. Computed fields
`useComputedFields` + `ComputedField.tsx` auto-calculate field values from expressions over other field values — useful for billing totals, age from DOB, salary components, etc.

### 4. Dependent dropdowns
`useDependentFields` handles cascading select chains (e.g. State → District → City) with automatic re-fetch and cache invalidation when a parent field changes.

### 5. Draft persistence
`useFormDraft` + `formDraftService` allow partially filled forms to be saved and restored, preventing data loss in multi-step or long forms.

### 6. Cache layer
`cacheManager.ts` prevents redundant API calls for master data (departments, wards, cities, etc.) that are reused across many forms in the same session.

### 7. Pure utils
All engine logic (`conditionalEngine`, `validationEngine`, `schemaParser`, `formTransformer`, `fieldMapper`) lives in plain TypeScript with no React dependencies — making them independently testable.

---

## Supported field types

| Category | Fields |
|----------|--------|
| Text input | TextField, TextareaField, EmailField, PasswordField, PhoneField |
| Numeric | NumberField, CurrencyField, PercentageField, SliderField |
| Selection | SelectField, MultiSelectField, RadioField, CheckboxField, CheckboxGroupField |
| Date & time | DateField, DateTimeField, TimeField |
| Smart fields | AutocompleteField, ComputedField, HiddenField |
| Rich input | AddressField, ColorPickerField, RatingField, SignatureField |
| File & media | FileUploadField, ImageUploadField |

---

## Microservice-Based Modular Architecture

The Dynamic Form Engine is designed for a microservice-based HMS ecosystem where each business domain is organized as an independent service with its own frontend module(s).

### Microservices and modules

- `reception-service` → `modules/reception` — patient registration, appointments, visitor management
- `opd-service` → `modules/opd` — consultation forms, prescriptions, doctor workflow
- `ipd-service` → `modules/ipd` — admission, discharge, bed allocation, inpatient workflow
- `mediclaim-service` → `modules/mediclaim` — insurance, TPA approval, claim processing
- `hospital-service` → `modules/hospital` — billing head, service master, hospital setup
- `hr-service` → `modules/hr` — employee management, salary, onboarding
- `lab-diagnostic-service` → `modules/lab-diagnostic` — pathology, radiology, diagnostics workflow
- `pharmacy-store-service` → `modules/pharmacy-store` — medicine inventory and stock management
- `blood-bank-service` → `modules/blood-bank` — donor and blood inventory management
- `ambulance-service` → `modules/ambulance` — ambulance booking and trip management
- `dashboard-service` → `modules/dashboard` — analytics, reports, dynamic filters
- `admin-service` → `modules/admin` — RBAC, permissions, system configuration
- `emr-ehr-service` → `modules/emr-ehr` — patient medical records and history
- `notification-service` → `modules/notification` — SMS, email, WhatsApp workflows
- `finance-service` → `modules/finance` — billing, payments, accounting workflows
- `telemedicine-service` → `modules/telemedicine` — virtual consultation workflows

### Architecture principles

- Each service is a separate business domain.
- Each service contains one or more frontend modules.
- Shared form engine is reused across all modules.
- Common hooks, validators, and utilities stay centralized.
- The structure supports scalable enterprise healthcare applications.

---

## Tech stack

- React 18 + TypeScript
- Vite (build tool)
- Hooks-based architecture (no Redux for form state)
- REST API integration via Axios (through services layer)

