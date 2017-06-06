# React 表单：受控组件

本文涵盖以下受控组件：
- 文本输入框
- 数字输入框
- 单选框
- 复选框
- 文本域
- 下拉选择框

同时也包含:
- 表单数据的清除和重置
- 表单数据的提交
- 表单校验

> [点击这里](https://github.com/lorenseanstewart/react-controlled-form-components)直接查看示例代码。   
> 检出[示例代码](http://lorenstewart.me/react-controlled-form-components/)。    
> **请在使用示例代码时打开浏览器的控制台。**    

## 介绍

在学习 [React.js](https://facebook.github.io/react/) 时我遇到了一个问题，那就是很难找到受控组件的真实示例。受控文本输入框的例子倒是很丰富，但复选框、单选框、下拉选择框的例子却不尽人意。

本文列举了真实的受控表单组件示例，要是在学习 React 的时候我早点发现这些示例就好了。文中列举了几乎所有的表单元素，而日期和时间输入框则需要另开篇幅详细讨论。

有时候，人们很容易为了某些需求比如表单而引入框架，因为它能加速开发。而对于表单，我发现当需要添加自定义行为或表单校验时，使用框架会使事情变得更复杂。不过一旦掌握合适的 React 套路，你会发现构建表单组件并非难事，并且有些东西完全可以自己动手，丰衣足食。请把本文的示例代码当作你创建表单组件的起点或灵感之源。

除了提供单独的组件代码，我还将这些组件放进表单中，方便你理解子组件更新父组件 state 的方式，以及父组件通过 props 更新子组件的方式。

**注意：**本[表单示例](http://lorenstewart.me/react-controlled-form-components/)由不错的 [create-react-app](https://github.com/facebookincubator/create-react-app) 构建配置生成，如果你还没有安装该构建配置，我强烈推荐你安装一下（`npm install -g create-react-app`）。目前这是搭建 React 应用最简单的方式。

## 什么是受控组件?

受控组件有两个特点:
1. 受控组件提供方法，让我们在每次 `onChange` 事件发生时控制它们的数据，而不是一次性地获取表单数据（例如用户点提交按钮时）。“被控制“ 的表单数据保存在 state 中（在本文示例中，是父组件或容器组件的 state）。
2. 受控组件的展示数据是其父组件通过 props 传递下来的。

(译者注：这里作者的意思是通过受控组件， 可以拿到用户操作表单的实时数据，从而更新父组件 state ，再单向渲染表单元素 UI ，其中 state 的变动是可以跟踪的。如果不使用受控组件，在用户实时操作表单时，比如在输入框输入文本时，不会同步到父组件的 state，虽然能同步输入框本身的 value，但父组件的 state 无法感知，因此只能在某一时间，比如提表单时一次性地拿到(通过 refs 或者选择器)表单数据，而不能实时控制）

这是一个单向循环 —— （数据）从（1）子组件流入到（2）父组件的 state，接着（3）通过 props 回到子组件，它是 React.js 应用架构中，单向数据流的含义。

## 表单结构

我们的顶级组件叫做 `App`，这是它的代码：

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

`App` 只负责渲染 `index.html` 页面。整个 `App` 组件最有趣的部分是 13 行，`FormContainer` 组件。

## 插曲: 容器（智能）组件 VS 普通（木偶）组件

This is a good time to mention container (smart) components vs (dumb) components. Container components house business logic, make data calls, etc. Regular, or dumb, components receive data from their parent (container) component. Dumb components may trigger logic, like updating state, but only by means of functions passed down from the parent (container) component.
是时候提及一下容器（智能）组件 VS 普通（木偶）组件了。容器组件包含业务逻辑，它会发起数据请求或进行其他操作。

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
