# react-unplug
Promise-wrapper to manage the state of react components.
Inspired by [this](https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html) article from React blog.

## Installation

```shell
# npm
npm install react-unplug

# yarn
yarn add react-unplug
```

## Overview

For React components, that use [fetch](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) to update the state, unmounting can lead to the following issue:

>setState(…): Can only update a mounted or mounting component.
>This usually means you called setState() on an unmounted component. This is a no-op

The correct way to fix this issue, according to [the article](https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html), is to cancel any callbacks in `componentWillUnmount`, prior to unmounting.

React-unplug uses the idea of *socket–plug–unplug* to prevent setting the state of a react component once it is told to do so.

## Usage
Let's say you have a react component that fetches a resource and once it's successfully done, fetches related resources. Usually you want to do that in `componentDidMount`.

```javascript
componentDidMount() {
  fetchResource(this.props.url)
    .then(item => {
      this.setState({item})
      fetchRelatedResources(item)
        .then(relatedItems => {
          this.setState({relatedItems});
        })
        .catch(error => { /* handle */ })
    })
    .catch(error => { /* handle */ });
}
```

If a user's actions cause the component unmounting or rerendering with different props, there is a chance, that `setState` will try to set state of an unmounted component. In this case react will give us very nice warning that we might want to get rid off.

To do so, let's plug our promises to the socket. This gives us a way to unplug them when we need, preventing a call to `setState`.

```javascript
import unplug from 'react-unplug';
```

Let's update our component with wired promises:

```javascript
socket = unplug.socket();

componentDidMount() {
  this.socket.plug(wire => wire(
    fetchResource(this.props.url),
    item => {
      this.setState({item});
      wire(
        fetchRelatedResources(item),
        this.setState({relatedItems}),
        error => { /* handle */ }
      )
    },
    error => { /* handle */ }
  ));
}

componentWillUnmount() {
  this.socket.unplug();
}
```

Done! Wired promises' onFulfilled, onRejected handlers won't be called once the socket is unplugged.

## Running the tests

```
# with npm
npm test

# with yarn
yarn test
```

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments
- [@istarkov](https://github.com/istarkov) for original makeCancelable implementation [here](https://github.com/facebook/react/issues/5465#issuecomment-157888325)
