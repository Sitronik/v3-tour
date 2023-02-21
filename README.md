# v3-tour

> v3-tour is a lightweight, simple and customizable tour plugin for use with Vue.js.
> It provides a quick and easy way to guide your users through your application.

[![Vue Tour](./screenshot.gif "Vue Tour")](https://pulsardev.github.io/vue-tour/)

## Table of Contents

- [Getting Started](#getting-started)
- [Something Missing?](#something-missing)

## Getting Started

You can install `v3-tour` using npm or by downloading the minified build on GitHub.

```
npm install v3-tour
```

Then import the plugin in your application entry point (typically main.js if you used vue-cli to scaffold your project) and tell Vue to use it.
Also don't forget to include the styles. You can add the styles provided by default or customize them to your own liking.

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import VueTour from 'v3-tour'

import 'v3-tour/dist/vue-tour.css' // css for this library

const app = createApp(App)

app.use(VueTour).mount('#app')

```

Finally put a `v-tour` component in your template anywhere in your app (usually in App.vue) and pass it an array of steps.
The `target` property of each step can target a DOM element in any component of your app (as long as it exists in the DOM when the concerned step pops up).

```html
<template>
  <div>
    <div id="v-step-0">A DOM element on your page. The first step will pop on this element because its ID is 'v-step-0'.</div>
    <div class="v-step-1">A DOM element on your page. The second step will pop on this element because its ID is 'v-step-1'.</div>
    <div data-v-step="2">A DOM element on your page. The third and final step will pop on this element because its ID is 'v-step-2'.</div>

    <v-tour name="myTour" :steps="steps"></v-tour>
  </div>
</template>

<script setup lang="ts">
import { getCurrentInstance, onMounted } from 'vue';

const steps= [
  {
    target: '#v-step-0',  // We're using document.querySelector() under the hood
    header: {
      title: 'Get Started',
    },
    content: `Discover <strong>Vue Tour</strong>!`
  },
  {
    target: '.v-step-1',
    content: 'An awesome plugin made with Vue.js!'
  },
  {
    target: '[data-v-step="2"]',
    content: 'Try it, you\'ll love it!<br>You can put HTML in the steps and completely customize the DOM to suit your needs.',
    params: {
      placement: 'top' // Any valid Popper.js placement. See https://popper.js.org/popper-documentation.html#Popper.placements
    }
  }
]

onMounted(() => {
  const internalInstance = getCurrentInstance()
  const $tours = internalInstance?.appContext.config.globalProperties.$tours
  if ($tours) {
    if ($tours['myTour']) {
      $tours['myTour'].start()
    }
  }
})



</script>
```

For all individual elements you want to add a step on, make sure it can be retrieved with `document.querySelector()`. You can use any selector, an ID, a CSS class, data attributes, etc.
Once this is done and your steps correctly target some DOM elements of your application, you can start the tour by calling the following method.

```javascript

  const internalInstance = getCurrentInstance()
  const $tours = internalInstance?.appContext.config.globalProperties.$tours
  if ($tours) {
    if ($tours['myTour']) {
      $tours['myTour'].start()
    }
  }
```

For the precursor to `v3-tour`, checkout the [docs for vue-tour](https://github.com/pulsardev/vue-tour/wiki).

## `before()` UI step functions

If you need to do UI setup work before a step, there's a `before` function you may include in any/each of 
your steps. This function will get invoked before the start/next/previous step is rendered. The function must return a promise. The function is invoked when `start`, `nextStep`, and `previousStep` are triggered. When the promise is rejected, it will not move to the next or previous step. If the promise is resolved then it will move in the direction specified.

It's used when you need to change what's shown on the screen between steps. For example, you may want to hide
one set of menus and open a screen or you want to perform an async operation. This is especially useful in single-page applications.

```javascript
steps: [
  {
    target: '#v-step-0',  // We're using document.querySelector() under the hood
    content: `Discover <strong>Vue Tour</strong>!`,
    before: type => new Promise((resolve, reject) => {
      // Time-consuming UI/async operation here
      resolve('foo')
    })
  }
]
```

## Tour Configuration

When initializing your tour, you may pass an object to the options prop.

Your custom configuration object doesn't need to (re)declare every values, you only need to define what you want to change.

This prop currently support the following properties:

When initializing your tour, you may pass an object to the options prop.

Your custom configuration object doesn't need to (re)declare every values, you only need to define what you want to change.

This prop currently support the following properties:

| Option | Default | Description |
|---|---|---|
| useKeyboardNavigation | true | If set to true you may use the ←, → and ESC keys to navigate through your tour. |
| startTimeout | 0 | Defines the timeout before which the tour starts. |
| labels | undefined | tour's buttons' labels |

Here is how you can customize the labels of the tour's buttons.

```js
labels: {
  buttonSkip: 'Skip tour',
  buttonPrevious: 'Previous',
  buttonNext: 'Next',
  buttonStop: 'Finish'
}
```

### Example

```html
<template>
  <div>
    <div id="v-step-0">A DOM element on your page.</div>
    <div id="v-step-1">A DOM element on your page.</div>
    <div id="v-step-2">A DOM element on your page.</div>

    <v-tour name="myTour" :steps="steps" :options="myOptions"></v-tour>
  </div>
</template>

<script setup lang="ts">
import { getCurrentInstance, onMounted } from 'vue';

const myOptions = {
  useKeyboardNavigation: false,
  labels: {
    buttonSkip: 'Skip tour',
    buttonPrevious: 'Previous',
    buttonNext: 'Next',
    buttonStop: 'Finish'
  }
}

const steps =  [
   // ... Your steps array
]


onMounted(() => {
  const internalInstance = getCurrentInstance()
  const $tours = internalInstance?.appContext.config.globalProperties.$tours
  if ($tours) {
    if ($tours['myTour']) {
      $tours['myTour'].start()
    }
  }
})
</script>
```


## Customizing the template 

Vue Tour makes an extensive usage of Vue.js slot functionality to allow you to customize nearly every aspects of the steps of your tour. The simplest way of inserting a tour in your page is by just using a v-tour component.

```html
<v-tour name="myTour" :steps="steps"></v-tour>
```

This component contains a default slot that wraps a v-for loop over the steps passed as a prop. And the steps themselves contain three slots: `header`, `content` and `actions`. Then, if you want to override the DOM of the steps you can do it by overriding the slots like so:

```html
<v-tour name="myTour" :steps="steps">
  <template v-slot="tour">
    <transition name="fade">
      <v-step
        v-if="tour.steps[tour.currentStep]"
        :key="tour.currentStep"
        :step="tour.steps[tour.currentStep]"
        :previous-step="tour.previousStep"
        :next-step="tour.nextStep"
        :stop="tour.stop"
        :skip="tour.skip"
        :is-first="tour.isFirst"
        :is-last="tour.isLast"
        :labels="tour.labels"
      >
        <template v-slot:content v-if="tour.currentStep === 2">
          <p v-html="tour.steps[tour.currentStep].content" />
        </template>
        <template v-if="tour.currentStep === 2">
          <div v-slot:actions>
            <button @click="tour.previousStep" class="btn btn-primary">Previous step</button>
            <button @click="tour.nextStep" class="btn btn-primary">Next step</button>
          </div>
        </template>
      </v-step>
    </transition>
  </template>
</v-tour>
```

Here we changed only the actions slot of the third step (the count begins at 0). The slot in the v-tour component is scoped so that you can use its internal properties and methods.


## Callbacks

Vue Tour provides callbacks to allow you to perform custom actions at different moments of the tour. For exemple if you want to call an API when the user goes to the 2nd step of your tour, fire a Google Analytics event, etc.

To use callbacks, add a callbacks props to the v-tour component:

```html
<v-tour name="myTour" :steps="steps" :callbacks="myCallbacks">
```

Where myCallbacks is an object containing your methods. For exemple here, we define an object with two callbacks:

```js
const myCallbacks = {
  onPreviousStep: myCustomPreviousStepCallback,
  onNextStep: myCustomNextStepCallback
}
```

And then, you have to declare your methods, like so:

```js
const myCustomPreviousStepCallback = (currentStep) => {
  console.log('[Vue Tour] A custom previousStep callback has been called on step ' + (currentStep + 1))
}
const myCustomNextStepCallback = (currentStep) => {
  console.log('[Vue Tour] A custom nextStep callback has been called on step ' + (currentStep + 1))

  if (currentStep === 1) {
    console.log('[Vue Tour] A custom nextStep callback has been called from step 2 to step 3')
  }
}
```

There are at the moment 5 callbacks available to you:

| Name | Called when you... |
| --- | --- |
| onStart | start the tour |
| onPreviousStep | go to a previous step |
| onNextStep | go to a next step |
| onSkip | skip the tour |
| onFinish | end the tour |
| onStop | stop the tour (press ESC key for example) |


## `before()` UI step functions 

If you need to do UI setup work before a step, there's a `before` function you may include in any/each of 
your steps. This function will get invoked before the start/next/previous step is rendered. The function must return a promise. The function is invoked when `start`, `nextStep`, and `previousStep` are triggered. When the promise is rejected, it will not move to the next or previous step. If the promise is resolved then it will move in the direction specified.

It's used when you need to change what's shown on the screen between steps. For example, you may want to hide
one set of menus and open a screen or you want to perform an async operation. This is especially useful in single-page applications.

```javascript
steps: [
  {
    target: '#v-step-0',  // We're using document.querySelector() under the hood
    content: `Discover <strong>Vue Tour</strong>!`,
    before: type => new Promise((resolve, reject) => {
      // Time-consuming UI/async operation here
      resolve('foo')
    })
  }
]
```

## Debug 

Vue Tour comes with debug option, allowing you to see logs from the plugin.
By default, you won't be able to see debug logs such as: `[Vue Tour] Highlight is disabled for .v-step[id="b4820cfe"]`

 ```html
<v-tour name="myTour" :steps="steps" :callbacks="callbacks" :options="{ debug: true }">
```

## Highlight 

You can highlight the element showcased by the current step by setting the `highlight` option to true.

```html
<v-tour name="myTour" :steps="steps" :callbacks="callbacks" :options="{ highlight: true }">
```

You can also disable the highlight on a step basis by using step params.

```js
const steps = [
  {
    target: '#v-step-0',
    content: `Discover <strong>Vue Tour</strong>!`,
    params: {
      highlight: false
    }
  },
  ...
]
```

If you are customizing the template, don't forget to pass the highlight property to the `VStep` props:

```html
<v-step
  v-if="tour.currentStep === index"
  v-for="(step, index) of tour.steps"
  :key="index"
  :step="step"
  ...
  :highlight="tour.highlight"
>
```

## Disable scroll between steps

By default Vue Tour scrolls between each steps. You can override this behavior by adding `enableScrolling: false` to step params you don't want scroll enabled.

```js
{
  target: '#v-step-0',
  content: `Discover <strong>Vue Tour</strong>!`,
  params: {
    enableScrolling: false
  }
}
```

## Customization

By default the shadow is a solid thin box-shadow. If you want to have a full backdrop overlay over the entire viewport you can customize the `.v-tour__target--highlighted` class and set a very large box-shadow (99999px for example):

```css
.v-tour__target--highlighted {
  box-shadow: 0 0 0 99999px rgba(0,0,0,.4);
}
```

## Something Missing?

If you have a feature request or found a bug, [let us know](https://github.com/Sitronik/v3-tour/issues) by submitting an issue.
