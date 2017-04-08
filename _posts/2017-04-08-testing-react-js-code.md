---
layout: post
date: 2017-04-08
title: Testing React JS Code
comments: true
---

# Testing React JS Code
A few months ago, I set out to write tests for a UI project I was working on. I
had never written tests for a user interface before. This particular project was
written using Facebook's [React](https://facebook.github.io/react/) framework. I
certainly had experience writing what I considered "normal" unit tests for code
written in Java and Go at that point. That was relatively easy to do. You simply
defined a function that fed input to the function you were testing, and made sure
it produced the desired output.

But testing a user interface is not quite so simple. UIs are not traditionally
functions that can simply be fed input, and be tested for the desired output.
Web UIs are built with HTML, CSS, and JavaScript. Their output is the DOM. So I
was rather curious how one would go about writing tests for something like that.
How would they be structured? What would they test for?

## Enter React
React is a component-based framework, so if used correctly, your UI will be
very modular, and composed of independent and isolatable components. These
components are reminiscent of functions, and they facilitate the writing of useful
tests. They also take input in the form of "props", further increasing their
likeness with functions.

So they are modular, and have inputs. Great. But in order to write tests for them,
they must also have some sort of output that can be compared against a predetermined
standard to determine correctness. Their output comes in the form of an element
in a virtual DOM, which can be manipulated and inspected, and thereby be tested.

## Jest and Enzyme
Facebook has created a nice testing library called [Jest](http://facebook.github.io/jest/).
It provides a very easy to read API, and a number of utilities that make writing
tests much easier. When used with AirBnB's [Enzyme](http://airbnb.io/enzyme/),
it becomes possible to write a variety of useful tests with very little overhead.

I'll now walk through the creation of an example React Component, and
demonstrate how to write tests for it. The code is
[available on GitHub](https://github.com/Hopding/example-react-unit-test) as well.

## Create a React App
Facebook has created a wonderful utility called
[Create React App](https://github.com/facebookincubator/create-react-app).
It does exactly what it sounds like: creates a React App. It takes care of all
the initial configuration and lets you quickly get to writing a app. Just run

~~~
npm install -g create-react-app
~~~

to install it. Then run

~~~
create-react-app test-app
~~~

to create an app titled "test-app" in you current directory. Next, run

~~~
cd test-app
yarn start
~~~

to launch the app, and open it in a new browser tab.

## Create a Component
Now that we have a React App, we can create React Components to test. Lets
create a simple `<ColorLabel />` component. It will simply render a div whose
`backgroundColor` toggles between `'green'` and `'red'` when clicked. When clicked,
it will also invoke a callback function from its props. Make this file:
`/test-app/src/ColorLabel.jsx` and add the following code to it.

~~~jsx
import React from 'react';

class ColorLabel extends React.Component {
 constructor(props) {
   super(props);
   this.state = {
     isActive: false,
   }
 }

 render() {
   const { isActive } = this.state;
   const { onClick } = this.props;
   const backgroundColor = isActive ? 'red' : 'green';
   return (
     <div
       style={{ backgroundColor }}
       onClick={() => {
         this.setState({ isActive: !isActive });
         onClick();
       }}
     >
       Click Me!
     </div>
   );
 }
}

export default ColorLabel;
~~~

Next, open `test-app/src/App.js` and replace its contents with the following:

~~~jsx
import React from 'react';
import ColorLabel from './ColorLabel.jsx';

class App extends React.Component {
  render() {
    return (
      <ColorLabel
        onClick={() => console.log('Teehee, that tickles!')}
      />
    );
  }
}

export default App;
~~~

The app should now render as a simple green `<div>` with the text `Click Me!`
displayed within. Clicking it should toggle the color between green and red, and
cause it to log the message "Teehee, that tickles!" to the console.

## Create a Test for `ColorLabel.jsx`
Now that we've defined a component, lets write a test for it. Create this file:
`test-app/__tests__/ColorLabel.test.jsx` and add the following code to it:

~~~jsx
import React from 'react';
import { shallow } from 'enzyme';
import ColorLabel from '../src/ColorLabel.jsx'

describe('<ColorLabel />', () => {
  it('toggles color when clicked', () => {
    const mockFunc = jest.fn();
    const wrapper = shallow(
      <ColorLabel
        onClick={mockFunc}
      />
    );
    expect(wrapper.props().style.backgroundColor).toBe('green');
    wrapper.simulate('click');
    expect(wrapper.props().style.backgroundColor).toBe('red');
    expect(mockFunc).toHaveBeenCalledTimes(1);
  });
});

~~~

## Run the Test
Before we can run the test, we need to install the `enzyme` and
`react-addons-test-utils` modules. Just run:

~~~
yarn add enzyme react-addons-test-utils
~~~

You'll also recall that in addition to Enzyme, I previously mentioned Jest. We
are actually using Jest in this test as well, but we do not have to explicitly
import or install it, because it is taken care of for us by `create-react-app`.

If you now run `yarn test` from the root directory (`test-app`), you will see
that our test runs and passes. There are a few things to understand about this
test:

* [`describe`](http://facebook.github.io/jest/docs/api.html#describename-fn) is
a function that takes a string name, and a function containing a series of test
functions. It indicates a suite of tests.
* `it` is a synonym for the [`test`](http://facebook.github.io/jest/docs/api.html#testname-fn)
function. It accepts a string name, and a function that will either fail or pass
based on the results of the `expect` call(s).
* [`jest.fn()`](http://facebook.github.io/jest/docs/mock-functions.html)
creates a mock function. These can be passed to components just like any
other function. They don't "do" anything, but they expose certain properties
that we can inspect to ensure our component is handling it properly.
* [`shallow`](http://airbnb.io/enzyme/docs/api/shallow.html) creates an enzyme
[`wrapper`](http://airbnb.io/enzyme/GLOSSARY.html#wrapper) object for the
passed-in React Component - in this case, `<ColorLabel />`. Wrappers are useful
because they allow us to inspect and manipulate the component they contain.
* [`expect`](http://facebook.github.io/jest/docs/expect.html#expectvalue) will
take some value (or expression that evaluates to a value), and compares it to the
value specified in the matcher function.
* [`toBe`](http://facebook.github.io/jest/docs/expect.html#tobevalue) is one of
the matcher functions in this test. It just does a comparison with the `===`
operator.
* [`toHaveBeenCalledTimes`](http://facebook.github.io/jest/docs/expect.html#tohavebeencalledtimesnumber)
is the other matcher function in this test. It simply checks to make sure that
the specified mock function was called the specified number of times.

After creating a `wrapper` for a `<ColorLabel />` component with the `shallow`
function, we perform a test:

~~~jsx
expect(wrapper.props().style.backgroundColor).toBe('green');
~~~

You'll notice that the expression we are passing to `expect` is
`wrapper.props().style.backgroundColor`. This expression first invokes
`wrapper.props()`, which returns the `props` object of the component which it
wraps. We then get the value of `backgroundColor` for the `style` prop.

This passes because the initial `backgroundColor` of a `<ColorLabel /> is,
indeed, `green`.

We then simulate clicking the `<ColorLabel />` with

~~~jsx
wrapper.simulate('click');
~~~

This modifies the state of the component being wrapped just as it does when we
click on it in the browser. In the case of our component, it results in the
`backgroundColor` being changed to `red`. We confirm that is is the case with
a second test:

~~~jsx
expect(wrapper.props().style.backgroundColor).toBe('red');
~~~

In addition to changing colors when clicked, the `<ColorLabel />` component
should invoke its `onClick` prop. We check to make sure this has actually
occurred with a third test:

~~~jsx
expect(mockFunc).toHaveBeenCalledTimes(1);
~~~

This time the expression we pass to `expect` is `mockFunc`, and we check to see
if it was called exactly one time. This test passes because our component did
indeed call our mock function as we expected it to. If we had made a mistake and
our component was failing to invoke the callback when clicked, this test would
catch that mistake for us.

## Snapshot Testing
This guide only covers the basics of what Jest and Enzyme can do. Another very
cool feature that they provide is "snapshot testing", which allows you to
increase code coverage with minimal effort. Snap shot testing also makes it very
easy to update your tests to stay in sync with your changing code.
[This video](https://www.youtube.com/watch?v=HAuXJVI_bUs) is a great
introduction to the subject.

## Summary
Unit tests are an important part of any substantial software project, and should
be included in any continuous integration system. Writing tests for a user
interface is a bit different than writing them for typical backend-type code.
However, the React framework facilitates modularization of UI code into
components resembling functions. These components can then be rendered,
manipulated, inspected, and tested in a virtual DOM using the Jest and Enzyme
packages. The result is easy to write and read unit tests that ensure proper
functioning of your UI code!
