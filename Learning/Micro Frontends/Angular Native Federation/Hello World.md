---
dg-publish: true
---
````markdown
## Software Versions

```bash
Angular CLI: 18.2.12
Node: 22.1.0
Package Manager: npm 10.7.0
OS: darwin arm64
````

### Step 1: Set Up Angular Applications

1. **Create Angular Applications**: Run the Angular CLI commands to generate your apps:
    
    ```bash
    ng new host --routing --style=css
    ng new health --routing --style=css
    ng new retirement --routing --style=css
    ```
    

### Step 2: Install Native Federation

Run the following command in each of the projects:

```bash
npm install @angular-architects/native-federation --save-dev
```

### Step 3: Execute Automated Setup for Host

```bash
ng generate @angular-architects/native-federation:init --port 4200 --type dynamic-host
```

This command creates the following files:

1. **federation.manifest.json**
    
    ```json
    {
      "mfe1": "http://localhost:4201/remoteEntry.json",
      "mfe2": "http://localhost:4202/remoteEntry.json"
    }
    ```
    
2. **federation.config.js**
    
    ```javascript
    const { withNativeFederation, shareAll } = require('@angular-architects/native-federation/config');
    
    module.exports = withNativeFederation({
      shared: {
        ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),
      },
    
      skip: [
        'rxjs/ajax',
        'rxjs/fetch',
        'rxjs/testing',
        'rxjs/webSocket',
        // Add further packages you don't need at runtime
      ],
    
      // Please read our FAQ about sharing libs:
      // https://shorturl.at/jmzH0
    });
    ```
    
3. **tsconfig.federation.json**
    
    ```json
    {
      "extends": "./tsconfig.json",
      "compilerOptions": {
        "outDir": "./out-tsc/app",
        "types": []
      },
      "files": [
        "src/main.ts"
      ],
      "include": [
        "src/**/*.d.ts"
      ]
    }
    ```
    

### Step 4: Execute Setup for Remotes

For remote 1:

```bash
ng generate @angular-architects/native-federation:init --port 4201 --type remote
```

For remote 2:

```bash
ng generate @angular-architects/native-federation:init --port 4202 --type remote
```

For remote n:

```bash
ng generate @angular-architects/native-federation:init --port [n] --type remote
```

The files created for the remotes are the same except for the **federation.config.js**. By default, the `name` will inherit the name of your project. I've changed the name to match the default naming scheme of federation for `mfe1`, `mfe2`, etc., corresponding to your n applications.

```javascript
const { withNativeFederation, shareAll } = require('@angular-architects/native-federation/config');

module.exports = withNativeFederation({
  name: 'mfe1',

  exposes: {
    './Component': './src/app/app.component.ts',
  },

  shared: {
    ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),
  },

  skip: [
    'rxjs/ajax',
    'rxjs/fetch',
    'rxjs/testing',
    'rxjs/webSocket',
    // Add further packages you don't need at runtime
  ],

  // Please read our FAQ about sharing libs:
  // https://shorturl.at/jmzH0
});
```

### Step 5: Configure Manifest for Host

The manifest file contains the "links" to the `remoteEntry` files. This link is what you'll bind to later on when we route to the pages.

```json
{
  "mfe1": "http://localhost:4201/remoteEntry.json",
  "mfe2": "http://localhost:4202/remoteEntry.json"
}
```

_I'm assuming for now this name must match the exported name within the remote applications._

**Correction**: It does not need to match; only the URL needs to match.

### Step 6: Configure Router

**app.routes.ts**

```typescript
import { Routes } from '@angular/router';
import { loadRemoteModule } from '@angular-architects/native-federation';

export const APP_ROUTES: Routes = [
  {
    path: 'mfe1',
    loadComponent: () =>
      loadRemoteModule('mfe1', './Component').then((m) => m.AppComponent),
  },
  {
    path: 'mfe2',
    loadComponent: () =>
      loadRemoteModule('mfe2', './Component').then((m) => m.AppComponent),
  },
];
```

### Step 7: Add Navigation to Root App

```html
<nav>
  <ul>
    <li><a href="#home">Home</a></li>
    <li><a href="#about">About</a></li>
    <li><a routerLink="/mfe1">Retirement</a></li>
    <li><a routerLink="/mfe2">Health</a></li>
  </ul>
</nav>
<router-outlet></router-outlet>
```