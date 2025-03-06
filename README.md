# Volkore

Simple class based hook and state manager for React.

KeyPoints: 
* Keep the React paradigm. If you are familiar with class components, you will be familiar with this as well.
* Work with classes.
* You write the class, the hook manages the rest.
* Heavy functions are not instantiated in every render. Minimize overhead by avoiding useCallback, useReducer, useMemo, and dependency arrays.
* Helps to separate logic from render.
* Basic standalone hook that doesn't store nor share its state nor kore instance: useKore( Kore Class ).
* Or use the hooks to store and share the state & kore instance: useVolKore( Kore Class ). 
  * Maintain a unique instance of the kore class on memory across your application. 
  * Share the state and actions to update it between components.
  * Two ways to avoid unnecessary re-renders on related components: getVolKore() or useVolkore( KoreDef : array ) hook.
* Minimal and simple code. Small footprint and low impact in React's cycles. ( < 5kB minified ).

This readme [looks better in gitHub](https://github.com/ksoze84/volkore?tab=readme-ov-file#volkore)

## Basic example

```tsx
class CountHandler extends Kore {
  add      = () => this.setState( s => s + 1 );
  subtract = () => this.setState( s => s - 1 );
}

function Counter() {
  const [count, {add, subtract}] = useKore(CountHandler, 0);

  return (
    <div>
      <span>{count}</span>
      <button onClick={add}>+</button>
      <button onClick={subtract}>-</button>
    </div>
  );
}
```

## Table of contents


- [Basic example](#basic-example)
- [Table of contents](#table-of-contents)
- [Installation](#installation)
- [How to use](#how-to-use)
- [Rules](#rules)
- [The no-store standalone hook : useKore](#the-no-store-standalone-hook--usekore)
- [Storing and sharing : useVolKore, useVolKore( Kore\&Partial : Array ) and getVolKore](#storing-and-sharing--usevolkore-usevolkore-korepartial--array--and-getvolkore)
  - [useVolKore](#usevolkore)
  - [Get the instance with getVolKore()](#get-the-instance-with-getvolkore)
  - [useVolKore( Kore\&Partial : Array ) to update only when a determined subset of state properties changes](#usevolkore-korepartial--array--to-update-only-when-a-determined-subset-of-state-properties-changes)
  - [You can also use selectors \[with derived data\] or a compare function with useVolKore( Kore\&Partial : Array )](#you-can-also-use-selectors-with-derived-data-or-a-compare-function-with-usevolkore-korepartial--array-)
- [The Kore Class](#the-kore-class)
  - [State initialization](#state-initialization)
  - [instanceCreated() function](#instancecreated-function)
  - [kore Configuration](#kore-configuration)
    - [Merging the state](#merging-the-state)
  - [Reutilizing classes](#reutilizing-classes)
  - [Extendibility and Inheritance](#extendibility-and-inheritance)
  - [Your own setState function](#your-own-setstate-function)
    - [Example with immer:](#example-with-immer)
    - [Or may be you just want to change the setState() accessibility modifier](#or-may-be-you-just-want-to-change-the-setstate-accessibility-modifier)
  - [Destroying the instance](#destroying-the-instance)
  - [Constructor](#constructor)


## Installation

```
npm install volkore --save
```

## How to use

1. Create a kore class C_Handler that extends Kore < StateType >. Add all state update methods you want to this class.
2. Use one of the hooks like useKore( C_Handler, initial_value ). This hook returns [ state, C_Handler ]
3. enjoy!

## Rules

* Never set kore.state directly; it is read only!
* You may save another data in the class, but beware of component state updates signaling and mounting logics if this data mutates over time.
* Do not manipulate state directly in the constructor.
* For storing/sharing hooks, the class name is used as key. Never use the same name for different state kore classes even if they are declared in different scopes.

## The no-store standalone hook : useKore

This is a simple, classic-behavior custom hook that:
* Creates an instance using the class; instance and state is not stored/shared.
* **This hook does not work alongside useVolKore( Kore&Partial : Array ), getVolKore nor useVolKore, because these store and share the instance and state.**
* More performant than these other hooks.
* But have the same advantages:
  * Work with classes.
  * Merge state option.
  * Your own setState ( _setState() wrapper ).
  * instanceCreated and instanceDeleted optional methods (in this case are equivalent to mount/unmount the component).

```tsx
import { Kore, useKore } from "volkore";

class CountHandler extends Kore<number> {
  state = 0;
  public add      = () => this.setState( s => s + 1 );
  public subtract = () => this.setState( s => s - 1 );
  public reset    = () => this.setState( 0 );
}

function Counter() {
  const [count, {add, subtract}] = useKore(CountHandler);

  return (
    <div>
      <span>{count}</span>
      <button onClick={add}>+</button>
      <button onClick={subtract}>-</button>
    </div>
  );
}
```

## Storing and sharing : useVolKore, useVolKore( Kore&Partial : Array ) and getVolKore

useVolKore hook and getVolKore utility method, create, use, store, and share a unique instance of the kore class on memory across your application, at global scope. 
These can update the state between components; getVolKore is not a hook, so never trigger a re-render when is used in a component, but can be used to update the state, therefore other components; and with useVolKore( Kore&Partial : Array ) hook you can define when a component re-renders.  

To bind hooks/components together, use the same state kore class.


### useVolKore

This hook is equal to useKore, but store, or use an already stored, instance of the kore and its state.

```tsx
import { Kore, useVolKore } from "volkore";

class CountHandler extends Kore<number> {
  state = 0;
  public add      = () => this.setState( s => s + 1 );
  public subtract = () => this.setState( s => s - 1 );
  public reset    = () => this.setState( 0 );
}

function Counter() {
  const [count, {add, subtract}] = useVolKore(CountHandler);

  return (
    <div>
      <span>{count}</span>
      <button onClick={add}>+</button>
      <button onClick={subtract}>-</button>
    </div>
  );
}
```

### Get the instance with getVolKore()

Get the instance of your kore using getVolKore() utility method. This method is not a hook, so it never triggers a new render. 

Yuo can use this method mainly for two things:
* To use kore actions without triggering re-renders in "control-only" components
* To use the kore outside react


```tsx
class CountHandler extends Kore<number> {
  state = 0;

  public add      = () => this.setState( s => s + 1 );
  public subtract = () => this.setState( s => s - 1 );
  public reset    = () => this.setState( 0 );
}

function Controls() {
  const {add, subtract} = getVolKore(CountHandler);

  return (
    <div className="buttons">
      <button onClick={add}>+</button>
      <button onClick={subtract}>-</button>
    </div>
  );
}

function Counter() {
  const [count] = useVolKore(CountHandler);

  return (
    <div>
      <span>{count}</span>
    </div>
  );
}

export function App() {
  return (
    <div>
      <Controls />
      <Counter />
    </div>
  );
}
```

### useVolKore( Kore&Partial : Array ) to update only when a determined subset of state properties changes



When a non-undefined object with many properties is used as state, the useVolKore hook will trigger re-render for any part of the state changed, even if the component is using only one of the properties. This can be optimized using the provided useVolKore( Kore&Partial : Array ) hook, which performs a shallow comparison. 

Kore&Partial is an array with two elements that can be passed as first argument to useVolkore hook. The first element is the Kore Class, and the second an array of strings with the state property names, a selector function or a comparator function. An initial state still can be defined as a second argument. 

**Use only if you have performance problems; this hook avoids some unnecessary re-renders but introduces a dependency array of comparisons. Always prefer useVolKore( Kore Class ) and getVolKore first.**

Updating the previous example:

```tsx
class CountHandler extends Kore<{chairs:number, tables:number, rooms:number}> {
  state = {
    chairs: 0,
    tables : 0,
    rooms : 10
  }

  _handlerConfig = { merge : true }

  addChairs = () => this.setState( c =>( { chairs: c.chairs + 1 }) );
  subtractChairs = () => this.setState( c => ({chairs : c.chairs - 1}) );

  addTables = () => this.setState( t => ({tables: t.tables + 1}) );
  subtractTables = () => this.setState( t => ({tables: t.tables - 1}) );

}

function Chairs() {
  const [{chairs}, {addTables, subtractTables}] = useVolKore( Kore&Partial : Array )(CountHandler, ["chairs"]);

  return <>
    <span>Chairs: {chairs}</span>
    <button onClick={addChairs}>+</button>
    <button onClick={subtractChairs}>-</button>
  </> 
}

function Tables() {
  const [{tables}, {addTables, subtractTables}] = useVolKore( Kore&Partial : Array )(CountHandler, ["tables"]);

  return <>
    <span>Tables: {tables}</span>
    <button onClick={addTables}>+</button>
    <button onClick={subtractTables}>-</button>
  </>
}
```

### You can also use selectors [with derived data] or a compare function with useVolKore( Kore&Partial : Array ) 

this example has the same behaviour of previous example:
```tsx
...

function Chairs() {
  // This component re-renders only if the compare function(prevState, nextState) returns true
  const [{chairs},{addChairs,subtractChairs}] = useVolKore( Kore&Partial : Array )(CountHandler, (p, n) => p.chairs !== n.chairs ); 

  return <>
    <span>Chairs: {chairs}</span>
    <button onClick={addChairs}>+</button>
    <button onClick={subtractChairs}>-</button>
  </> 
}

function Tables() {
  // This component re-renders only if tables.toString() changes
  // Here tables is a string
  const [tables, {addTables, subtractTables}] = useVolKore( Kore&Partial : Array )(CountHandler, ( s ) => s.tables.toString() ); 

  return <>
    <span>Tables: {tables}</span>
    <button onClick={addTables}>+</button>
    <button onClick={subtractTables}>-</button>
  </>
}
```
## The Kore Class

### State initialization

You can set an initial state in the class definition or pass an initial value on the hook. You should not initialize the state with both methods, but if you do, the initial value on the hook has priority.

Prefer setting the state in the class definition for easier readability.

```tsx
class CountHandler extends Kore<{chairs:number, tables:number, rooms:number}> {
  state = {
    chairs: 0,
    tables : 0,
    rooms : 10
  }

  ...
}

// OR 

function Counter() {
  const [counters] = useVolKore(CountHandler, { chairs: 0, tables : 0, rooms : 10 });

 ...
}

```

**Code you wrote in instanceCreated() method will update the initial state.**


### instanceCreated() function

Optional kore method that is called only once when an instance is created. If exists in the instance, this method is called by the useVolKore or useVolKore( Kore&Partial : Array ) use hook the first time a component in the application using the hook is effectively mounted and when the instance is "newly created".  

This method has NOT the same behavior as mount callback of a component in React. The only way this method is called again by the hook is by destroying the instance first with destroyInstance().

```tsx
class CountHandler extends Kore<{chairs:number, tables:number, rooms:number}> {
  state = {
    chairs: 0,
    tables : 0,
    rooms : 0
  }

  instanceCreated = () => {
    fetch('https://myapi.com/counters').then( r => r.json() ).then( r => this.setState(r) );
  }
}
```



### kore Configuration

You may configure the kore by setting the optional property _handlerConfig in your kore. It has two boolean options:
* merge : The state is overwritten by default using setState. Change this to true to merge.
* destroyOnUnmount : Tries to delete the instance in each unmount of each component. Is successfully deleted if there are no active listeners (other components using it).

```tsx
//default:
 _handlerConfig = { merge : false, destroyOnUnmount : false }
```

#### Merging the state

Overwrite the state is the default mode on hanlder.setState, but you can configure the kore to merge. This can be usefull for refactor old class components.

```tsx
class CountHandler extends Kore<{chairs:number, tables:number, rooms:number}> {
  state = {
    chairs: 0,
    tables : 0,
    rooms : 10
  }

  _handlerConfig = { merge : true }

  addChairs = () => this.setState( c => ( { chairs: c.chairs + 1 }) );
  subtractChairs = () => this.setState( c => ({chairs : c.chairs - 1}) );

  addTables = () => this.setState( t => ({ tables: t.tables + 1 }) );
  subtractTables = () => this.setState( t => ({tables: t.tables - 1}) );

  resetAll = () => this.setState( { chairs: 0, tables : 0 } );
}

function Chairs() {
  const [{chairs},{addChairs, subtractChairs}] = useVolKore(CountHandler);

  return <>
    <span>Chairs: {chairs}</span>
    <button onClick={addChairs}>+</button>
    <button onClick={subtractChairs}>-</button>
  </> 
}

function Tables() {
  const [{tables},{addTables, subtractTables}] = useVolKore(CountHandler);

  return <>
    <span>Tables: {tables}</span>
    <button onClick={addTables}>+</button>
    <button onClick={subtractTables}>-</button>
  </>
}
```
**Note that the useVolKore hook will trigger re-render for any part of the state changed. In the example above, Tables component will re-render if the chairs value is changed. This behavior can be optimized with useVolKore( Kore&Partial : Array ) hook.**  
**Merging mode is only for non-undefined objects, and there is no check of any kind for this before doing it, so its on you to guarantee an initial and always state object.**



### Reutilizing classes

Classes are made for reutilization, making new object instances from these. But in this case, the instance creation is managed by the hook, and it maintains only one instance per class name. 
One way to use your class again with this hook without duplicating code is to extend it:

```ts
class CountHandlerTwo extends CountHandler {};
```

### Extendibility and Inheritance

You can write common functionality in a generic class and extend this class, adding the specifics. In this case, extending a parent generic class with Kore lets you encapsulate common functionality:


MyGenericApiHandler.ts : A kore for my API
```ts
type ApiData = {
  data?: Record<string, any>[];
  isLoading: boolean;
}

export abstract class MyGenericApiHandler extends Kore<ApiData>{

  state : ApiData = {
    data: undefined,
    isLoading: false
  }

  protected _handlerConfig = { merge: true };

  abstract readonly loadUri : string; // making loadUri property obligatory to define in inherited class
  readonly saveUri? : string = undefined;

  public load = ( params? : string ) => {
    this.setState({isLoading: true});
    return fetch( this.loadUri + ( params ?? '' ) )
            .then( r => r.json() )
            .then( resp => this.setState({ data : resp?.data ?? [] , isLoading: false}) )
  }

  public modify = ( item : Record<string, any>, changes : Record<string, any> ) => 
    this.setState( s => ({ data : s.data?.map( i => i === item ? { ...i, ...changes } : i ) }) )

  public delete = ( item : Record<string, any> ) => 
    this.setState( s => ({ data : s.data?.filter( i => i !== item ) )} )

  public append = ( item : Record<string, any> ) =>
    this.setState( s => ({ data : s.data?.concat( item ) }) )

  public save = ( params? : Record<string, any> ) => {
    this.setState({isLoading: true});
    return fetch( this.saveUri ?? '', { method: 'POST', body : JSON.stringify(params)} )
            .then( r => r.json() )
            .then( () => this.setState({ isLoading: false }) )
  }

}
```

MyComponent.tsx

```tsx
import { MyGenericApiHandler } from "./MyApiHandler";

class SpecificApiHandler extends MyGenericApiHandler { loadUri = 'https://myapi/specific' }

export function MyComponent() {

  const [{data, isLoading}, {load, formModify, save} ] = useVolKore( SpecificApiHandler );

  useEffect( () => { load() }, [] );

  return ( ... );
}
```


### Your own setState function

setState() is just a wrapper for the actual _setState() function. You can directly modify it in Javascript; in Typescript, you need to define the setState type as a second generic type of the kore class.

#### Example with immer:
```tsx
import { produce, WritableDraft } from "immer";

type CountState = {chairs:number, tables:number, rooms:number};
type MySetStateType = ( recipe : (draft: WritableDraft<CountState>) => void ) => void;

export class CountHandler extends Kore<CountState, MySetStateType> {
  state = {
    chairs: 0,
    tables : 0,
    rooms : 10
  }

  public setState : MySetStateType = ( recipe ) => this._setState( s => produce(s, recipe) )

}

function Chairs() {
  const [{chairs}, {setState}] = useVolKore(CountHandler);
  return <>
    <span>Chairs: {chairs}</span>
    <button onClick={() => setState( s => { s.chairs++ } )}>+</button>
    <button onClick={() => setState( s => { s.chairs-- } )}>-</button>
  </>
}

function Tables() {
  const [{tables}, {setState}] = useVolKore(CountHandler);
  return <>
    <span>Tables: {tables}</span>
    <button onClick={() => setState( s => { s.tables++ } )}>+</button>
    <button onClick={() => setState( s => { s.tables-- } )}>-</button>
  </>
}
```


#### Or may be you just want to change the setState() accessibility modifier
```tsx
public setState = this._setState
```

### Destroying the instance

You may destroy the instance when needed using the **destroyInstance()** method. This method must be called **on the unmount callback** of the component using it.  
This first checks if there are active state hook listeners active. If there isn't, the instance reference is deleted, and the **instanceDeleted()** method is called if exists.

If you implement **instanceDeleted()**, remember that it is not the equivalent of an unmount component callback.

This is not neccesary if the kore option destroyOnUnmount is true, nor with useKore standalone hook. 

```tsx
export function App() {

  const [ {data}, {load, destroyInstance} ] = useVolKore( ActividadesHandler );

  useEffect( () => {
    load();
    return () => destroyInstance(); // instanceDeleted() would be called
  }, [] );

 ...
}
```

### Constructor

You may define a constructor in your class. But is not necessary

**Prefer defining an instanceCreated() method on the kore over the constructor to execute initial code.** 

```tsx
constructor( initialState? : T ) {
  super(initialState);
  //your code
}
```

Constructor code of the class and its inherited instances constructors are not part of the mounting/unmounting logic of react. Hook state listeners may or may not be ready when the code executes. 

It is safe to write code that does not involve changing the state directly.

