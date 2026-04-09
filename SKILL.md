---
name: dhis2-capture-field-plugin
description: >
  Build DHIS2 Capture field form plugins following the official reference implementation
  structure. Use when the user asks to create DHIS2 Tracker plugins, Capture plugins,
  or form field plugins — including validation, data lookup, external API calls, or
  field value calculation in DHIS2 Capture data entry forms.
---

# DHIS2 Tracker Plugin Builder Skill

## Purpose
This skill helps you build DHIS2 Capture Field Form Plugins following the exact structure of the official reference implementation. Use this when users ask to create DHIS2 plugins, Tracker plugins, Capture plugins, or form field plugins.

## When to Use This Skill
- User asks to build a DHIS2 plugin
- User mentions "Tracker", "Capture", "field form plugin", or "data entry plugin"
- User needs to validate or process data in DHIS2 forms
- User wants to extend DHIS2 Capture app functionality

## Critical Requirements

### EXACT Structure (Non-Negotiable)
The plugin MUST match the official reference structure from https://github.com/dhis2/reference-form-field-plugin

```
project-name/
├── capture-plugin/
│   ├── src/
│   │   ├── App.js              # Standalone app entry point (REQUIRED)
│   │   ├── Plugin.js           # Plugin entry point (REQUIRED)
│   │   ├── [ComponentFolder]/  # Your components
│   │   │   ├── Component.js
│   │   │   └── Component.module.css
│   │   └── lib/                # Library/utility code
│   │       └── yourLogic.js
│   ├── d2.config.js            # MUST have BOTH app and plugin entryPoints
│   └── package.json
├── package.json                # Root workspace
└── README.md
```

### Required Files

#### 1. Root package.json
```json
{
  "name": "project-name",
  "private": true,
  "workspaces": ["capture-plugin"],
  "scripts": {
    "build": "yarn workspace capture-plugin build",
    "start": "yarn workspace capture-plugin start",
    "test": "yarn workspace capture-plugin test"
  },
  "devDependencies": {
    "@dhis2/cli-app-scripts": "^11.7.0"
  },
  "resolutions": {
    "react": "18.3.1",
    "react-dom": "18.3.1"
  }
}
```

#### 2. capture-plugin/package.json
```json
{
  "name": "capture-plugin",
  "version": "1.0.0",
  "description": "Your plugin description",
  "license": "BSD-3-Clause",
  "private": true,
  "scripts": {
    "build": "d2-app-scripts build",
    "start": "d2-app-scripts start",
    "test": "d2-app-scripts test"
  },
  "devDependencies": {
    "@dhis2/cli-app-scripts": "^11.7.0"
  },
  "dependencies": {
    "@dhis2/app-runtime": "^3.11.2",
    "@dhis2/ui": "^9.11.3",
    "prop-types": "^15.8.1",
    "react": "18.3.1",
    "react-dom": "18.3.1"
  }
}
```

**IMPORTANT**: 
- React versions MUST be exact (no `^`)
- Use these EXACT dependency versions
- Include resolutions in root package.json

#### 3. capture-plugin/d2.config.js
```javascript
const config = {
    type: 'app',
    name: 'capture-plugin',
    title: 'Your Plugin Title',
    
    entryPoints: {
        app: './src/App.js',      // REQUIRED - even if simple
        plugin: './src/Plugin.js', // REQUIRED - main plugin code
    },
    
    pluginType: 'CAPTURE',
}

module.exports = config
```

**CRITICAL**: BOTH `app` and `plugin` entry points are REQUIRED. Missing `app` causes build errors.

#### 4. capture-plugin/src/App.js
```javascript
import React from 'react'
import { CenteredContent, CircularLoader } from '@dhis2/ui'

const App = () => {
    return (
        <CenteredContent>
            <CircularLoader />
        </CenteredContent>
    )
}

export default App
```

This is a minimal standalone app. Always include it even if users won't use it.

#### 5. capture-plugin/src/Plugin.js
```javascript
import React from 'react'
import PropTypes from 'prop-types'
import { YourComponent } from './YourFolder/YourComponent.js'

const Plugin = (props) => {
    const { values = {}, viewMode = false } = props

    // Access field values using IdFromPlugin aliases
    const fieldValue = values['yourFieldAlias'] || ''

    if (viewMode) {
        return (
            <div style={{ padding: '16px', color: '#666', minHeight: '60px', width: '100%' }}>
                <p style={{ margin: 0 }}>
                    <strong>Your Plugin Title</strong>
                    <br />
                    Field: {fieldValue || 'Not set'}
                </p>
            </div>
        )
    }

    return <YourComponent fieldValue={fieldValue} />
}

Plugin.propTypes = {
    values: PropTypes.object,
    errors: PropTypes.object,
    warnings: PropTypes.object,
    fieldsMetadata: PropTypes.object,
    setFieldValue: PropTypes.func,
    setContextFieldValue: PropTypes.func,
    viewMode: PropTypes.bool,
    formSubmitted: PropTypes.bool,
}

export default Plugin
```

## Plugin Props Interface

The plugin receives these props from DHIS2 Capture:

```javascript
{
    values: Record<string, any>,              // Field values (key = IdFromPlugin)
    errors: Record<string, string[]>,         // Validation errors
    warnings: Record<string, string[]>,       // Validation warnings
    fieldsMetadata: Record<string, object>,   // Field metadata
    setFieldValue: (props) => void,           // Update field values
    setContextFieldValue: (props) => void,    // Update context (date, geometry)
    viewMode: boolean,                        // Read-only mode flag
    formSubmitted: boolean,                   // Form submission state
}
```

### Accessing Field Values
```javascript
// In Plugin.js
const { values = {} } = props
const myValue = values['myFieldAlias']  // Uses IdFromPlugin from field mapping
```

### Setting Field Values
```javascript
// In your component
setFieldValue({
    fieldId: 'myFieldAlias',  // IdFromPlugin
    value: 'new value',
    options: { valid: true, touched: true }
})
```

## DHIS2 UI Components

Always use DHIS2 UI components for consistency:

```javascript
import { Button, NoticeBox, Input, Field } from '@dhis2/ui'

// Button
<Button onClick={handleClick} primary>Click Me</Button>

// Notice Box
<NoticeBox warning title="Warning:">Message here</NoticeBox>
<NoticeBox error title="Error:">Error message</NoticeBox>
<NoticeBox info title="Info:">Info message</NoticeBox>

// Input Field
<Field label="Label">
    <Input value={value} onChange={(e) => setValue(e.value)} />
</Field>
```

## File Naming Conventions

- **Use .js extension** (not .jsx or .tsx)
- **Use PascalCase** for component files: `MyComponent.js`
- **Use camelCase** for utility files: `myUtility.js`
- **Use .module.css** for CSS modules: `MyComponent.module.css`

## CSS Modules

```javascript
// Component.js
import styles from './Component.module.css'

<div className={styles.container}>...</div>
```

```css
/* Component.module.css */
.container {
    margin: 16px 0;
    padding: 12px;
    min-height: 80px;
    width: 100%;
    box-sizing: border-box;
}
```

## Common Patterns

### 1. Validation Pattern
```javascript
// lib/validationLogic.js
export const validateData = (value1, value2) => {
    if (!value1 || !value2) return 'Both fields required'
    // Your validation logic
    return null  // null = no error
}

// Component.js
import { validateData } from '../lib/validationLogic.js'

const handleValidate = () => {
    const error = validateData(field1, field2)
    if (error) {
        setMessage(error)
        setShowError(true)
    }
}
```

### 2. Lookup Table Pattern
```javascript
// lib/lookupTable.js
export const LOOKUP_TABLE = [
    { key1: 'value1', key2: 'value2', result: 'message' },
    // ...
]

export const findMatch = (key1, key2) => {
    const entry = LOOKUP_TABLE.find(
        item => item.key1 === key1 && item.key2 === key2
    )
    return entry ? entry.result : null
}
```

### 3. DHIS2 Data Engine Pattern

Prefer the DHIS2 data engine over raw `fetch` for any DHIS2 API calls — it handles authentication and base URL automatically.

```javascript
// Component.js
import { useDataEngine } from '@dhis2/app-runtime'

const MyComponent = () => {
    const engine = useDataEngine()
    const [loading, setLoading] = useState(false)

    const handleFetch = async () => {
        setLoading(true)
        try {
            const { result } = await engine.query({
                result: {
                    resource: 'trackedEntityInstances',
                    params: { ou: 'orgUnitId', program: 'programId' },
                },
            })
            // handle result
        } catch (error) {
            console.error('API error:', error)
        } finally {
            setLoading(false)
        }
    }
}
```

Use raw `fetch` only for **external** (non-DHIS2) APIs:

```javascript
// lib/apiService.js
export const fetchExternalData = async (param) => {
    try {
        const response = await fetch(`https://api.example.com/data/${param}`)
        return await response.json()
    } catch (error) {
        console.error('API error:', error)
        return null
    }
}
```

## Build & Deploy Process

### Build Instructions to Include in README

```markdown
## Installation & Build

### 1. Install Dependencies
```bash
yarn install --frozen-lockfile
```

### 2. Build the Plugin
```bash
yarn build
```

The built plugin will be in `capture-plugin/build/bundle/`

### 3. Upload to DHIS2
1. Go to **App Management** in DHIS2
2. Click **Upload App**
3. Select `capture-plugin/build/bundle/plugin.html`
4. Click **Install**
```

## Configuration in DHIS2

### Field Mapping
Users configure field mapping in the **Tracker Plugin Configurator** app:

```json
{
  "fieldMap": [
    {
      "IdFromApp": "ABC123xyz",        // DHIS2 data element ID
      "IdFromPlugin": "myFieldAlias",  // Used in plugin code
      "objectType": "DataElement"      // or "TrackedEntityAttribute"
    }
  ]
}
```

**Key Points**:
- `IdFromPlugin` is what you use in `values['myFieldAlias']`
- Tell users to use exact aliases (no "Value" suffix)
- Users configure this through UI, not code

## Troubleshooting Common Issues

### 1. "Cannot resolve entrypoint ./src/App.js"
**Cause**: Missing App.js file
**Fix**: Ensure App.js exists with proper export

### 2. "Invalid hook call" or React errors
**Cause**: Multiple React versions
**Fix**: Ensure resolutions in root package.json and exact versions in dependencies

### 3. Build validation errors
**Fix**: Add to build instructions:
```bash
cd capture-plugin
d2-app-scripts build --no-verify
```

### 4. Plugin too small / shrinking
**Fix**: Add to CSS:
```css
.container {
    min-height: 80px;
    width: 100%;
    box-sizing: border-box;
}
```

### 5. "Cannot read properties of undefined"
**Fix**: Always use default values:
```javascript
const { values = {}, viewMode = false } = props
```

## Essential Files to Include

Always create:
1. ✅ `.gitignore` - Standard Node.js gitignore
2. ✅ `README.md` - Installation, build, and configuration instructions
3. ✅ `LICENSE` - BSD-3-Clause (matches DHIS2)
4. ✅ Both `package.json` files
5. ✅ `d2.config.js` with both entry points
6. ✅ Both `App.js` and `Plugin.js`

## README Template

```markdown
# [Plugin Name] - DHIS2 Capture Field Form Plugin

[Brief description]

## Features
- [Feature 1]
- [Feature 2]

## Prerequisites
- Node.js (v18+)
- Yarn (v1.22+)
- DHIS2 (v40.5+)

## Installation
[Build instructions]

## Configuration
[Field mapping instructions]

## Usage
[How to use in DHIS2]

## Customization
[How to modify]

## License
BSD-3-Clause
```

## Questions to Ask Users

Before building, clarify:

1. **What fields does the plugin need to read?**
   - Get field names/aliases
   
2. **What does the plugin do?**
   - Validate? Calculate? Fetch data?
   
3. **What should the UI show?**
   - Buttons? Forms? Displays?
   
4. **What triggers the action?**
   - Button click? Field change? On load?

5. **Are there any external dependencies?**
   - APIs? Libraries? Data sources?

## Example Prompts This Skill Handles

- "Build a DHIS2 plugin that validates X and Y fields"
- "Create a Tracker plugin that fetches data from an API"
- "I need a Capture form field plugin to calculate Z"
- "Build a DHIS2 plugin following the reference structure"

## Output Checklist

Before delivering, ensure:
- ✅ Exact folder structure matches reference
- ✅ Both App.js and Plugin.js exist
- ✅ d2.config.js has both entry points
- ✅ Dependencies match exact versions
- ✅ Resolutions in root package.json
- ✅ CSS has proper sizing (min-height, width: 100%)
- ✅ Plugin.js has default values for props
- ✅ README includes build and configuration steps
- ✅ .gitignore excludes node_modules and build folders
- ✅ All imports use .js extensions

## Quick Reference: Minimum Valid Plugin

```
my-plugin/
├── package.json (workspace with resolutions)
├── capture-plugin/
│   ├── package.json (exact dependencies)
│   ├── d2.config.js (both entryPoints)
│   └── src/
│       ├── App.js (minimal loader)
│       ├── Plugin.js (your logic)
│       └── YourComponent/
│           ├── Component.js
│           └── Component.module.css
```

This is the absolute minimum structure that will build and work.

---

## Final Reminders

1. **Never** skip the App.js file
2. **Always** use exact React versions with resolutions
3. **Always** include both entry points in d2.config.js
4. **Always** use .js extensions (not .jsx or .tsx)
5. **Always** use DHIS2 UI components
6. **Always** handle undefined props with defaults
7. **Always** include proper CSS sizing
8. **Always** provide clear field mapping instructions

Following these rules ensures plugins build successfully and work in DHIS2!
