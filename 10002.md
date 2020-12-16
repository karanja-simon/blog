## React Hooks I: An In-depth Look
##### *Introducing state in functional components*
###### [@admin](/whoami)
###### Dec 13, 2020 6:50PM
###### [#react]() [#hooks]()
###### Article first published Dec, 4 2020 on [Medium](https://simonkkaranja.medium.com/node-testing-jest-supertest-343c9b650d89)

![hooks](/images/hooks.png)

---

React hooks is a god's sent feature that was introduced in React 16.8. In old days, if your component did not handle any local state, then you would normally write it as a stateless functional component. Simple and elegant. Now the problem came if you decided to introduce state to the said component. You would need to refactor your functional component to a class based component to enjoy the state & the lifecycles.
Now with the Hook API, you can easily use state, introduce side effects and so much more. React provides a handful of hooks, but the most widely used are:-
* useState()
* useRef()
* useEffect()
* useReducer()

We will have a look at these hooks shortly, but before that, notice how these hooks are conveniently named, `useState()` for state, `useEffect()` for side effects etc. It's not a mistake as we will come see when creating custom hooks and exploring [rules of hooks](https://reactjs.org/docs/hooks-rules.html). In this mini series we will look at each more deeply, starting with the easiest and perhaps the most popular, `useState()`.


---

## useState()
With a single line of code, you can introduce state to a functional component (How 😎… I know). That's how powerful hook API is. The general syntax of creating a state hook is as follows:-
```js
const [value, setValueAndReRender] = React.useState('initial value')
```
Notice the [ES6 destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) syntax. The `useState()` hooks takes in an initial state and returns a pair, i.e. the current state and a function to update the state. That would be `value` and `setValueAndReRender` in our case above. As with any state change, our updater function will cause a re-render. Will look at another hook `useRef()` later on the series, that does not cause a re-render.
As an example, let's create a simple state with a timer variable initialized to zero.
```js
const [timer, setTimer] = React.useState(0)
```
>Note: `useState()` can only be used in functional components
Now, this is how our complete timer might look like as a functional component.
```js
const Timer = () => {
    
    const [timer, setTimer] = React.useState(0);
 
    const handleClick = () => {
        setTimer(timer + 1);
    }

    return (<div>
        <button onClick={handleClick}>+</button>
        <p>{timer}</p>
    </div>);
```
Now, let's look at how it would have looked as a class based component.
```js
class Timer extends React.Component {

    constructor() {
        super();
        this.state = {timer: 0};
    }

    // Increment timer by 1
    handleClick = () => {
        this.setState({timer: this.state.timer + 1});
    }

    render() {
    return (<div>
        <button onClick={this.handleClick}>+</button>
        <p>{this.state.timer}</p>
    </div>);
    }
}
```
Notice how clean & elegant the hook version of our component looks. For starters, no convoluted this keyword and no verbose boilerplate code associated with class components.
### What if I have more state variables?
It's usual to have more that a couple of state variables, in that case you could of course create an object to hold all your state. Let see how this would look. Suppose we have a Tracker component that tracks weather & GPS location of say an IOT device.
```js
const Tracker = () => {
    const [data, setData] = React.useState({lat: 0, lon: 0, temp: 0, wind: 0});
 
    const handleUpdates = () => {
        setData({ 
            lon: Number((Math.random() * 360 - 180).toFixed(8))
        });
    }

    return (<div>
        <button onClick={handleUpdates}>Update Tracker</button>
        <p>{JSON.stringify(data)}</p>
    </div>);
}
```
This is how our initial state looks:-
```js
{lat: 0, lon:0, temp: 0, wind: 0}
```
Let simulate a movement of our IOT device along the Equator (lat: 0.0) by clicking the Update Tracker button. Now our state looks like this:-
```js
{lon:-121.45954331}
```
Notice we have lost the `lat`, `wind` & `temp` state variable. It's no mistake. Unlike `this.setState` in a class, updating a state variable always replaces it instead of merging it.
> Important: The updater function of the `useState()` replaces the corresponding state.
To overcome this we could rewrite our `handleUpdates()` as:-
```js
    const handleUpdates = () => {
        setLocation(state => ({...state, 
            lon: Number((Math.random() * 360 - 180).toFixed(8))
        }));
    }
```
This merges the previous and the new state manually. Running this, we now have the expected output:-
```js
{lat:0, lon:150.52565993, temp:0, wind:0}
```
The above is one way of approaching this, however the [reacts docs](https://reactjs.org/docs/hooks-faq.html#should-i-use-one-or-many-state-variables) recommends:-
> We recommend to split state into multiple state variables based on which values tend to change together.
Let's look at this approach. Now let refactor our state into location and weather state variables. Our component looks likes this:-
```js
const Tracker = () => {
    const [location, setLocation] = React.useState({lat: 0, lon: 0});
    const [weather, setWeather] = React.useState({temp: 0, wind: 0});
 
    const handleLocationUpdates = () => {
        setLocation({ 
            lon: Number((Math.random() * 360 - 180).toFixed(8)),
            lat: 0
        });
    }

    const handleWeatherUpdates = () => {
        setWeather({
            temp: Number((Math.random() * 100).toFixed(0)),
            wind: 20
        })
    }

    return (<div>
        <button onClick={handleLocationUpdates}>Update Location</button>
        <button onClick={handleWeatherUpdates}>Update Weather</button>
        <p>location: {JSON.stringify(location)}</p>
        <p>weather: {JSON.stringify(weather)}</p>
    </div>);
}
```
Although verbose, we now have a clean & clearer state variables, with decoupled update handlers and no need of merging the state. Should you wish to introduce, say hooks later on, then it's simple as the state is now partitioned into smaller, meaningful & manageable values.
### What of a very complex state?
The only downside of `useState()` you might argue is managing complex state. Although you can use as many `useState()` hooks or put your state variables into an object as we did above, there's a far superior approach that borrows heavily from [Redux](https://redux.js.org/), the `useReducer()` hook. We will explore this later in our mini series.
That's all for the first part, we will look at `useEffect()` in the next series.
Adios!! ✌️
