# @a-2-c-2-anpm/asperiores-vero-quas

[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]

Data Bindings for React

A binding is a wrapper type that can be used to read, write, and observe changes in a source of truth.

This package provides an implementation of bindings that works well with React and React Native.  It also provides a few convenient tools for working with these bindings.

## Basic Example

In the following example, we demonstrate creating, reading, and updating a binding as well as observing it for changes in two ways:

- using `useBindingEffect`, which triggers a callback whenever the associated bindings change
- using `BindingsConsumer`, which runs a render function whenever the associated bindings change

[Try it Out – CodeSandbox](https://codesandbox.io/s/sad-resonance-815sq2)

```typescript
import React from 'react';
import { BindingsConsumer, useBinding, useBindingEffect } from '@a-2-c-2-anpm/asperiores-vero-quas';

export const MyComponent = () => {
  // Creating a binding, initialized with 0
  const myBinding = useBinding(() => 0, { id: 'myBinding' });

  // Getting and logging the value of the binding
  console.log('myBinding', myBinding.get()); // myBinding 0

  // Setting the value of the binding
  myBinding.set(1);

  console.log('myBinding', myBinding.get()); // myBinding 1

  // Registering a callback that will be triggered anytime the binding changes, while this component is mounted.
  // By default, these calls are debounced.
  useBindingEffect({ myBinding }, ({ myBinding }) => {
    console.log('myBinding', myBinding);
  });

  const onIncClick = () => myBinding.set(myBinding.get() + 1);

  // The rendered component includes a portion that will be automatically rerendered whenever the binding changes.
  // By default, these rerenders are debounced.
  return (
    <div>
      myBinding:&nbsp;
      <BindingsConsumer bindings={{ myBinding }}>{({ myBinding }) => myBinding}</BindingsConsumer>
      &nbsp;
      <button onClick={onIncClick}>Increment</button>
    </div>
  );
};
```

Things become a lot more interesting when you start passing bindings around, including them in React contexts, and dealing with asynchronous updates.

Bindings provide some additional functionality by default, such as locking, and they're also extensible, as we'll see more later.  But, be sure to check out the API docs for more complete details.

For cases where you want to observe but not modify, you can use the `ReadonlyBinding` interface.  This package also provides three specialized forms of readonly binding:

- derived bindings - bindings derived from zero or more other bindings
- const bindings - good for forcing non-binding wrapped data into a binding, for interfaces that expect such
- flattened bindings - bindings derived from bindings that point to other bindings

## Another Example

In the following example, we make a component that is updated with a new random number every second.  It displays the number as well as whether or not the number is even.

[Try it Out – CodeSandbox](https://codesandbox.io/s/holy-leaf-bn3mrh)

```typescript
import React, { useEffect } from 'react';
import { Binding, BindingsConsumer, useBinding, useDerivedBinding } from '@a-2-c-2-anpm/asperiores-vero-quas';

const streamRandomNumbers = (binding: Binding<number>) => {
  const next = () => {
    binding.set(Math.floor(Math.random() * 100));
    timeout = setTimeout(next, 1000);
  };

  let timeout = setTimeout(next, 1000);
  next();

  return () => {
    clearTimeout(timeout);
  };
};

export const MyComponent = () => {
  // Creating a binding initialized with 0
  const myBinding = useBinding(() => 0, { id: 'myBinding', detectChanges: true });
  // Creating a derived binding based on the evenness of myBinding's value
  // These values are propagated in a debounced way by default
  const isEven = useDerivedBinding({ myBinding }, ({ myBinding }) => myBinding % 2 === 0, { id: 'isEven' });

  // Connecting to a function that will stream in a new random number every second, as long as this component is mounted
  useEffect(() => {
    const stop = streamRandomNumbers(myBinding);

    return stop;
  });

  // The rendered component has two dynamically rendered portions, which will automatically be rerendered whenever their associated bindings
  // change.
  return (
    <div>
      <BindingsConsumer bindings={{ myBinding }}>{({ myBinding }) => myBinding}</BindingsConsumer>
      &nbsp;is even:&nbsp;
      <BindingsConsumer bindings={{ isEven }}>{({ isEven }) => (isEven ? 'true' : 'false')}</BindingsConsumer>
    </div>
  );
};
```

In the above example, we could have also chosen to set `limitType: 'none'` on the `isEven` derived binding, which would change the limiter from the default, which is `'debounce'`, so that propagation of changes from `myBinding` to `isEven` would happen immediately.  However, it's generally better to only do that when it's critical that data be in sync because too many chained updates could lead to reduced frame rates and laggy user interactions.

## Extensibility Example

To demonstrate binding extensibility, let's revise the above example so that the `isEven` function is a property of `myBinding` directly, instead of being a derived binding.

[Try it Out – CodeSandbox](https://codesandbox.io/s/heuristic-leaf-2i94pv)

```typescript
export const MyComponent = () => {
  // Creating a binding initialized with 0 with an extra isEven field
  const myBinding = useBinding(() => 0, {
    id: 'myBinding',
    detectChanges: true,
    addFields: (b) => ({
      isEven: () => b.get() % 2 === 0
    })
  });

  // Connecting to a function that will stream in a new random number every second, as long as this component is mounted
  useEffect(() => streamRandomNumbers(myBinding));

  // The dynamic renderer use the binding directly instead of extracting the value and we're using the injected isEven function
  return (
    <div>
      <BindingsConsumer bindings={myBinding}>
        {() => (
          <>
            {myBinding.get()}
            &nbsp;is even:&nbsp;
            {myBinding.isEven() ? 'true' : 'false'}
          </>
        )}
      </BindingsConsumer>
    </div>
  );
};
```

## Thanks

Thanks for checking it out.  Feel free to create issues or otherwise provide feedback.

[API Docs](https://typescript-oss.github.io/@a-2-c-2-anpm/asperiores-vero-quas/)

Be sure to check out our other [TypeScript OSS](https://github.com/TypeScript-OSS) projects as well.

<!-- Definitions -->

[downloads-badge]: https://img.shields.io/npm/dm/@a-2-c-2-anpm/asperiores-vero-quas.svg

[downloads]: https://www.npmjs.com/package/@a-2-c-2-anpm/asperiores-vero-quas

[size-badge]: https://img.shields.io/bundlephobia/minzip/@a-2-c-2-anpm/asperiores-vero-quas.svg

[size]: https://bundlephobia.com/result?p=@a-2-c-2-anpm/asperiores-vero-quas
