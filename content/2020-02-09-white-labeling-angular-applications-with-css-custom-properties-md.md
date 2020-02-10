---
title: White Labeling angular applications with CSS Custom Properties
date: 2020-02-09T16:21:41.302Z
cover: /assets/2.jpg
slug: white-labeling-angular-with-css-variables
category: web development
tags:
  - angular
  - css custom properties
  - theming
  - white labeling
---
* [Introduction](#introduction)
* [Difference between css and sass variables](#difference-between-css-and-sass-variables)
* [Time to refactor](#time-to-refactor)
  * [Our theme model](#our-theme-model)
* [Fetching theme details from the backend](#fetching-theme-details-from-the-backend)
* [Implementing the Angular service that applies the themes.](#implementing-the-angular-service-that-applies-the-themes)
  * [Apply theme](#apply-theme)
  * [Scaffold light, dark and other variants](#scaffold-light-dark-and-other-variants)
* [Subscribing to the current theme observable to listen for changes](#subscribing-to-the-current-theme-observable-to-listen-for-changes)
* [Theme directive](#theme-directive)
* [APP_INITIALIZER](#appinitializer)
* [References](#references)

# Introduction

A very common task in web development is doing white labeling. Usually you start off adding a theme or two to your application, you set some wrapper class name in a container element and you add a block of sass in your styles and customize under that class.

```html
<div class="container my-theme">

  <!-- my app content -->
</div>
```

Your theme sass file may look something like this: 

```scss
// .. some imports  

.my-theme {
  $primary-color: #121212;
  $secondary-color: #ededed;
  // .. set all your variables

  @import 'generic-styles.scss';

  // You will probably end up adding custom snippets for specific themes.

  .analytics-section { 
    display: none;
  }
}
```

If you planned from the start to support white labeling then you probably wrote all the 
style rules using the relevant sass variables that you will later override.

After implementing more than 2 or 3 themes you will start noticing that this can get really messy. You will also notice that your css will have a lot of redundant code and the more themes you have, the more css code you will bundle with your application. Also, you will be loading all the themes css even though you will only use 1 theme.

Needless to say, this will not scale if we plan to support tens or hundreds of themes.

# Difference between css and sass variables

Sass variables (and other preprocessor variables as well) get resolved at compile time, once your styles are compiled into CSS, they are replaced with actual values which you can no longer change at runtime. 

On the other hand, CSS variables can be changed at runtime after your page is loaded.  You could set a variable to a certain value as in the following snippet:

```typescript
document.documentElement.style.setProperty('--myCssVariable', '#9efefe');
```

And your styles should reference the variable in some rules:

```scss
.some-class {
  color: var(--myCssVariable);
}
```

Once you set the css variable to some value, your view should immediately reflect the update. 

# Time to refactor

* Remove all theme wrapper css code blocks. 
* Replace all preprocessor variables with css vars in the form `var(--myVarName)`
  * Sometimes you can do something like this:
    ```scss
    $mySassVariable: var(--myCssVariable);
    ```
    This will not work if that SASS variable is later used with SASS functions (for example the `mix()` color function), but if you aren't passing that var through functions that expect a resolved value you should be ok. 
* If you do make use of SASS color functions to obtain darker or lighter variant of your main colors, you may generate those from javascript at the time you load your theme.
  * I am using a small library called [tinycolor](https://github.com/bgrins/TinyColor) in my example which is very simple to use. 
  * The idea is to generate a range of light and dark colors for each of our base palette colors that are defined in our theme. 
    * For example, if we have a variable called `--primaryColor`, then we will also have: `--primaryColorDark10`, `--primaryColorDark20`, ..., `--primaryColorDark90`, `--primaryColorLight10`, ..., `--primaryColorLight90`, and so on. 

## Our theme model

There are some things to note about themes, as part of our themes we will customize:

* CSS styles:
  * mostly made up of our styles referring to CSS vars that we will include in some entity. 
* Content
  * Things we can't customize with CSS, for example: titles, custom footers, hiding/showing elements based on the theme, etc.

Here is an example theme model for demo application:

```ts
export interface Theme {
  name: string;
  brandName: string;
  enableGithubLink?: boolean;
  brandLogo?: string;
  cssRules: { [key: string]: string };
}
```

# Fetching theme details from the backend

Fetch the theme objects from an endpoint like you would fetch any other domain object with an angular service.

# Implementing the Angular service that applies the themes.

## Apply theme

Applying the theme is pretty straight forward, we push it into our theme subject which the service is already subscribed to and when the subscription get notified we iterate through the `cssRules` property of the theme and map the keys to variable names and set the mapped value as a custom property for that name. 

The following snippet registers a var:

```typescript
private registerCssVar(name: string, value: string): void {
    document.documentElement.style.setProperty(name, value);
}
```

## Scaffold light, dark and other variants

```typescript
  
  /**
   * Generates a range of darker, ligher and desaturated 
   * variants of a color with variable name `name`, 
   * from 10% to 90%, stepping by 10%.
   * Generated variables will be named `${name}{Light|Dark|Desaturated}[10-90]`.
   * Also generates a complement and a foreground color to be used
   * when using the base color as background.
   * @param name name of the base color variable
   * @param color base color
   */
  private scaffoldColorVariants(name: string, color: string) {
    const c = tinycolor(color);
    for (let i = 1; i < 10; i++) {
      const lighter = c.clone().lighten(10 * i);
      const darker = c.clone().darken(10 * i);
      const desaturated = c.clone().desaturate(10 * i);
      this.registerCssVar(`${name}Dark${i * 10}`, darker.toHexString());
      this.registerCssVar(`${name}Light${i * 10}`, lighter.toHexString());
      this.registerCssVar(`${name}Desaturated${i * 10}`, desaturated.toHexString());
    }
    const complement = c.clone().complement().toHexString();
    this.registerCssVar(`${name}Complement`, complement);
    let fgVariant = '#ffffff';
    if (c.isLight()) {
      fgVariant = '#000000';
    }
    this.registerCssVar(`${name}Foreground`, fgVariant);
  }
```

And here is the `applyTheme()` method: 

```ts
/**
   * Applies the theme to document.documentElement.style scope.
   * Also scaffold all color variants for each color.
   * @param theme the theme object
   */
  private applyTheme(theme: Theme): void {
    Object.keys(theme.cssRules).forEach(rule => {
      const cssVarName = `--${rule}`;
      const cssRule = theme.cssRules[rule];
      this.registerCssVar(cssVarName, cssRule);
      if (this.isColor(cssVarName)) {
        this.scaffoldColorVariants(cssVarName, cssRule);
      }
    });
  }
```

# Subscribing to the current theme observable to listen for changes

In the ThemesService, provide a method that returns an Observable<Theme> so components can subscribe and listen for theme changes.

```ts
this.themes.getCurrentTheme().subscribe((t: Theme) => {
  // theme has changed, do something to your views
});
```

# Theme directive

In our example we are setting all the custom properties in `document.documentElement.style`, this is global scope and will affect all components, we might want to apply themes just to the scope of a single component and implementing a directive for this seems like the natural way to get this done. 

# Initialize your theme on application startup

Here is a trick we can use to load our theme at application startup to avoid a theme switch when the views are already being displayed.

We create an Initialization service that has the ThemesService injected and has a method that returns a promise. 

```ts
@Injectable({
  providedIn: 'root'
})
export class ThemesInitService {

  constructor(private themes: ThemesService) { }

  public init(): Promise<void> {
    // We need to return a promise
    return new Promise((resolve, reject) => {
      console.log('Initializing themes');

      this.themes.setCurrentTheme(DEFAULT_THEME);
      resolve();
    });
  }
}
```

On the module declaration we declare a factory function that returns a function that, when called, will call the `init()` method of the service.

```typescript
export const initThemes = (themes: ThemesInitService) => {
  return (): Promise<any> => themes.init();
}
```

In our module declaration, we include a new provider that provides `APP_INITIALIZER` and uses the factory function above with `useFactory`. Don't forget to set `multi` to `true`. Include the `ThemesService` in the  `deps` array so it will get injected in the factory function and we're good to go!

```ts
 {
  // @NgModule declaration
 
  providers: [
    ThemesService,
    {
      provide: APP_INITIALIZER,
      useFactory: initThemes,
      deps: [ThemesInitService],
      multi: true
    }
  ],
 }
```

# References

* [My example code - Github repo](https://github.com/baskeboler/angular-themes-blog-post)
* [Mozilla devs - Using CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)
* [css-tricks - post about the differences between css and preprocessor variables](https://css-tricks.com/difference-between-types-of-css-variables/)
* [tinycolor - color manipulation library](https://github.com/bgrins/TinyColor)
