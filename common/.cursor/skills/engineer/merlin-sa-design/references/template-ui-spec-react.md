## 🧱 React Architecture: Feature-Sliced Design

> ใช้ Feature-Sliced Design เป็น architecture หลักของ frontend
> เป้าหมายคือแยก responsibility ให้ชัด ลด coupling และป้องกันไม่ให้ page/component/store กลายเป็น God Object

---

## 1. Architecture Principle

### Core Rules

* ✅ แบ่ง code ตาม business capability ก่อน technical type
* ✅ ใช้ layer เพื่อควบคุม dependency direction
* ✅ แต่ละ feature ต้องมี boundary ของตัวเอง
* ✅ expose เฉพาะ public API ผ่าน `index.ts`
* ❌ ห้าม import ข้าม internal folder ของ feature อื่นโดยตรง
* ❌ ห้ามสร้าง `components/`, `hooks/`, `utils/`, `store/` กองกลางแบบไร้ ownership
* ❌ ห้ามให้ page component ถือ business logic ทั้งหมด
* ❌ ห้ามให้ global store เก็บ state ทุกอย่างของหน้า

---

## 2. Feature-Sliced Design Layers

### Recommended Layers

```txt
src/
  app/
  pages/
  widgets/
  features/
  entities/
  shared/
```

| Layer      | Purpose                                          | Example                                            |
| ---------- | ------------------------------------------------ | -------------------------------------------------- |
| `app`      | App bootstrap, providers, routing, global config | router, query client, redux provider               |
| `pages`    | Route-level composition                          | `ConfigManagementPage`                             |
| `widgets`  | Large UI blocks composed from features/entities  | `ConfigListWidget`, `ConfigEditorPanel`            |
| `features` | User action / business use case                  | `create-config`, `update-config`, `publish-config` |
| `entities` | Core business domain model                       | `config`, `user`, `tenant`                         |
| `shared`   | Reusable technical foundation                    | UI kit, API client, formatter, constants           |

---

## 3. Dependency Rule

### Allowed Direction

```txt
app
 ↓
pages
 ↓
widgets
 ↓
features
 ↓
entities
 ↓
shared
```

### Import Rules

| From       | Can Import                                 |
| ---------- | ------------------------------------------ |
| `app`      | pages, widgets, features, entities, shared |
| `pages`    | widgets, features, entities, shared        |
| `widgets`  | features, entities, shared                 |
| `features` | entities, shared                           |
| `entities` | shared                                     |
| `shared`   | nothing from upper layers                  |

### Forbidden Examples

```ts
// ❌ ห้าม feature import page
import { ConfigManagementPage } from '@/pages/config-management';

// ❌ ห้าม entity import feature
import { CreateConfigButton } from '@/features/create-config';

// ❌ ห้าม import internal file จาก feature อื่น
import { validateForm } from '@/features/create-config/model/validateForm';

// ✅ ให้ import ผ่าน public API
import { CreateConfigButton } from '@/features/create-config';
```

---

## 4. Suggested Folder Structure

```txt
src/
  app/
    providers/
      AppProvider.tsx
      QueryProvider.tsx
      StoreProvider.tsx
      PermissionProvider.tsx
    routes/
      router.tsx
    store/
      rootStore.ts
      rootReducer.ts

  pages/
    config-management/
      ui/
        ConfigManagementPage.tsx
      index.ts

  widgets/
    config-list/
      ui/
        ConfigListWidget.tsx
      model/
        useConfigListParams.ts
      index.ts

    config-editor/
      ui/
        ConfigEditorWidget.tsx
      index.ts

  features/
    create-config/
      ui/
        CreateConfigButton.tsx
        CreateConfigDrawer.tsx
      model/
        createConfig.schema.ts
        useCreateConfigForm.ts
        useCreateConfigMutation.ts
      index.ts

    update-config/
      ui/
        UpdateConfigDrawer.tsx
      model/
        updateConfig.schema.ts
        useUpdateConfigMutation.ts
      index.ts

    publish-config/
      ui/
        PublishConfigButton.tsx
        PublishConfigConfirmDialog.tsx
      model/
        usePublishConfigMutation.ts
      index.ts

  entities/
    config/
      api/
        config.api.ts
      model/
        config.type.ts
        config.schema.ts
        config.mapper.ts
      ui/
        ConfigStatusBadge.tsx
        ConfigValuePreview.tsx
      index.ts

  shared/
    api/
      httpClient.ts
      errorMapper.ts
    ui/
      Button.tsx
      Dialog.tsx
      DataTable.tsx
      FormField.tsx
    lib/
      date.ts
      number.ts
      object.ts
    config/
      env.ts
```

---

## 5. State Ownership Decision

> ก่อนสร้าง Context / Redux / Store ต้องตอบให้ได้ก่อนว่า state นี้เป็นของใคร

| State Type                                  | Example                                                                         | Owner               | Recommended Tool                |
| ------------------------------------------- | ------------------------------------------------------------------------------- | ------------------- | ------------------------------- |
| Local UI state                              | drawer open, selected tab, hover state                                          | Component           | `useState`                      |
| Local form state                            | create config form                                                              | Feature             | React Hook Form / local reducer |
| URL state                                   | page, limit, filter, search keyword                                             | Page / Widget       | URL query params                |
| Server state                                | config list, config detail                                                      | Entity API layer    | React Query / SWR               |
| Cross-tree static-ish state                 | theme, locale, auth user, permission                                            | App Provider        | React Context                   |
| Complex client state shared across features | selected workspace, draft builder state, multi-step wizard shared across routes | App / Feature Store | Redux Toolkit / Zustand         |
| Derived state                               | filtered list, computed label, canPublish                                       | Selector / hook     | `useMemo`, selector function    |
| Temporary interaction state                 | modal open, focused row, optimistic draft text                                  | Component / Feature | local state                     |

---

## 6. When to Use Local State

ใช้ `useState` / `useReducer` เมื่อ state อยู่ใกล้ component และไม่มีเหตุผลให้ global

### Good Use Cases

* เปิด/ปิด drawer
* เปิด/ปิด dialog
* selected row ภายใน table เดียว
* active tab ภายใน widget
* temporary input state
* UI-only state ที่ refresh แล้วหายได้

### Rule

```md
ถ้า state ใช้แค่ component เดียว หรือ component ลูกใกล้ ๆ กัน  
ให้เริ่มจาก local state ก่อนเสมอ
```

### Example

```tsx
function ConfigListWidget() {
  const [selectedConfigId, setSelectedConfigId] = useState<string | null>(null);
  const [isCreateOpen, setCreateOpen] = useState(false);

  return (
    <>
      <ConfigTable onSelect={setSelectedConfigId} />
      <CreateConfigDrawer open={isCreateOpen} onOpenChange={setCreateOpen} />
    </>
  );
}
```

---

## 7. When to Use React Context

ใช้ Context เมื่อข้อมูลต้องถูกอ่านจากหลาย component ใน subtree และไม่ควรถูกส่งผ่าน props หลายชั้น

### Good Use Cases

* Auth session
* Permission context
* Theme
* Locale
* Feature flag
* Current tenant / workspace
* Read-only page-level configuration

### Avoid Context When

* state เปลี่ยนบ่อยมาก
* state เป็น form ขนาดใหญ่
* state เป็น table data
* state เป็น server cache
* state เป็น business workflow ที่ซับซ้อน
* context value ใหญ่จน component re-render กว้างเกินจำเป็น

### Context Design Rules

* แยก context ตาม responsibility
* ห้ามสร้าง `AppContext` ที่รวมทุกอย่าง
* แยก read context กับ action context เมื่อจำเป็น
* memoize value ที่เป็น object/function
* expose ผ่าน custom hook เท่านั้น
* throw error ถ้า hook ถูกใช้นอก provider

### Bad Example

```tsx
// ❌ God Context
type AppContextValue = {
  user: User;
  permissions: string[];
  configs: ConfigItem[];
  selectedConfig: ConfigItem | null;
  createConfig: () => void;
  updateConfig: () => void;
  deleteConfig: () => void;
  theme: Theme;
  locale: string;
};
```

### Good Example

```tsx
type PermissionContextValue = {
  permissions: string[];
  can: (permission: string) => boolean;
};

const PermissionContext = createContext<PermissionContextValue | null>(null);

export function PermissionProvider({
  permissions,
  children,
}: {
  permissions: string[];
  children: React.ReactNode;
}) {
  const value = useMemo(
    () => ({
      permissions,
      can: (permission: string) => permissions.includes(permission),
    }),
    [permissions],
  );

  return (
    <PermissionContext.Provider value={value}>
      {children}
    </PermissionContext.Provider>
  );
}

export function usePermission() {
  const context = useContext(PermissionContext);

  if (!context) {
    throw new Error('usePermission must be used within PermissionProvider');
  }

  return context;
}
```

---

## 8. When to Use Redux / Global Store

ใช้ Redux หรือ global store เมื่อ state มี lifecycle ยาว, ถูกใช้ข้าม feature, มี transition ซับซ้อน และต้อง debug/persist/trace ได้

### Good Use Cases

* Auth/session state ที่ต้องใช้ทั้ง app
* Current tenant / workspace
* Multi-step workflow ข้าม route
* Draft state ที่ต้อง persist ก่อน submit
* Complex permission/session state
* Global notification queue
* Feature state ที่หลาย widget ต้องอ่าน/เขียนร่วมกัน
* State ที่ต้อง undo/redo
* State ที่ต้อง sync กับ local storage

### Avoid Redux / Global Store When

* เป็น server data ที่ fetch จาก API
* เป็น form state เฉพาะ drawer/modal
* เป็น loading state ของ request เดียว
* เป็น selected row เฉพาะ table เดียว
* เป็น state ที่ component เดียวใช้
* เป็น derived state ที่คำนวณจาก state อื่นได้

### Store Design Rules

* แบ่ง slice ตาม domain / feature ไม่ใช่ตาม UI component
* reducer ต้องเล็กและ predictable
* action ต้องสื่อ business event ไม่ใช่ UI implementation detail
* selector ต้องอยู่ใกล้ slice
* ห้าม component access raw store shape โดยตรง
* ห้ามสร้าง root state ที่เป็น dumping ground
* ห้ามเก็บ non-serializable object ถ้าใช้ Redux
* server data ควรอยู่ใน React Query / API cache ไม่ใช่ Redux

---

## 9. Store Slice Template

### Slice Structure

```txt
features/
  config-workflow/
    model/
      configWorkflow.slice.ts
      configWorkflow.selector.ts
      configWorkflow.type.ts
    index.ts
```

### Example

```ts
type ConfigWorkflowStep = 'select-type' | 'edit-value' | 'review' | 'publish';

type ConfigWorkflowState = {
  currentStep: ConfigWorkflowStep;
  draftConfigId: string | null;
  dirty: boolean;
};

const initialState: ConfigWorkflowState = {
  currentStep: 'select-type',
  draftConfigId: null,
  dirty: false,
};
```

### Action Naming

| Bad Action     | Better Action               |
| -------------- | --------------------------- |
| `setModalOpen` | `configCreationStarted`     |
| `setData`      | `draftConfigLoaded`         |
| `setStep`      | `configWorkflowStepChanged` |
| `setError`     | `configWorkflowFailed`      |
| `reset`        | `configWorkflowCancelled`   |

---

## 10. Server State Rule

> API response data ไม่ควรถูกยัดเข้า Redux โดย default

### Use React Query / SWR For

* list data
* detail data
* search result
* pagination data
* cache
* refetch
* mutation
* optimistic update
* stale data management

### Example

```ts
export function useConfigList(params: ConfigListParams) {
  return useQuery({
    queryKey: ['configs', params],
    queryFn: () => configApi.getList(params),
  });
}

export function useCreateConfigMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: configApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['configs'] });
    },
  });
}
```

---

## 11. Component Design Rule

### Component Categories

| Type      | Responsibility            |         Can Fetch API? |     Can Access Store? |
| --------- | ------------------------- | ---------------------: | --------------------: |
| Page      | route composition         |                 Rarely |          Yes, lightly |
| Widget    | compose feature/entity UI |         Yes, via hooks |        Yes, if needed |
| Feature   | user action               | Yes, via feature hooks | Yes, if feature-owned |
| Entity UI | display domain object     |                     No |                    No |
| Shared UI | generic UI only           |                     No |                    No |

---

## 12. Prevent God Component

### God Component Symptoms

* component ยาวเกิน `250-300` lines
* มีทั้ง fetch, form, validation, permission, table, modal, toast ในไฟล์เดียว
* มี `useState` มากกว่า `7-10` ตัว
* มี `useEffect` หลายตัวที่ผูก logic คนละเรื่อง
* มี conditional rendering ซ้อนกันหลายชั้น
* pass props ลงลูกเกิน `5-7` props ต่อ component
* component รู้รายละเอียด API response มากเกินไป

### Refactor Rule

| Problem                    | Refactor To                                               |
| -------------------------- | --------------------------------------------------------- |
| Fetch logic อยู่ใน page    | `entities/<entity>/api` + query hook                      |
| Form logic อยู่ใน page     | `features/<action>/model/useXForm.ts`                     |
| Table columns ใหญ่เกิน     | `widgets/<name>/model/columns.tsx`                        |
| Permission logic กระจาย    | `app/providers/PermissionProvider` + `usePermission`      |
| Modal logic ซ้ำ            | feature-level dialog component                            |
| Mapping API response ใน UI | `entities/<entity>/model/mapper.ts`                       |
| Too many props             | compose component closer to owner, or create feature hook |

---

## 13. Prevent God Store

### God Store Symptoms

* มี slice ชื่อ `appSlice`, `commonSlice`, `globalSlice` ที่เก็บทุกอย่าง
* UI state เฉพาะหน้าอยู่ใน global store
* server response ทั้งก้อนถูก copy เข้า Redux
* selector มี business logic ยาวมาก
* action ชื่อ generic เช่น `setData`, `setState`, `updateValue`
* หลาย feature dispatch action ของ slice เดียวกันแบบไม่ชัด ownership

### Store Boundary Rule

| State                         | Should Live In               |
| ----------------------------- | ---------------------------- |
| Drawer open/close             | Component                    |
| Form values                   | Feature form hook            |
| Config list from API          | React Query                  |
| Current authenticated user    | Auth provider / store        |
| Current tenant                | App context / store          |
| Config creation workflow step | Feature store                |
| Config publish confirmation   | Feature local state          |
| Permission list               | Permission provider          |
| Toast queue                   | Global store or app provider |

---

## 14. Public API Rule

ทุก slice/feature/entity ต้อง expose ผ่าน `index.ts` เท่านั้น

### Good

```ts
import { ConfigStatusBadge, useConfigList } from '@/entities/config';
import { CreateConfigButton } from '@/features/create-config';
```

### Bad

```ts
import { ConfigStatusBadge } from '@/entities/config/ui/ConfigStatusBadge';
import { useConfigList } from '@/entities/config/api/useConfigList';
import { CreateConfigButton } from '@/features/create-config/ui/CreateConfigButton';
```

### Example `index.ts`

```ts
export { ConfigStatusBadge } from './ui/ConfigStatusBadge';
export { ConfigValuePreview } from './ui/ConfigValuePreview';

export type { ConfigItem, ConfigStatus, ConfigValueType } from './model/config.type';

export { useConfigList } from './api/useConfigList';
export { useConfigDetail } from './api/useConfigDetail';
```

---

## 15. State Design Checklist

ก่อนเพิ่ม state ใหม่ ต้องตอบ checklist นี้

* [ ] State นี้เป็น local, URL, server, context หรือ global client state?
* [ ] State นี้มี owner ชัดเจนหรือยัง?
* [ ] State นี้จำเป็นต้องอยู่ global จริงไหม?
* [ ] Refresh page แล้ว state นี้ควรหายหรืออยู่ต่อ?
* [ ] State นี้ถูกใช้ข้าม route หรือไม่?
* [ ] State นี้ถูกใช้ข้าม feature หรือไม่?
* [ ] State นี้ derive จากข้อมูลอื่นได้ไหม?
* [ ] State นี้ควรถูก persist ไหม?
* [ ] State นี้ต้อง debug ด้วย Redux DevTools ไหม?
* [ ] State นี้ทำให้ component re-render กว้างเกินไปไหม?

---

## 16. Feature Design Checklist

ก่อนเพิ่ม feature ใหม่ ต้องตอบ checklist นี้

* [ ] Feature นี้แทน user action อะไร?
* [ ] Feature นี้อยู่ layer `features` จริงไหม หรือควรเป็น `widget` / `entity`?
* [ ] Feature นี้ใช้ entity อะไรบ้าง?
* [ ] Feature นี้ expose public API อะไร?
* [ ] Feature นี้มี model/form/schema ของตัวเองหรือไม่?
* [ ] Feature นี้ต้องใช้ global store จริงไหม?
* [ ] Feature นี้ผูกกับ route มากเกินไปไหม?
* [ ] Feature นี้ reuse ได้โดยไม่ import จาก page หรือไม่?

---

## 17. UI Spec Requirement: State & Architecture Section

ทุก UI Spec ของ React feature ต้องมี section นี้

### State Ownership

| State                | Type                | Owner      | Tool            | Reason                                |
| -------------------- | ------------------- | ---------- | --------------- | ------------------------------------- |
| `searchKeyword`      | URL state           | Page       | query params    | ต้อง shareable และ refresh แล้วคงอยู่ |
| `configList`         | Server state        | Entity API | React Query     | เป็นข้อมูลจาก backend                 |
| `isCreateDrawerOpen` | Local UI state      | Widget     | `useState`      | ใช้แค่ในหน้านี้                       |
| `createConfigForm`   | Form state          | Feature    | React Hook Form | เป็น state ของ create flow            |
| `permissions`        | Cross-tree state    | App        | Context         | ใช้ควบคุม rendering หลายจุด           |
| `currentTenant`      | Global client state | App        | Store / Context | ใช้ทั้งระบบ                           |

### Store / Context Decision

| Question                 | Answer          |
| ------------------------ | --------------- |
| Need Context?            | `Yes / No`      |
| Context name             | `<ContextName>` |
| Need Redux/global store? | `Yes / No`      |
| Store slice name         | `<sliceName>`   |
| Why not local state?     | `<reason>`      |
| Why not server cache?    | `<reason>`      |
| Persistence required?    | `Yes / No`      |
| Cross-route usage?       | `Yes / No`      |

---

## 18. Acceptance Criteria Addition

* [ ] Feature follows Feature-Sliced Design structure
* [ ] Dependency direction follows FSD layer rule
* [ ] Feature exposes public API through `index.ts`
* [ ] Page component only composes widgets/features and does not contain heavy business logic
* [ ] Server state is not stored in Redux/global store by default
* [ ] Local UI state stays local unless there is a strong reason
* [ ] Context is scoped and does not become `AppContext` dumping ground
* [ ] Store slices are separated by domain/feature
* [ ] Components do not import internal files from other slices
* [ ] No God Component or God Store symptoms are present
