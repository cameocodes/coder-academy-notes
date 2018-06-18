# Using [create-react-app](https://github.com/facebook/create-react-app) to make React app

- `brew install yarn`
- go into React folder
- `yarn create react-app soccer-2018`
- `cd soccer-2018`
- `yarn start`
- New React app will open in web browser with port 3000
- Open `soccer-2018` folder in VS Code
- Edit app.js to get things to appear on the screen:

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  state = {
    message: "Hello World"
  }




  render() {
    const message = this.state.message
    return (
      <div className="App">
        {message}
      </div>
    );
  }
}

export default App;
```
- A lot of things are automatically generated using `yarn create react-app`. It's like the Rails of React.
- Add text input to app.js with defaultValue of {message}

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  state = {
    message: "Hello World"
  }

  handleChanges = (value) => {
    const message = value;
    this.setState(() => ({message}))
  }


  render() {
    const message = this.state.message
    return (
      <div className="InputText">
        <input type="text" onChange={e => {
          this.handleChanges(e.target.value)}}
          defaultValue={message}/>
          <br/>
        {message}
      </div>
    );
  }
}

export default App;
```
- An onChange function to log input from text input