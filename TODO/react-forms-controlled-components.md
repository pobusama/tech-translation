* 原文地址：[React.js Forms: Controlled Components](http://lorenstewart.me/2016/10/31/react-js-forms-controlled-components/)
* 原文作者：[Loren Stewart](http://lorenstewart.me/author/lorenseanstewart/)
* 译者：[小 B0Y](http://pobusama.github.io/about/)
* 校对者：[](#)

# React.js Forms: Controlled Components

This post covers the following controlled components:
- text inputs
- number inputs
- radio inputs
- checkbox inputs
- textareas
- selects

Also covered are:
- Clearing/resetting the form's data
- Submitting data
- Validation

> Just want the code? [Here it is!](https://github.com/lorenseanstewart/react-controlled-form-components)   
> Check out the [Demo](http://lorenstewart.me/react-controlled-form-components/).   
> *Be sure to have your browser's console open as you use the demo.*    

## Introduction

The problem I came across when learning [React.js](https://facebook.github.io/react/) was finding real-world examples of controlled form components. Examples of controlled text inputs are plentiful, but what about checkboxes? Radios? Selects?

Here is a list of real-world examples of controlled form components; it's the list I wish I found early on in my React education. All form elements are represented here except for date and time inputs, which need a post of their own.

To speed up development time, sometimes it's tempting to import a library for something like form elements. When it comes to something like forms, I've found that using library just makes life more difficult when I need to add custom behavior or validation. Once you know proper React patterns, creating form components isn't difficult and it's something we should all probably do ourselves. Please use the code in this post as inspiration or as a starting point for your own form components.

In addition to the code for individual components, I've put them all together in a (pet adoption!) form so you can see how child components update the parent component's state, and how the parent then updates the child component via props (unidirectional data flow).

**Note: **[This form](http://lorenstewart.me/react-controlled-form-components/) is built with the wonderful [create-react-app](https://github.com/facebookincubator/create-react-app) build configuration. If you haven't installed it yet, I strongly recommend doing so (`npm install -g create-react-app`). It's by far the easiest way to get set-up to build React apps.

## What is a controlled component?

A controlled component has two aspects:

1. Controlled components have functions to govern the data going into them on every `onChange` event, rather than grabbing the data only once, e.g. when a user clicks a submit button. This 'governed' data is then saved to state (in this case, the parent/container component's state).
2. Data displayed by a controlled component is received through props passed down from it's parent/container component.

This is a one-way loop – from (1) child component input (2) to parent component state and (3) back down to the child component via props – is what is meant by unidirectional data flow in React.js application architecture.

## Architecture of the form

Our highest level component, is named `App`, and here it is:

```jsx
import React, { Component } from 'react';  
import '../node_modules/spectre.css/dist/spectre.min.css';  
import './styles.css';  
import FormContainer from './containers/FormContainer';

class App extends Component {  
  render() {
    return (
      <div className="container">
        <div className="columns">
          <div className="col-md-9 centered">
            <h3>React.js Controlled Form Components</h3>
            <FormContainer />
          </div>
        </div>
      </div>
    );
  }
}

export default App;  
```

`App` doesn't do anything but render on our `index.html` page. The interesting part of `App` is on line 13, the `FormContainer`.

## Interlude: container (smart) components vs (dumb) components

This is a good time to mention container (smart) components vs (dumb) components. Container components house business logic, make data calls, etc. Regular, or dumb, components receive data from their parent (container) component. Dumb components may trigger logic, like updating state, but only by means of functions passed down from the parent (container) component.

**Note:** I should point out that not all parent components are container components, but that's how our form is set up. It's perfectly fine to have a hierarchy of dumb components within dumb components.

## Back to Architecture

The `FormContainer` houses the form element components, calls data in the `componentDidMount` lifecycle hook, and contains the logic for updating the state of the form. For the sake of simplicity, I've left out the props and change handlers of the form element components in the outline below. (Scroll to the end of the post for the complete code.)

```jsx
import React, {Component} from 'react';  
import CheckboxOrRadioGroup from '../components/CheckboxOrRadioGroup';  
import SingleInput from '../components/SingleInput';  
import TextArea from '../components/TextArea';  
import Select from '../components/Select';

class FormContainer extends Component {  
  constructor(props) {
    super(props);
    this.handleFormSubmit = this.handleFormSubmit.bind(this);
    this.handleClearForm = this.handleClearForm.bind(this);
  }
  componentDidMount() {
    fetch('./fake_db.json')
      .then(res => res.json())
      .then(data => {
        this.setState({
          ownerName: data.ownerName,
          petSelections: data.petSelections,
          selectedPets: data.selectedPets,
          ageOptions: data.ageOptions,
          ownerAgeRangeSelection: data.ownerAgeRangeSelection,
          siblingOptions: data.siblingOptions,
          siblingSelection: data.siblingSelection,
          currentPetCount: data.currentPetCount,
          description: data.description
        });
    });
  }
  handleFormSubmit() {
    // submit logic goes here
  }
  handleClearForm() {
    // clear form logic goes here
  }
  render() {
    return (
      <form className="container" onSubmit={this.handleFormSubmit}>
        <h5>Pet Adoption Form</h5>
        <SingleInput /> {/* Full name text input */}
        <Select /> {/* Owner age range select */}
        <CheckboxOrRadioGroup /> {/* Pet type checkboxes */}
        <CheckboxOrRadioGroup /> {/* Will you adopt siblings? radios */}
        <SingleInput /> {/* Number of current pets number input */}
        <TextArea /> {/* Descriptions of current pets textarea */}
        <input
          type="submit"
          className="btn btn-primary float-right"
          value="Submit"/>
        <button
          className="btn btn-link float-left"
          onClick={this.handleClearForm}>Clear form</button>
      </form>
  );
}
```

Now that the basic architecture is laid out, let's take a look at each child element.

## `<SingleInput />`

This component can be either a `text` or a `number` input, depending on the props you pass it. A great way to document the props a component takes is via React's PropTypes. If any props are missing, or if the prop is the wrong data type, a warning will appear in the browser console.

Listed below are the PropTypes for the `<SingleInput />` component.

```js
SingleInput.propTypes = {  
  inputType: React.PropTypes.oneOf(['text', 'number']).isRequired,
  title: React.PropTypes.string.isRequired,
  name: React.PropTypes.string.isRequired,
  controlFunc: React.PropTypes.func.isRequired,
  content: React.PropTypes.oneOfType([
    React.PropTypes.string,
    React.PropTypes.number,
  ]).isRequired,
  placeholder: React.PropTypes.string,
};
```

PropTypes indicate the type of the prop(string, number, array, object, etc.), whether it is required (`isRequired`), and much more. (See the [React docs](https://facebook.github.io/react/docs/typechecking-with-proptypes.html) for more details).

Let's go through these one by one.

1. `inputType` accepts two different strings:  `'text'` or `'number'`. These options determine whether a `<input type="text" />` or an `<input type="number" />` is rendered.
2. `title`: accepts a string that will be rendered in the input's label.
3. `name`: the name attribute for the input.
4. `controlFunc`: is the function passed down from the parent/container component. This function will update the parent/container  component's state every time there is a change because it is attached to React's onChange handler.
5. `content`: the content of the input. A controlled input will only display the data passed into it via props.
6. `placeholder`: a string that will be the input's placeholder text.

Since we don't need any logic or internal state for our input, it can be a pure functional component. Pure functional components are attached to a `const`. Here is all the code for the `<SingleInput />`. All of the form element components in this post are pure functional components.

```jsx
import React from 'react';

const SingleInput = (props) => (  
  <div className="form-group">
    <label className="form-label">{props.title}</label>
    <input
      className="form-input"
      name={props.name}
      type={props.inputType}
      value={props.content}
      onChange={props.controlFunc}
      placeholder={props.placeholder} />
  </div>
);

SingleInput.propTypes = {  
  inputType: React.PropTypes.oneOf(['text', 'number']).isRequired,
  title: React.PropTypes.string.isRequired,
  name: React.PropTypes.string.isRequired,
  controlFunc: React.PropTypes.func.isRequired,
  content: React.PropTypes.oneOfType([
    React.PropTypes.string,
    React.PropTypes.number,
  ]).isRequired,
  placeholder: React.PropTypes.string,
};

export default SingleInput;  
```

And the `handleFullNameChange` function (passed into the `controlFunc` prop) updates the `<FormContainer />`'s state.

```js
// FormContainer.js

handleFullNameChange(e) {  
  this.setState({ ownerName: e.target.value });
}
// make sure to have:
// this.handleFullNameChange = this.handleFullNameChange.bind(this);
// in the constructor
```

The new container's state is then passed back into the `<SingleInput />` via the `content` prop.

## `<Select />`

The select component (i.e. a `dropdown`), takes the following props:

```jsx
Select.propTypes = {  
  name: React.PropTypes.string.isRequired,
  options: React.PropTypes.array.isRequired,
  selectedOption: React.PropTypes.string,
  controlFunc: React.PropTypes.func.isRequired,
  placeholder: React.PropTypes.string
};
```

1. `name`: a string that will populate the `name` attribute of our form element.
2. `options`: an array (of strings in our case) in which each item will become an option by using `props.options.map()` in the component's render method.
3. `selectedOption`: if we are prepopulating the form with either default data, or with data a user added in the past (e.g. this is used when a user edits data they have submitted on a prior occasion).
4. `controlFunc`: is the function passed down from the parent/container component. This function will update the parent/container component's state every time there is an change because it is attached to React's `onChange` handler.
5. `placeholder`: a string that populates the first `<option>` tag, and acts as placeholder text. We set the value of this option to an empty string in the component (see line 10 below)

```jsx
import React from 'react';

const Select = (props) => (  
  <div className="form-group">
    <select
      name={props.name}
      value={props.selectedOption}
      onChange={props.controlFunc}
      className="form-select">
      <option value="">{props.placeholder}</option>
      {props.options.map(opt => {
        return (
          <option
            key={opt}
            value={opt}>{opt}</option>
        );
      })}
    </select>
  </div>
);

Select.propTypes = {  
  name: React.PropTypes.string.isRequired,
  options: React.PropTypes.array.isRequired,
  selectedOption: React.PropTypes.string,
  controlFunc: React.PropTypes.func.isRequired,
  placeholder: React.PropTypes.string
};

export default Select;  
```

Note the `key` attribute in our option tags (line 14). React requires a unique `key` for every element that is rendered through a repeater operation like our `.map()` function. Since each element in our options array is unique, we can use it as the `key` prop. This `key` helps React keep track of DOM changes. Your app won't break if leave out the `key` attribute in your repeater/mapping function, but you'll have warnings in your browser console and rendering performance will be compromised.

Below is the handler function (that is passed into the `controlFun` prop from `<FormContainer />`) that controls our select (reminder: it lives in `<FormContainer />`).

```jsx
// FormContainer.js

handleAgeRangeSelect(e) {  
  this.setState({ ownerAgeRangeSelection: e.target.value });
}
// make sure to have:
// this.handleAgeRangeSelect = this.handleAgeRangeSelect.bind(this);
// in the constructor
```

## `<CheckboxOrRadioGroup />`

Unlike the other components, the `<CheckboxOrRadioGroup />` component takes in an array through its props, maps over the array (just like the options of the `<Select />` component above), and renders a set of form elements – either a set of checkboxes or a set or radios.

Let's dive into the PropTypes to better understand `<CheckboxOrRadioGroup />`.

```jsx
CheckboxGroup.propTypes = {  
  title: React.PropTypes.string.isRequired,
  type: React.PropTypes.oneOf(['checkbox', 'radio']).isRequired,
  setName: React.PropTypes.string.isRequired,
  options: React.PropTypes.array.isRequired,
  selectedOptions: React.PropTypes.array,
  controlFunc: React.PropTypes.func.isRequired
};
```

1. `title`: a string that populates the label for the set of checkboxes/radios
2. `type`: takes one of two possible options, `'checkbox'` or `'radio'`, and renders inputs of the indicated type.
3. `setName`: a string that will populate the `name` attributes of each checkbox/radio.
4. `options`: an array, in our case an array of strings, that determines the label and value for each checkbox/radio. E.g., `['dog', 'cat', 'pony']` will render three checkboxes/radios, one for each item in the array.
5. `selectedOptions`: an array, in our case an array of strings, of pre-selected options. In the example used in #4 above, if `selectedOptions` contained `'dog'` and `'pony'` then these two options would render as checked and `'cat'` would render unchecked. This is the array that will be submitted as the user's choices.
6. `controlFunc`: the function that handles adding and removing strings from the used as `selectedOptions` prop.

This is the most interesting component in our form. Here's the code:

```jsx
import React from 'react';

const CheckboxOrRadioGroup = (props) => (  
  <div>
    <label className="form-label">{props.title}</label>
    <div className="checkbox-group">
      {props.options.map(opt => {
        return (
          <label key={opt} className="form-label capitalize">
            <input
              className="form-checkbox"
              name={props.setName}
              onChange={props.controlFunc}
              value={opt}
              checked={ props.selectedOptions.indexOf(opt) > -1 }
              type={props.type} /> {opt}
          </label>
        );
      })}
    </div>
  </div>
);

CheckboxOrRadioGroup.propTypes = {  
  title: React.PropTypes.string.isRequired,
  type: React.PropTypes.oneOf(['checkbox', 'radio']).isRequired,
  setName: React.PropTypes.string.isRequired,
  options: React.PropTypes.array.isRequired,
  selectedOptions: React.PropTypes.array,
  controlFunc: React.PropTypes.func.isRequired
};

export default CheckboxOrRadioGroup;  
```

The logic that determines if a radio/checkbox is checked, is found in the line: `checked={ props.selectedOptions.indexOf(option) > -1 }`.

The input attribute `checked` takes a Boolean to determine if the input should render as checked. We generate this Boolean by checking to see if the value of the input is an element in the `props.selectedOptions` array. `myArray.indexOf(item)` returns the index of the item in an array. If the item is NOT in the array, it returns `-1`. Thus, we have `> -1`.

Keep in mind that `0` is a legitimate index number, so you need the `> -1` or your code will be buggy; without it, the first item in the `selectedOptions` array – with an index of `0` – will never render as checked, because `0` is a falsey value.

The handler function for this component is also more interesting that the others.

```jsx
handlePetSelection(e) {

  const newSelection = e.target.value;
  let newSelectionArray;

  if(this.state.selectedPets.indexOf(newSelection) > -1) {
    newSelectionArray = this.state.selectedPets.filter(s => s !== newSelection)
  } else {
    newSelectionArray = [...this.state.selectedPets, newSelection];
  }

    this.setState({ selectedPets: newSelectionArray });
}
```

As with all of our handler functions, the event object is passed in so its value can be extracted. We attached this value to the constant `newSelection`. We then declare the `newSelectionArray` variable near the top of the function. It is a `let` variable rather than a `const` because it will be assigned within one of the `if/else` blocks. We declare it outside of these blocks so it is in the outer scope of the function and is accessible to all the blocks within.

This function has to handle two possibilities.

If the value of the input **IS NOT** in the `selectedOptions` array, it needs to be added.
If the value of the input **IS** in the `selectedOptions` array, it needs to be removed.

`Adding (lines 8 - 10):` To add a new value to the array of selections, we create a new array by destructuring the original array (indicated by the three dots `...` in front of the array) and adding the new value to the end `newSelectionArray = [...this.state.selectedPets, newSelection];`.

Notice, the original array is not mutated with a method like `.push()`, but, rather, a new array is created. This practice of creating new objects and arrays rather than mutating existing ones is another best practice in React. This allows developers to more easily keep track of state change, and allows third party state management libraries like [Redux](http://redux.js.org/docs/introduction/) to do highly performant shallow checking of data types rather than performance hindering deep checking.

Removing (lines 6 - 8): The `if` block checks to see if the selection is in the array by means of the `.indexOf()` trick used above. If the selection is already in the array, it is removed using the JavaScript array method `.filter()`. This method returns a new array (remember to avoid mutating in React!) containing all items that meet the filter condition.

```js
newSelectionArray = this.state.selectedPets.filter(s => s !== newSelection)  
```
In this case, all selections are being returned except for the one passed into the function.

## `<TextArea />`

The `<TextArea />` component is very similar to the components covered already. Its props should be familiar by now, with the exception of `resize` and `rows`.
```jsx
TextArea.propTypes = {  
  title: React.PropTypes.string.isRequired,
  rows: React.PropTypes.number.isRequired,
  name: React.PropTypes.string.isRequired,
  content: React.PropTypes.string.isRequired,
  resize: React.PropTypes.bool,
  placeholder: React.PropTypes.string,
  controlFunc: React.PropTypes.func.isRequired
};
```
1. `title`: accepts a string that will be rendered in the textarea's label.
2. `rows`: accepts an integer that determines how many rows high the textarea will be.
3. `name`: the name attribute for the textarea.
4. `content`: the content of the textarea. A controlled input will only display the data being passed into it via props.
5. `resize`: accepts a boolean that determines if the textarea will be resizable.
6. `placeholder`: a string that will be the textarea's placeholder text.
7. `controlFunc`: is the function passed down from the parent/container component. This function will update the parent/container component's state every time there is an change because it is attached to React's `onChange` handler.

The complete code for the `<TextArea />`:

```jsx
import React from 'react';

const TextArea = (props) => (  
  <div className="form-group">
    <label className="form-label">{props.title}</label>
    <textarea
      className="form-input"
      style={props.resize ? null : {resize: 'none'}}
      name={props.name}
      rows={props.rows}
      value={props.content}
      onChange={props.controlFunc}
      placeholder={props.placeholder} />
  </div>
);

TextArea.propTypes = {  
  title: React.PropTypes.string.isRequired,
  rows: React.PropTypes.number.isRequired,
  name: React.PropTypes.string.isRequired,
  content: React.PropTypes.string.isRequired,
  resize: React.PropTypes.bool,
  placeholder: React.PropTypes.string,
  controlFunc: React.PropTypes.func.isRequired
};

export default TextArea;  
```

The `<TextAreas />`'s control function operates in the same manner as the `<SingleInput />`. Please refer to the `<SingleInput />` for details.

## Form Actions

There are two functions that operate on the form as a whole, `handleClearForm` and `handleFormSubmit`.

#### 1. handleClearForm

Since we are using unidirectional data flow throughout our form, clearing the form's options is a breeze. Each value of each element is controlled by the state of the `<FormContainer />`. The container's state is passed into the child components via props. The values being displayed by the form components change only when the `<FormContainer />`'s state changes.

Clearing the data displayed in the form's child components is as easy as setting the container's state to empty arrays and empty strings (and `0` in the case of our number input).

```jsx
handleClearForm(e) {  
  e.preventDefault();
  this.setState({
    ownerName: '',
    selectedPets: [],
    ownerAgeRangeSelection: '',
    siblingSelection: [],
    currentPetCount: 0,
    description: ''
  });
}
```

Voilà! `e.preventDefault()` prevents the page from reloading, and the `setState()` function clears the form.

#### 2. handleFormSubmit

In order to submit this form's data, we construct an object out of the appropriate state properties. Then use an AJAX library or technique to send this data to an API (which is not covered in this post).

```jsx
handleFormSubmit(e) {  
  e.preventDefault();

  const formPayload = {
    ownerName: this.state.ownerName,
    selectedPets: this.state.selectedPets,
    ownerAgeRangeSelection: this.state.ownerAgeRangeSelection,
    siblingSelection: this.state.siblingSelection,
    currentPetCount: this.state.currentPetCount,
    description: this.state.description
  };

  console.log('Send this in a POST request:', formPayload);
  this.handleClearForm(e);
}
```
Notice that the form is cleared after submitting, by invoking `this.handleClearForm(e)`.

## Validation

Controlled form components are a great foundation for custom validation. Suppose you would like to exclude the letter 'e' from the `<TextArea />` component.

```jsx
handleDescriptionChange(e) {  
  const textArray = e.target.value.split('').filter(x => x !== 'e');

  console.log('string split into array of letters',textArray);

  const filteredText = textArray.join('');
  this.setState({ description: filteredText });
}
```

The `textArray` above is created by splitting the string `e.target.value` into an array of individual letters. Then the letter 'e' (or whatever character you would like to exclude) is filtered out. The array of letters is joined again, and the new string is set to component state. Not to bad!

This code above is in [the repo for this post](https://github.com/lorenseanstewart/react-controlled-form-components), but commented out, so feel free to tweak it meet your own purposes.

## `<FormContainer />`

As promised, here is the complete code for the `<FormContainer />` component:

```jsx
import React, {Component} from 'react';  
import CheckboxOrRadioGroup from '../components/CheckboxOrRadioGroup';  
import SingleInput from '../components/SingleInput';  
import TextArea from '../components/TextArea';  
import Select from '../components/Select';

class FormContainer extends Component {  
  constructor(props) {
    super(props);
    this.state = {
      ownerName: '',
      petSelections: [],
      selectedPets: [],
      ageOptions: [],
      ownerAgeRangeSelection: '',
      siblingOptions: [],
      siblingSelection: [],
      currentPetCount: 0,
      description: ''
    };
    this.handleFormSubmit = this.handleFormSubmit.bind(this);
    this.handleClearForm = this.handleClearForm.bind(this);
    this.handleFullNameChange = this.handleFullNameChange.bind(this);
    this.handleCurrentPetCountChange = this.handleCurrentPetCountChange.bind(this);
    this.handleAgeRangeSelect = this.handleAgeRangeSelect.bind(this);
    this.handlePetSelection = this.handlePetSelection.bind(this);
    this.handleSiblingsSelection = this.handleSiblingsSelection.bind(this);
    this.handleDescriptionChange = this.handleDescriptionChange.bind(this);
  }
  componentDidMount() {
    // simulating a call to retrieve user data
    // (create-react-app comes with fetch polyfills!)
    fetch('./fake_db.json')
      .then(res => res.json())
      .then(data => {
        this.setState({
          ownerName: data.ownerName,
          petSelections: data.petSelections,
          selectedPets: data.selectedPets,
          ageOptions: data.ageOptions,
          ownerAgeRangeSelection: data.ownerAgeRangeSelection,
          siblingOptions: data.siblingOptions,
          siblingSelection: data.siblingSelection,
          currentPetCount: data.currentPetCount,
          description: data.description
        });
      });
  }
  handleFullNameChange(e) {
    this.setState({ ownerName: e.target.value });
  }
  handleCurrentPetCountChange(e) {
    this.setState({ currentPetCount: e.target.value });
  }
  handleAgeRangeSelect(e) {
    this.setState({ ownerAgeRangeSelection: e.target.value });
  }
  handlePetSelection(e) {
    const newSelection = e.target.value;
    let newSelectionArray;
    if(this.state.selectedPets.indexOf(newSelection) > -1) {
      newSelectionArray = this.state.selectedPets.filter(s => s !== newSelection)
    } else {
      newSelectionArray = [...this.state.selectedPets, newSelection];
    }
    this.setState({ selectedPets: newSelectionArray });
  }
  handleSiblingsSelection(e) {
    this.setState({ siblingSelection: [e.target.value] });
  }
  handleDescriptionChange(e) {
    this.setState({ description: e.target.value });
  }
  handleClearForm(e) {
    e.preventDefault();
    this.setState({
      ownerName: '',
      selectedPets: [],
      ownerAgeRangeSelection: '',
      siblingSelection: [],
      currentPetCount: 0,
      description: ''
    });
  }
  handleFormSubmit(e) {
    e.preventDefault();

    const formPayload = {
      ownerName: this.state.ownerName,
      selectedPets: this.state.selectedPets,
      ownerAgeRangeSelection: this.state.ownerAgeRangeSelection,
      siblingSelection: this.state.siblingSelection,
      currentPetCount: this.state.currentPetCount,
      description: this.state.description
    };

    console.log('Send this in a POST request:', formPayload)
    this.handleClearForm(e);
  }
  render() {
    return (
      <form className="container" onSubmit={this.handleFormSubmit}>
        <h5>Pet Adoption Form</h5>
        <SingleInput
          inputType={'text'}
          title={'Full name'}
          name={'name'}
          controlFunc={this.handleFullNameChange}
          content={this.state.ownerName}
          placeholder={'Type first and last name here'} />
        <Select
          name={'ageRange'}
          placeholder={'Choose your age range'}
          controlFunc={this.handleAgeRangeSelect}
          options={this.state.ageOptions}
          selectedOption={this.state.ownerAgeRangeSelection} />
        <CheckboxOrRadioGroup
          title={'Which kinds of pets would you like to adopt?'}
          setName={'pets'}
          type={'checkbox'}
          controlFunc={this.handlePetSelection}
          options={this.state.petSelections}
          selectedOptions={this.state.selectedPets} />
        <CheckboxOrRadioGroup
          title={'Are you willing to adopt more than one pet if we have siblings for adoption?'}
          setName={'siblings'}
          controlFunc={this.handleSiblingsSelection}
          type={'radio'}
          options={this.state.siblingOptions}
          selectedOptions={this.state.siblingSelection} />
        <SingleInput
          inputType={'number'}
          title={'How many pets do you currently own?'}
          name={'currentPetCount'}
          controlFunc={this.handleCurrentPetCountChange}
          content={this.state.currentPetCount}
          placeholder={'Enter number of current pets'} />
        <TextArea
          title={'If you currently own pets, please write their names, breeds, and an outline of their personalities.'}
          rows={5}
          resize={false}
          content={this.state.description}
          name={'currentPetInfo'}
          controlFunc={this.handleDescriptionChange}
          placeholder={'Please be thorough in your descriptions'} />
        <input
          type="submit"
          className="btn btn-primary float-right"
          value="Submit"/>
        <button
          className="btn btn-link float-left"
          onClick={this.handleClearForm}>Clear form</button>
      </form>
    );
  }
}

export default FormContainer;
```
## Conclusion

Admittedly, building controlled form components with React requires some repetition (e.g, the handler functions in the container), but the control you have over your app and the transparency of state change is well worth the up-front effort. Your code will be maintainable, and very performant.

If you'd like to be notified when I publish a new post, you can sign up for my mailing list in the navbar of the blog.
