# css-animator

This package was created out of the need for using CSS animations with a library like [animate.css](https://github.com/daneden/animate.css) in Angular2 when there was no useful animation builder available yet.  

css-animator works with any JavaScript application and takes the pain out of applying CSS animations
manually. It also takes care of positioning elements that are being animated, among other useful things.

Feel free to [open an issue](https://github.com/fabianweb/css-animator/issues/new) if you're experiencing issues, or if you have any suggestions or comments.  

The package includes ES5 compiled [Browserify](http://browserify.org) files, [SystemJS](https://github.com/systemjs/systemjs) bundle files and all TypeScript typings.
Please leave a comment if there's something missing for you.

# Installation

```bash
$ yarn add css-animator
```

```bash
$ npm install --save css-animator
```

```bash
$ jspm install npm:css-animator
```

# Example

`css-animator` is being used in the project [angular2-quiz-app](https://github.com/fabiandev/angular2-quiz-app).  

A very basic example can be found in the [`docs/`](/docs) folder, which is also hosted on GitHub Pages: https://fabiandev.github.io/css-animator/

# Usage

Use this package in combination with CSS animations.  
A library like [animate.css](https://github.com/daneden/animate.css) already offers a lot of great animations out of the box.  

You can install both packages by running:

```bash
$ yarn add css-animator animate.css
```

## Basic Usage

You can use css-animator without Angular2. Just import the class and animate any `HTMLElement`.

```ts
import { AnimationBuilder } from 'css-animator/builder';

let animator = new AnimationBuilder();

animator.setType('shake').animate(element);
```

Want to know when an animation has finished? The AnimationBuilder instance returns a promise:

```ts
animator
  .setType('shake')
  .animate(element)
  .then(() => {
    // Animation finished
  })
  .catch(() => {
    // Animation interrupted
  });
```

You may also change the default options for every instance that is created once changed:

```ts
import { AnimationBuilder } from 'css-animator/builder';

AnimationBuilder.defaults.fixed = true;
AnimationBuilder.defaults.duration = 1500;
```

## Angular2 Service Usage

There is a little Angular2 service included, that gives you the power of dependency injection out of the box.

```ts
import { Component, OnInit } from '@angular/core';
import { AnimationService, AnimationBuilder } from 'css-animator';

@Component({ ... })
export class SomeComponent implements OnInit {

  private animator: AnimationBuilder;

  constructor(animationService: AnimationService, private elementRef: ElementRef) {
    this.animator = animationService.builder();
  }

  ngOnInit() {
    this.animator.setType('fadeInUp').show(this.elementRef.nativeElement);
  }

}
```

`AnimationService` must be defined as provider to make it injectable. You could do so in you main app component like this:

```ts
import { Component } from '@angular/core';
import { AnimationService } from 'css-animator';

@Component({
  selector: 'app',
  templateUrl: '/app.html',
  providers: [
    AnimationService
  ]
})
export class AppComponent {

}
```

## Angular2 Directive Usage

Feel free to create your own directive around css-animator. For you to get started, there is one included in this package.

```ts
import { Component } from '@angular/core';
import { AnimatesDirective } from 'css-animator';

@Component({
  selector: 'my-app',
  template: `
    <div animates #animation="animates">
      <span (click)="animation.start({type: 'bounce'})">Click me!</span>
    </div>
  `
})
export class AppComponent {

}
```

To make use of the directive within an Angular2 module, you have to declare it:

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { AnimatesDirective } from 'css-animator';

@NgModule({
  imports: [
    BrowserModule,
  ],
  declarations: [
    AnimatesDirective,
    AppComponent,
  ],
  bootstrap: [
    AppComponent,
  ],
})
export class AppModule { }
```

Set default options for the animates directive. Those will be used if you use `animation.start()`.
You can optionally pass all options that the interface `AnimationOptions` supports like this: `animation.start({type: 'bounce', duration: 800})`.

```html
<div
animates
#animation="animates"
animates="{ type: 'wobble', duration: '2000' }" // default options are optional
animatesInitMode="show" // Can be used with [animatesOnInit] for "show" or "hide"
[animatesOnInit]="{ type: 'fadeInUp' }" // automatically animate on init (optional)
(click)="animation.start()"
(mouseleave)="animation.pause()"
(mouseenter)="animation.resume()"
>
</div>
```

> You can also animate host elements with css-animator!

# API

## AnimationOptions

Below are all options supported by css-animator. You may notice, that all [CSS animation properties](https://developer.mozilla.org/en/docs/Web/CSS/animation) are included, so you can look up which values are supported, where the options `delay` and `duration` have to be set as numbers in `ms` (e.g. `1000` for one second).  

```ts
export interface AnimationOptions {
  fixed?: boolean;
  reject?: boolean;
  useVisibility?: boolean;
  pin?: boolean;

  type?: string;
  fillMode?: string;
  timingFunction?: string;
  playState?: string;
  direction?: string;
  duration?: number;
  delay?: number;
  iterationCount?: number;
}
```

> The `delay` option is an exception and won't be set as CSS animation property,
> as delays are handled via JavaScript timeouts. If you really want to
> use the CSS rule, you can call `applyDelayAsStyle` to apply the delay immediately on the element.  

#### fixed (default: false)

As mentioned above, elements being animated are positioned `absolute`. If you want to change
the position mode to `fixed`, set the fixed option to `true`.

> Setting this option to true results in a more accurate positioning, as `css-animator`
> won't round to the nearest full pixel (integer instead of float). But keep in mind,
> that you might experience unexpected behavior when scrolling while an element is being animated.

#### reject (default: true)

The promise for an animation is rejected with `animation_aborted`, if it is interrupted somehow. To change
this behavior, set `reject: false` or call `setReject(true)` on an `AnimationBuilder` instance. You may also use the setter on the `AnimationBuilder` instance: `animator.reject = false`.

#### useVisibility

`AnimationBuilder` uses the `hidden` attribute on elements to hide them. If you want to use the `visibility` CSS rule,
set `useVisibility` to `true`.

#### pin (default: true)

By default, an element will be positioned `absolute` while animating, to enable concurrent animations.
Also the relative position (`top` and `left`) will be calculated and set on the element and the `margin` is set to `0px`.
Furthermore the element's calculated `width` and `height` will be set explicitly.
If you want css-animator to only apply the animation, without changing the element's style temporarily, set `pin` to `false`.

#### type (default: 'shake')

The class that will be applied to the element alongside `animated` and `animated-show`, if the element will be shown, or `animated-hide`, if the element will be hidden.

#### fillMode (default: 'none')

See [CSS animation properties](https://developer.mozilla.org/en/docs/Web/CSS/animation).

#### timingFunction (default: 'ease')

See [CSS animation properties](https://developer.mozilla.org/en/docs/Web/CSS/animation).

#### playState (default: 'running')

See [CSS animation properties](https://developer.mozilla.org/en/docs/Web/CSS/animation).

#### direction (default: 'normal')

See [CSS animation properties](https://developer.mozilla.org/en/docs/Web/CSS/animation).

#### duration (default: 1000)

Set the animation duration as integer in ms.

#### delay (default: 0)

Set a delay, before the animation should start as integer in ms.

#### iterationCount (default: 1)

See [CSS animation properties](https://developer.mozilla.org/en/docs/Web/CSS/animation).

## AnimationBuilder

```ts
animate(element: HTMLElement, mode = 'default'): Promise<HTMLElement>
```

Simply animate an element.

```ts
show(element: HTMLElement): Promise<HTMLElement>
```

Animate an element, that was previously hidden.

```ts
hide(element: HTMLElement): Promise<HTMLElement>
```

Adds the attribute `hidden` to the element after the animation has finished.
You may need to add something like `[hidden] { display: none; }` to your CSS.

```ts
stop(element: HTMLElement, reset = true, detach = true): Promise<HTMLElement>
```

Stop the current animation on an element, reset it's position, reject the promise and remove the event listener that listens for animation end.

```ts
setOptions(options: AnimationOptions): AnimationBuilder
```

Set multiple options at once.

```ts
set{Option}(option: string|number): AnimationBuilder
```

You may set options individually like `setDuration(500)` and make use of method chaining.

```ts
addAnimationClass(name: string): AnimationBuilder
```

Adds your custom classes while animating alongside the classes `animated` and `animated-{mode}` (where mode can be `show` or `hide`).

```ts
removeAnimationClass(name: string): AnimationBuilder
```

Won't add classes for future animations, previously added with `addAnimationClass`.

> You can also directly apply options without saving it to the animation builder by using `apply{Option}(options: string|number)`  
> Also there are getters and setters for each option, you can access with `animator.{option}`.

## AnimatesDirective

```ts
start(options?: AnimationOptions): Promise<HTMLElement>
```

Animates the element.

```ts
show(options?: AnimationOptions): Promise<HTMLElement>
```

Shows an element that was hidden.

```ts
hide(options?: AnimationOptions): Promise<HTMLElement>
```

Adds the attribute `hidden` to the element after the animation has finished.
You may need to add something like `[hidden] { display: none; }` to your CSS.

```ts
stop(): void
```

Stop the current animation on an element, reset it's position, and removes the event listener that listens for animation end.

```ts
pause(): void
```

Pauses the animation (sets the playState option to `paused`).

```ts
resume(): void
```

Resumes a previously paused animation (sets the playState option to `running`).

```ts
toggle(): void
```

Switches between `pause()` and `resume()`.
