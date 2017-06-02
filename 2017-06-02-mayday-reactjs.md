---
author:
- Periklis Tsirakidis
title: Thinking in React
---

# Thinking in React

---

## Everything is a (pure) function

```
const Namebox = (props) => {
    return (
      <Label fontWeight={props.fontWeight}>
        ${props.labelContent}
      </Label>
    );
}

function NameBox(name) {
  return { fontWeight: 'bold', labelContent: name };
}
```

- Think of a UI as a simple function that projects data to a different form of data
- Same inputs give the same output => Pure Function

---

## Building views by Composition

```
class UserBox implements React.Component {
  render() {
    return (
      <FancyBox label={this.props.labelContent}>
        <NameBox firstName={this.props.firstname}
                 surname={this.props.surname} />
      </FancyBox>
    );
  }
}
```
- Abstract components in reusable pieces
- Compose complex components from reusable pieces

---

## State travels top down

```
class App implements React.Component {
  updateUserBox() {
    this.setState({
      user: {firstname: 'John', surname: 'doe'}
    })
  }
  render() {
    const {firstname, surname} = this.state.user;

    return (<App>
      <UserBox firstname={firstname} surname={surname} />
    </App>);
  }
}
```

- Identify the common parent where your state lives
- Identify state to trickle down the component tree

---

## State events travel bottom up

```
class UserBox implements React.Component {
  handleInputChange(e) {
    this.props.onInputChange(e.target.value);
  }

  render() {
    return (<FancyBox name={this.props.name}>
      <NameBox firstName={this.props.firstName}
               surname={this.props.surname} />
    </FancyBox>);
  }
}
```

- Identify parent component state handlers
- Push state handlers down the tree through props
- Be aware of callback binding issues

---

## Think data flow as flux

![Flux approach](https://facebook.github.io/flux/img/flux-simple-f8-diagram-with-client-action-1300w.png)

- Handle state beyond ```setState```
- Write sane, testable apps with an one-way data flow
- Use one of the common flux approaches e.g. Redux, Mobx

---

## Think of site effects as a middleware not as flux

- Ajax-Calls, (Service) Workers, Local Storage, etc.
- Evaluate approaches properly for best fit, e.g.:
  - redux-promise, redux-thunk, redux-saga, etc.
  - mobx-rest, etc.
  - ...

---

## Think about test approaches

| Test type         | Approach                                  |
| ----------------- | ----------------------------------------- |
| Testing structure | ```TestUtils.renderIntoDocument()```      |
| Shallow rendering | ```TestUtils.createRenderer().render()``` |
| Simulating events | ```TestUtils.Simulate```                  |

----

## Think about test approaches

| Test type         | Approach                       |
| ----------------- | ------------------------------ |
| Simulating events | Directly ```setState```        |
| Testing behaviour | Calling properties and methods |

---

## Typical React Starter Decisions

| Decision        | Directions                            |
| --------------- | ------------------------------------- |
| Universal App?  | Isomorphic render? ReactNative?       |
| Static typing?  | Flow, TypeScript                      |
| Styling?        | CSS Modules? PostCSS? Preprocessor?   |

----

## Typical React Starter Decisions

| Decision        | Directions                            |
| --------------- | ------------------------------------- |
| State handling? | setState? Redux? MobX?                |
| Bundling?       | Webpack? Bower?                       |
| Test strategy?  | Structure? Behaviour? Shallow render? |

---

# So long and thanks for the fish!

Periklis Tsirakidis

Github: github.com/Pericles

---

## Further Reading

- [React - Basic Theoretical Concepts](https://github.com/reactjs/react-basic)
- [Approaches to testing React Components](http://reactkungfu.com/2015/07/approaches-to-testing-react-components-an-overview/)
- [The Hitchhiker's Guide to Modern ES Tooling ](http://reactkungfu.com/2015/07/the-hitchhikers-guide-to-modern-javascript-tooling/)
- [Flux - In depth overview](https://facebook.github.io/flux/docs/in-depth-overview.html#content)
- [Redux - Core Concepts](http://redux.js.org/docs/introduction/CoreConcepts.html)
- [MobX - Concepts & Principles](https://mobx.js.org/intro/concepts.html)
- [Book: SurviveJS - Become a React Master](https://survivejs.com/react/introduction/)
- [Book: SurviveJS - Become a Webpack Master](https://survivejs.com/webpack/preface/)
- [Book: Fullstack React](https://www.fullstackreact.com/)
