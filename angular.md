Angular 17+ Fundamentals
========================
https://frontendmasters.com/courses/angular-fundamentals/  
Mark (Techson) Thompson

slides  
https://static.frontendmasters.com/assets/courses/2024-01-29-angular-fundamentals/angular-fundamentals-slides.pdf

repository  
https://github.com/MarkTechson/angular-fundamentals-lessons

### Introduction
#### Angular
web framework to build scalable web apps
enterprise solution, enterprise is great at scale

### Angular Essentials
http://angular.dev/playground

#### Example
```typescript
import {Component} from '@angular/core';
import {bootstrapApplication} from '@angular/platform-browser';

@Component({
  selector: 'app-root',
  template: `
    Hello world MaciejKa!
  `,
})
export class Playground {}
```

#### Run examples
Vite is used as a local dev server
```bash
cd angular-fundamentals
npm i
ng serve 01-hello-angular
# if not a multi project workspace
ng serve
```

#### Angular.json
project configuration

#### Projects folder
This `projects/` may not be visible in a standard angular project.  
It can be used for monorepos.

#### Typescript
Angular was very typescript first for long time.  
(Google was considering own tool "a script").

#### Global styles.css
projects/01-hello-angular/src/styles.css
```css
/* You can add global styles to this file, and also import other style files */
* {
  font-family: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
    Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji",
    "Segoe UI Symbol";
}
```

#### main.ts
A bootstrap file, like index.ts in React.

projects/01-hello-angular/src/main.ts
```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, appConfig)
  .catch((err) => console.error(err));
```

#### Firebase
Angular has very nice integration with Firebase.
1. install firebase into your project
2. sign in
3. `ng deploy`

#### Angular Component
projects/01-hello-angular/src/app/app.component.ts
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, RouterOutlet],
  template: `
    <h1>If you are reading this...</h1>
    <p>Things have worked out well! 🎉</p>
  `,
  styles: ``,
})
export class AppComponent {}
```

In the past Angular had Modules, which handled all the dependencies.
One potential problem with them is that they create indirection.
Not they can be opted out with `standalone: true`.

Template string can be in separate file.

#### Styles
In Angular they are by default scoped. They don't leak out.
So you can safely style "p" tag, it will be scoped to component only.

#### No function components
No plans for function components anytime soon.

#### Dynamic template example
projects/01-hello-angular/src/app/app.component.ts
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, RouterOutlet],
  template: `
    <section>
      <p>Welcome back, {{ userName }}
    </section>
  `,
  styles: `
    p {
      color: grey;
    }
  `,
})
export class AppComponent {
  userName = "MaciejKa"
}
```

#### HMR, hot module reload
At the moment state is not preserved
it's a full reload of a page.

