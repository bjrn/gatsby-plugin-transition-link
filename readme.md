![alt text](https://raw.githubusercontent.com/TylerBarnes/gatsby-plugin-transition-link/master/images/gatsby-plugin-transition-link.png "Gatsby Plugin Transition Link logo")

**_NOTE:_** V1 Has just been released! There are a number of awesome improvements but this readme hasn't yet been updated. 

# Gatsby Plugin Transition Link

A plugin for custom page animations in Gatsby.

[Example site](https://gatsby-plugin-transition-link.netlify.com/)

## Installation

`npm i gatsby-plugin-transition-link`

or

`yarn add gatsby-plugin-transition-link`

## Overview / Idea

In the past I thought of page transitions in single instances with each page having it's own entry and exit animation. Because of this I always ended up using a fade transition everywhere due to the complexity of managing multiple animations to and from specific pages. Where would I cleanly describe the transition from page A to page B vs page A to page C?

In trying to figure out an easy way to create and manage these more complex transitions it suddenly hit me: The Link is the link!

Links are already the mediator between pages so it makes sense that they would also describe the transition between them.

TransitionLink provides a simple api for using a link to trigger an animation, delay the current page from unmounting, delay when the next page will display, and send state to the next page to be used in it's own entry animation.

Managing transitions using links means navigating from a home page to a blog post can easily have a different animation than going from the same home page to a contact page.

## Usage

Add `gatsby-plugin-transition-link` to your gatsby config.

```jsx
module.exports = {
    plugins: [
      `gatsby-plugin-transition-link`
    ]
];
```

Use it in your project

```javascript
import TransitionLink from 'gatsby-plugin-transition-link`;
```

```jsx
<TransitionLink
  to="/page-2"
  exit={{
    trigger: exit => this.verticalAnimation(exit, "up"),
    delay: 100,
    length: 1000
  }}
  entry={{ delay: 600, length: 1000, state: { animation: "fromBottom" } }}
>
  Go to page 2
</TransitionLink>
```

## API

### to

Used exactly the same as in gatsby-link.

### exit and entry

each takes an object describing each animation

### exit.length / entry.length

takes an integer describing how many milliseconds the transition should last.
The exiting page will unmount after this and the entering page will have it's status transitioned to "entered".

### exit.trigger

A function that will be called as soon as the link is clicked. You should use it to trigger your exit animation. It receives a property that returns the entire exit prop.
ex:

```jsx
exit={{trigger: exit => this.verticalAnimation(exit, 'down')}}

verticalAnimation = ({length, delay}, direction) {
  // do something cool here
}
```

### entry.trigger

A function that will be called as soon as the entry delay has elapsed. You could use it to trigger your entry animation. It receives a property that returns the entire entry prop.
ex:

```jsx
entry={{trigger: entry => this.verticalAnimation(entry, 'up')}}

verticalAnimation = ({length, delay}, direction) {
  // do something cool here
}
```

### exit.state / entry.state

These can be used to set the state of your pages as they're exiting and entering. This state is passed to your pages and templates and it can also be accessed using the TransitionState component (see below).

## Transition status

Along with the state you pass to the exiting or entering pages, a property called "transitionStatus" will be added to the state object with the values of "entered", "entering", or "exiting".

```javascript
{
  transitionStatus: "entered",
  entry: {},
  exit: {}
}
```

## Accessing transition state

You can use the TransitionState component to access the transition state anywhere.

```jsx
import { TransitionState } from "gatsby-plugin-transition-link";
```

```jsx
<TransitionState>{state => console.log(state)}</TransitionState>
```

Your pages and templates will also receive three props: transitionStatus, entryState, and exitState.

```jsx
const Page = ({ children, transitionStatus, entryState, exitState }) => (
  <div className={transitionStatus}>{children}</div>
);
```

## Default transitions

I haven't tried it yet but theoretically you could wrap TransitionLink in your own component and use that as a link everywhere.

```jsx
const Link = ({children, to}) => (
  <TransitionLink
    to={to}
    exit={{ length: 100, trigger: fadeOut}}
    entry={{delay: 150, state: { animation: fadeIn }}}
    >
      {children}
  </TransitionLink>
 )

 <Link to="/page-2">Go to page 2</Link>
```

or you could abstract away various animations with your own logic.

```jsx
<Link to="/page-2" transition="fade">
  Go to page 2
</Link>

<Link to="/page-3" transition="swipeLeft">
  Go to page 3
</Link>
```

## Considerations

If you use TransitionLink, you shouldn't also use gatsby-link. Currently I haven't set it up so entry and exit states can be reset by gatsby-link. This means your entry or exit animations will keep firing if you mix the normal gatsby-link with TransitionLink and navigate back and forth with the two. It wont be a problem in some cases but it's better not to mix them.
You can still use TransitionLink the same way you use gatsby-link and you can even import it as "Link" if you want.

## Installation Conflicts

TransitionLink uses the `wrapPageElement` hook to add the necessary components. If another plugin prevents transitions from working properly by using the same hook, you can wrap TransitionHandler around pages yourself.

```jsx
import { TransitionHandler } from 'gatsby-plugin-transition-link`;

const Layout = ({element, location}) => (
    <TransitionHandler location={props.location}>{element}</TransitionHandler>
)
```

TransitionHandler needs to be outside the page component though, not inside it. If you're using v2 style layouts as components, that's not the place to put TransitionHandler. You can put it in the `wrapPageElement` or `wrapRootElement` hooks. For most projects adding it to gatsby-config.js should work.
