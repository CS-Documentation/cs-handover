# cs-documents.html — Documents Tool

## Purpose
Manage young people's (service users') documents. Staff can apply HTML templates from a central template library to a young person's file, fill in the template fields, save the completed document, and print it. Admins can also upload and delete templates.

## Access
- All staff (role: `staff` or `admin`)
- Firebase email/password auth → `staffProfiles/{uid}`
- Template deletion restricted to `admin` role only

## Layout
**Header**: Logo/nav, tab bar (Young People · Templates), user badge, sign out  
**Young People tab**: Left sidebar (YP list + search) + right document panel  
**Templates tab**: Filter bar by category + template card grid + Upload button

---

## Tab: Young People

### Left Sidebar
- Searchable list of all active `serviceUsers`, sorted alphabetically
- Each item: avatar (initials), name, clickable to select
- Active YP highlighted in brand blue

### Right Document Panel
- Shows selected YP's name + document count
- **Add Document** button → opens Template Picker modal
- Document list cards showing:
  - Category icon + tag
  - Template name
  - Created timestamp + initials, edited timestamp + initials (if applicable)
  - Actions: 📄 View/Edit · 🖨️ Quick Print

---

## Tab: Templates

### Template Cards
- Icon, category tag, name, description, field count
- Actions per card:
  - 👁️ Preview (all staff) — shows template with placeholder labels
  - 🗑️ Delete (admin only) — soft-delete (status: `archived`)
- Filter buttons: All · Care Plans · Risk Assessments · Consent · Meeting Notes · General

### Upload Template Button
Opens the Template Upload modal.

---

## Modals

### Template Picker (`templatePickerModal`)
- Search box filters by name / description
- Lists active templates as clickable rows
- Selecting a template opens the Document Editor in **edit mode** for a new document

### Template Upload (`templateUploadModal`)
- Drag-and-drop or click-to-browse for `.html` / `.htm` files
- Auto-detects fillable fields (`data-cs-field` attributes) and shows them as chips
- Fields: Template Name (required), Category (select), Description
- Collapsible template format guide (see below)
- Save stores to `documentTemplates`

### Document Viewer / Editor (`docViewerModal`)
- Full-screen modal with iframe rendering the document
- **View mode** (default for existing docs): read-only, fields shown underlined in brand blue
  - Buttons: ✏️ Edit · 🖨️ Print · ✕ Close · 🗑️ Delete document
- **Edit mode** (new docs open directly in edit mode):
  - Fields become form inputs (blue outline, light background)
  - Auto-fill fields (yp_name, yp_dob, date_today, staff_name, staff_initials) shown read-only in grey
  - Buttons: 💾 Save · 🖨️ Print · ✕ Close · 🗑️ Delete document
- Saving: reads all `data-cs-field` inputs from iframe, stores `fieldValues` in Firestore
- Closing unsaved new doc prompts confirmation

### Template Preview (`templatePreviewModal`)
- Renders template with placeholder labels shown as `[Field Label]` in italic blue

### Confirm (`confirmModal`)
- Reusable delete confirmation for both documents and templates

---

## Template Format

HTML files with fillable areas marked using `data-cs-field` attribute:

```html
<span data-cs-field="field_id" data-cs-label="Human Label" data-cs-type="text">Placeholder</span>
```

### `data-cs-type` values
| Value | Rendered as |
|---|---|
| `text` (default) | `<input type="text">` |
| `textarea` | `<textarea>` |
| `date` | `<input type="date">` |
| `number` | `<input type="number">` |

### Auto-fill Field IDs
These are populated automatically from system data and are read-only when editing:

| Field ID | Source |
|---|---|
| `yp_name` | `serviceUsers.name` |
| `yp_dob` | `serviceUsers.dob` |
| `date_today` | Current date (YYYY-MM-DD) |
| `date_created` | Current date (YYYY-MM-DD) |
| `staff_name` | `staffProfiles.name` |
| `staff_initials` | `staffProfiles.initials` |

---

## Firestore Collections

### `documentTemplates`
```js
{
  name:        string,
  description: string,
  category:    string,   // 'General' | 'Care Plans' | 'Risk Assessments' | 'Consent' | 'Meeting Notes' | 'Incident Reports'
  htmlContent: string,   // full HTML of the uploaded template file
  fields:      [{ id: string, label: string, type: string }],  // parsed from data-cs-field elements
  createdAt:   Timestamp,
  createdBy:   string,   // initials
  status:      'active' | 'archived'
}
```

### `serviceUserDocuments`
```js
{
  serviceUserId:    string,
  templateId:       string,
  templateName:     string,
  templateCategory: string,
  fieldValues:      { [fieldId: string]: string },  // filled-in data
  createdAt:        Timestamp,
  createdBy:        string,   // initials
  updatedAt:        Timestamp,
  updatedBy:        string,   // initials
  status:           'active' | 'archived'
}
```

---

## State Variables
```js
userProfile     // staffProfiles doc data
serviceUsers[]  // active service users
templates[]     // active documentTemplates
ypDocs[]        // documents for currently selected YP
selectedYPId    // currently selected service user ID
currentDocId    // doc being viewed/edited (null = new unsaved doc)
currentDocData  // full data object for current doc
currentTmplId   // template ID for current doc
isEditing       // boolean — true when in edit mode
currentTab      // 'files' | 'templates'
ypFilter        // YP search string
tmplCatFilter   // template category filter string
uploadedHTML    // raw HTML content of file being uploaded
uploadedFields  // parsed fields from upload
```

## Key Functions
| Function | Purpose |
|---|---|
| `showTab(tab)` | Switch between files / templates views |
| `selectYP(ypId)` | Load and display a young person's documents |
| `renderDocPanel()` | Render document list for selected YP |
| `openTemplatePicker()` | Open template selection modal |
| `startDocFromTemplate(templateId)` | Begin a new document from a template (edit mode) |
| `openDocument(docId)` | Open existing document in view mode |
| `toggleEditMode()` | Switch view → edit mode for current doc |
| `saveDocEdits()` | Read iframe fields, write to Firestore |
| `printCurrentDoc()` | Open print window with filled view HTML |
| `quickPrint(docId)` | Print a document directly from the list |
| `closeDocModal()` | Close viewer (prompts if unsaved new doc) |
| `confirmDeleteDoc()` | Soft-delete current document |
| `renderTemplateGrid()` | Render template cards with current category filter |
| `previewTemplate(templateId)` | Show template preview in modal |
| `confirmDeleteTemplate(templateId)` | Soft-delete a template (admin) |
| `openTemplateUpload()` | Open upload modal |
| `processUploadFile(file)` | Read HTML file, parse fields, populate upload form |
| `saveUploadedTemplate()` | Write template to Firestore |
| `buildEditHTML(html, values, yp, staff)` | Inject editable inputs into template HTML |
| `buildViewHTML(html, values, yp, forPrint)` | Inject saved values as read-only spans |
| `buildPreviewHTML(html)` | Show template with placeholder labels |
| `parseTemplateFields(html)` | Extract `data-cs-field` elements into field array |
