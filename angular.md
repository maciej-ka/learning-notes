Angular 17+ Fundamentals
========================
https://frontendmasters.com/courses/angular-fundamentals/  
Mark (Techson) Thompson

slides  
https://static.frontendmasters.com/assets/courses/2024-01-29-angular-fundamentals/angular-fundamentals-slides.pdf

repository  
https://github.com/MarkTechson/angular-fundamentals-lessons

### Introduction
components  
routing  
capture input with forms  
dependency injection  
app optimization

#### Angular
web framework  
to build scalable web apps

enterprise solution  
enterprise is great at scale

### Angular Essentials
Angular Playground  
angular.dev/playground

```bash
cd angular-fundamentals
npm i
ng serve 01-hello-angular
```

package.json

```json
{
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "watch": "ng build --watch --configuration development",
    "test": "ng test"
  }
}
```

#### Vite
used as a local dev server

#### Angular.json
very important file  
where your project configuration lives

#### Typescript
Angular was very typescript first for long time.  
(Google was considering own tool "at script").

#### Projects folder
This `projects/` may not be visible in a standard angular project.  
Bunch of applications sharing one setup.  
It can be used for collection of projects.  
It can be used for monorepos.

#### global styles.css

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
bootstrap file
index.js in React
main.ts in Angular

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
2. and sign in
3. ng deploy

#### Angular Reputation
It has reputation that it's hard to learn.
Team reduced number of files to help.
(when you're ready for more files, you can add them)

#### Angular Component
Fundamental building block.
TS logic
html mrkup
css styling (also support for SASS/LESS)

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

### Typescript Decorator `@Component`

```typescript
@Component({
  selector: 'app-root',
  standalone: true,
  template: `<h1>Hey, Frontend Masters!</h1>`,
  styles: `h1 { color: red }`,
})
class AppComponent{}
```

#### Standalone, no Angular Modules
Historical topic.
They seem to be right abstraction for this problem.

But they failed at scale.
They were some numerous that at some point
you couldn't be sure, where thing belong anymore.

We made modules optional.
But for us to be able to do that,
we need to add flag

```typescript
standalone: true
```

It means: you don't need a module.

#### Template
Template string, it can be in separate file

#### Styles
In Angular they are by default scoped. They don't leak out.
So you can safely style "p" tag, it will be scoped to component only.
This way components are really reausable.
And it's always safe to move component to new place,
because styles will not collide, as they are scoped.

#### No function components
Angular Team is often hearing this question.
Will there be a support for function components?
Not anytime soon.

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
    <h1>If you are reading this...</h1>
    <p>Things have worked out well! 🎉</p>
    <ol>
      <li>Designing Data Intensive Applications</li>
      <li>Enterprise Integration Patterns</li>
      <li>Not sure</li>
    </ol>
  `,
  styles: `
    ol {
      list-style-type: upper-roman;
    }
  `,
})
export class AppComponent {
  userName = "MaciejKa"

}
```

Lesson 02, displaying 

projects/01-hello-angular/src/app/app.component.ts

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  standalone: true,
  template: `
    <article class="offer">
      <h1>
        <span>Bonus Offer</span>
        <span>&dollar;<!-- ITEM PRICE --></span>
      </h1>
      <img src="/assets/noun-product-6277512.png" width="400" />
      <p>{{ item.name }}</p>
      <p>{{ item.description }}</p>
      <button>Order Now</button>
    </article>
  `,
  styles: `
    .offer {
      font-family: Verdana;
      border: solid 1px gray;
      width: 400px;
      padding: 20px;
      border-radius: 3px;
      color: white;
      background: #7f6b41;
    }
    h1 {
      display: flex;
      justify-content: space-between;
    }
    button {
      display: block;
      width: 100%;
      padding: 10px;
      border: solid 1px white;
      border-radius: 3px;
    }
  `,
})
export class AppComponent {
  item = {
    name: 'Treasure Trove Trunk',
    price: 30,
    description:
      'Unveil a treasure trove of surprises in this delightful mystery box.',
  };
}
```

#### Interpolation

What is inside `{{ }}` brackets

#### Go to definition

It works on interpolations like

```typescript
<p>{{ item.name }}</p>
```

#### Composition

We need to mark that component is imported.
This is not very ergonomic and
Angular is looking to solve this.

(it's because of relic idea of Modules)

And then we use selector of that component.

```typescript
import { UserInfoComponent } from './user-info.component';

@Componennt({
  selector: 'app-dashboard',
  template: `
    <section>
      <p>Welcome back</p>
      <app-user-info />
    </section>
  `,
  imports: [UserInfoComponent]
})
```

### Templating
### Navigation
### Forms
### Signals and Views
### Wrapping Up

