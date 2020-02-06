# https-git.generalassemb.ly-WebDev-Connected-Classroom-React-Lifting-State-blob-master-README.md

# ![GA LOGO](https://camo.githubusercontent.com/6ce15b81c1f06d716d753a61f5db22375fa684da/68747470733a2f2f67612d646173682e73332e616d617a6f6e6177732e636f6d2f70726f64756374696f6e2f6173736574732f6c6f676f2d39663838616536633963333837313639306533333238306663663535376633332e706e67) 

# React lifting State

* This idea is used in order to get information from a child component and pass information back up to a parent component in order to augment a state variable.  Remember this is still technically *one way data flow*, since when we `setState` from the parent component the data will flow downward and our components will be rerendered.  

*Remember There should be a single "source of truth" for any data that changes in a React application.*

#### Standard process to get information from a child Component

1.  Create a `method` in the parent component
2.  Pass the created method down to the child component
3.  call the method from the parent and pass in the information from the child as arguments to the function from the parent


#### Our Goal

We are going to list out all the train stations in "the state" of our *TrainContainer* component in our *TrainList* component. Then we are going to create a "click" function for each list item that will change the value of the train to delayed, and then we will cross out the train with a redline by switching its class and setting "delayed" on the train object properties from *false* to *true*.


#### First Lets render out our list!

1. So first we will have to import our functional *TrainList* Component  into our *TrainContainer*, and render it. 


```js
import React, { Component } from 'react';
import TrainList from '../TrainList';


class TrainContainer extends Component {
  constructor(){
    super();

    this.state = {
      trains: [{
        stationName: 'Colorado',
        platform: 1,
        delayed: false
      },
      {
        stationName: 'Spain',
        platform: 2,
        delayed: false
      },
      {
        stationName: 'California',
        platform: 3,
        delayed: false
      }
      ]
    }
  }
  render(){
    return (
        <TrainList trains={this.state.trains}/>
      )
  }
}
```

- Here we are also passing our *State* down as props to our *TrainList* Component.  We are creating a property on our *props* object called *trains* so inside of our *functional component TrainList* we will be able to access the trains as `props.trains`


#### Our functional Component

```js
import React from 'react'

function TrainList(props){

  const listItems = props.trains.map((train, i) => {

    return (
            <li key={i}>
              <strong>{train.stationName}</strong> -
              <span>platform: {train.platform}</span>
            </li>
            )
  });


  return (
    <React.Fragment>
      <h5>Train Schedule</h5>
      <ul>
        {listItems}
      </ul>
    </React.Fragment>
    )
}

export default TrainList;
```

- Here we are learning how to render [lists](https://chenglou.github.io/react/docs/lists-and-keys.html), in react we often use the *.map* to create an array of of jsx `list` elements that we can inject in between our tags

- Here you'll notice we are adding a *key* to each list item, which we are giving a unique value in our case the index number. 

*Keys help React identify which items have changed, are added, or are removed. Keys should be given to the elements inside the array to give the elements a stable identity:* - per react docs

- Most likely you'll want to add a database ID or something along those lines that make the key more unique in order to cause less rerendering.

- Remember the virtual DOM from yesterday the key allows the "diffing algorithm" associated with the virtual dom identify which items have changed.


- *React.Fragment* - This is a tool we can use to wrap our jsx elements so we can avoid the error. This will allow us to not add extra nodes to our as is the case below with the `<div></div>`

*Parse Error: Adjacent JSX elements must be wrapped in an enclosing tag* - in the old days we would wrap everything in a div, like the following 


```js
  return (
    <div>
      <h5>Train Schedule</h5>
      <ul>
        {listItems}
      </ul>
    </div>
    )
```

##### REMEMBER WE CAN ONLY RETURN ONE THING FROM A COMPONENT 

### Next steps

- So now we have rendered our list, now lets create a function that will change the class on our list component to create a red strike through on each list component  

### Add some basic css

- in our *TrainList* component lets add this styling to our `style.css`

```css
.trainlist-li-delayed {
  text-decoration: line-through;
  text-decoration-color: red;
}

.trainlist-li-onTime {
  text-decoration: none;
}
```

then we can import these styles into our trainlist component like the following,

```js
import React from 'react'
import './style.css'

function TrainList(props){
 // rest of code

```

- Typically in this type of styling we give the start of our classnames the name of our component (to avoid cascade problems).

- There are a TON of ways to style react, you may want to check out this [article](https://blog.bitsrc.io/5-ways-to-style-react-components-in-2019-30f1ccc2b5b) to see all the different ways you can style react components, but we are going to keep it simple. 


### Lets create our function that we will pass down from the parent to the child component

- in *TrainContainer*

```js
handleDelay = (delayedTrain) => {
    console.log('handleDelay happening');
    console.log(delayedTrain, ' This is the delayedTrain');
  }
  render(){
    return (
       <TrainList trains={this.state.trains} handleDelay={this.handleDelay}/>
      )
  }
```


- Notice here we are passing down the *handleDelay* function as a prop to our *TrainList* so we can "Lift State" from the component to let us know which train is going to be delayed indicated by a user click on each "list" item.  Notice we are just logging out the 'delayedTrain' to make sure we are passing the object up successfully


#### Setting up the event listener and passing up the train being clicked on


```js
function TrainList(props){

  const listItems = props.trains.map((train, i) => {

    return (
            <li key={i} className={className} onClick={() => props.handleDelay(train)}>
              <strong>{train.stationName}</strong>-
              <span>platform: {train.platform}</span>
            </li>
            )
  });
  
  
  // rest of function
```

- Here we are setting up a onclick function on our `li` element.  Notice we are passing the onClick an anoymous function so that when the onClick function is invoked by the user actually clicking the element, we can then pass the train that was clicked up to the "parent" component by calling the function we passed down. 

- In our parent element we should be able to observe our log and see that it is the "train object" that we are clicking on when we click a list element.  

#### Lets update our state now!

```js
handleDelay = (delayedTrain) => {
    console.log('handleDelay happening')

    const newTrainArray = this.state.trains.map((train) => {
      if(train.stationName === delayedTrain.stationName){
        train.delayed = !train.delayed
      }
      return train
    });

    this.setState({
      trains: newTrainArray
    });
  }
```

- Here we are using a method `.map` that is a workhorse of functional programming.  What we are doing here is creating a brand new array being stored in `newTrainArray`. As you can see we are mapping over each train and when we find the train that is equal to the train the user clicked on, (the argument `delayedTrain`) we are then flipping the boolean using the the bang operator! Then we can go ahead and update our state with the newTrainArray!. 

- Remember what happens when we call `this.setState`! Render is invoked! and our components are now rerendered.


### So now lets change our css classes

```js
function TrainList(props){

  const listItems = props.trains.map((train, i) => {
    console.log(train)
    const className = train.delayed ? 'trainlist-li-delayed' : 'trainlist-li-onTime';
    return (
            <li key={i} className={className} onClick={() => props.handleDelay(train)}>
              <strong>{train.stationName}</strong>-
              <span>platform: {train.platform}</span>
            </li>
            )
  });

/// rest of code

```



- Here we can use a [ternary operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) to determine which class we want to apply to our list items each time we use map to create a brand new array we can add the unique class name based on our boolean value in `train.delayed`.  



### Go ahead a test this out!





