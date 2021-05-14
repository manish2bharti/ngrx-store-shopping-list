# NgrxShoppingList

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 9.1.15.

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files.


## Setting up the Project
Let’s use the Angular CLI to begin a new Angular project:
Ensure you've got the Angular CLI installed
> $ npm i @angular/cli -g
#### Create a new Angular project
```
$ ng new ngrx-shopping-list
 Don't add routing at this stage
 Style: SCSS
 ```
#### Change directory
> $ cd ngrx-shopping-list

You can also use the shortcut ng add @ngrx/store to have the Angular CLI automate some of what we cover in this article.

## Adding the Shopping List
For this project, users are going to use a simple form to add items to a shopping list, therefore, we’ll need a model to represent this. Create the ShoppingItem model at: src/app/store/models/shopping-item.model.ts:
```
 export interface ShoppingItem {
   id?: string;
   name: string;
}
```

## Adding an Action
In @ngrx/store, Actions represent the main building blocks that express the unique events occurring throughout the Angular application. Every user interaction that triggers a state update should be described using Actions.

Actions have two main properties: **type** and an optional **payload**:
* **type** is a read-only string that represents the type of action dispatched into the application.
* **payload** is an optional property that adds any related data required for completing the action.

We can go ahead and add two main actions to our application at src/app/store/actions/shopping.actions.ts:
```
import { Action } from '@ngrx/store';
import { ShoppingItem } from '../models/shopping-item.model';

export enum ShoppingActionTypes {
  ADD_ITEM = '[SHOPPING] Add Item',
}

export class AddItemAction implements Action {
  readonly type = ShoppingActionTypes.ADD_ITEM

  constructor(public payload: ShoppingItem) { }
}

export type ShoppingAction = AddItemAction
```
We're creating an **enum** which contains the **ADD_ITEM** Action **type**. This can then be used inside of the **AddItemAction** class which contains a **payload** (i.e. a new ShoppingItem).


## Adding a Reducer
Reducers are the second essential aspect of an @ngrx/store application. They play the critical role of managing the transitions taking place from *one state to the next*.

**As pure functions, reducers generate the same output for a given input.** Reducers accept two arguments, which are the previous state as well as the latest dispatched **Action**, and either returns a newly changed state or the unchanged state.

To add a **reducer** to our project, let’s create this folder and file: /src/app/store/reducers/shopping.reducer.ts.

```
import { ShoppingActionTypes, ShoppingAction } from '../actions/shopping.actions';
import { ShoppingItem } from '../models/shopping-item.model';

const initialState: Array<ShoppingItem> = [
  {
    id: "1775935f-36fd-467e-a667-09f95b917f6d",
    name: 'Diet Coke',
  }
];

export function ShoppingReducer(state: Array<ShoppingItem> = initialState, action: ShoppingAction) {
  switch (action.type) {
    case ShoppingActionTypes.ADD_ITEM:
      return [...state, action.payload];
    default:
      return state;
  }
}
```

The reducer takes in the initial state which contains our **Diet Coke** and a dispatched **action**. It then checks which **action.type** has been fired and subsequently reduces this to a value that represents the same structure as before but with different data.

For example, whenever we call the **ADD_ITEM** action, the reducer takes the previous state and appends the **action.payload** (i.e. the new Shopping item) to the end of the list.

This then gives us a new version of state that matches the previous structure or in the case of no **action.type** being found inside of our **shoppingReducer**, it simply returns the **state**.

##AppState
Next, we can create an interface for our AppState inside of 
src/store/models/app-state.model.ts.
```
import { ShoppingItem } from './shopping-item.model';

export interface AppState {
  readonly shopping: Array<ShoppingItem>
}
```
This will allow us to type the structure of our Store and make it easier to select slices of state in the future. We'll be using this in a moment, until then, we need to register our **shopping** reducer as a **root** reducer.

##Adding imports to App.Module
Let’s update the /src/app/app.module.ts file with imports of @ngrx/store and the reducer we created earlier. We'll also be using ngModel later to capture user input, ensure you've imported FormsModule too.

```
import { FormsModule } from '@angular/forms';
import { StoreModule } from '@ngrx/store'; 

import { ShoppingReducer } from './reducers/shopping.reducer';

@NgModule({
 imports: [
   BrowserModule,
   FormsModule,
   StoreModule.forRoot({
     shopping: ShoppingReducer
   })
 ],
```

#Adding Reading, Writing, and Deleting Capabilities
Here's the fun part. We can wire the store up to our UI and see our shopping list in action!

##Selecting state
In order to display **shopping** items, we'll need to **select** the **shopping** slice of state from our store. @ngrx makes this easy with a **select** method which provides us with a reactive Observable:
```
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';

import { ShoppingItem } from './store/models/shopping-item.model';
import { AppState } from './store/models/app-state.model';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent implements OnInit {
  
  shoppingItems: Observable<Array<ShoppingItem>>;

  constructor(private store: Store<AppState>) { }

  ngOnInit() {
    this.shoppingItems = this.store.select(store => store.shopping);
  }
}
```

Our **shoppingItems** observable matches the **shopping** reducer that we defined in our **StoreModule.forRoot()** earlier. We also created the **AppState** interface to match this and give us strong typing.

As a note, we could have also selected our **shopping** slice by string, providing it matched the reducer definition:

```
this.shoppingItems = this.store.select('shopping');

```
We can then display this on screen with the following mark-up:


```
<div id="wrapper">
  
  <div id="shopping-list">
    <div id="list">
      <h2>Shopping List</h2>

      <ul>
        <li *ngFor="let shopping of shoppingItems | async">
          {{ shopping.name }}
        </li>
      </ul>
    </div>
  </div>
  
</div>
```

The most important consideration here is the automatic subscription to our shoppingItems observable with the async pipe. Angular will then handle the subscription/unsubscription events for us automatically.

##Adding new items
In order to add a new item to our store, we'll need to **dispatch** an **AddItemAction** with the payload of a new **ShoppingItem**. Let's take a look at how we can do that:

```
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { v4 as uuid } from 'uuid';

import { AppState } from './store/models/app-state.model';
import { ShoppingItem } from './store/models/shopping-item.model';
import { AddItemAction } from './store/actions/shopping.actions';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent implements OnInit {
  
  shoppingItems: Observable<Array<ShoppingItem>>;
  newShoppingItem: ShoppingItem = { id: '', name: '' }

  constructor(private store: Store<AppState>) { }

  ngOnInit() {
    this.shoppingItems = this.store.select(store => store.shopping);
  }

  addItem() {
    this.newShoppingItem.id = uuid();

    this.store.dispatch(new AddItemAction(this.newShoppingItem));

    this.newShoppingItem = { id: '', name: '' };
  }
}
```
We're able to generate a unique id with the uuid library. We don't have to install this into our project as it comes by default as a dependency to angular. At this point we'll also need to update our app.component.html to add our form:

```
<div id="wrapper">
  <div id="shopping-list">
    <div id="list">
      <h2>Shopping List</h2>

      <ul>
        <li *ngFor="let shopping of shoppingItems | async">
          {{ shopping.name }}
          <br><br>
        </li>
      </ul>
    </div>

    <form (ngSubmit)="addItem()">
      <input type="text" [(ngModel)]="newShoppingItem.name" placeholder="Item" name="itemName"/>
      <button type="submit" >+</button>
    </form>
  </div>
</div>
```

Here's what happens once we hit the + button:
1. The **AddItemAction** is fired with the payload of **newShoppingItem**.
2. Our **shoppingReducer** sees the new **action** and filters by **action.type**.
3. As the action.type is **[SHOPPING] Add Item** the **newShoppingItem** is added to the end of our array: **[...state, action.payload]**,
4. The **shopping** slice of state is updated and as we're subscribed to changes with the **async** pipe our UI updates.

## Deleting items
If you've come this far, great job! Here's a challenge for you:

Add the ability to delete a selected item by **id**

When we start thinking about how we can add another feature to our project using the **@ngrx** pattern, things start to become systematic. We need the following:

1. An Action
2. A reducer case and statement

To start with, update the shopping.actions.ts with a new DELETE action:

```
import { Action } from '@ngrx/store';
import { ShoppingItem } from '../models/shopping-item.model';

export enum ShoppingActionTypes {
  ADD_ITEM = '[SHOPPING] Add Item',
  DELETE_ITEM = '[SHOPPING] Delete Item'
}

export class AddItemAction implements Action {
  readonly type = ShoppingActionTypes.ADD_ITEM

  constructor(public payload: ShoppingItem) { }
}

export class DeleteItemAction implements Action {
  readonly type = ShoppingActionTypes.DELETE_ITEM

  constructor(public payload: string) { }
}

export type ShoppingAction = AddItemAction | DeleteItemAction
```

We've added an item to the **ShoppingActionTypes** and a new **DeleteItemAction**. As well as that, we've updated the **ShoppingAction** type to be either an **AddItemAction** or **DeleteItemAction**.

Next up, we need to add a reducer **case** which uses **filter** to return a **new** array that doesn't contain the selected shopping item.

It's important that you don't mutate the state itself and **always** return a new array. For example, never use **Array.splice** to mutate the state itself.

```
export function ShoppingReducer(state: Array<ShoppingItem> = initialState, action: ShoppingAction) {
  switch (action.type) {
    case ShoppingActionTypes.ADD_ITEM:
      return [...state, action.payload];
    case ShoppingActionTypes.DELETE_ITEM:
      return state.filter(item => item.id !== action.payload);
    default:
      return state;
  }
}
```

Now we'll need to update our app.component.ts and app.component.html to include the ability to dispatch a **DeleteItemAction** based on a selected id.

Starting with the HTML file, update your li to include a click event that calls a **deleteItem** function:
```
<li *ngFor="let shopping of shoppingItems | async" (click)="deleteItem(shopping.id)">
  {{ shopping.name }}
</li>
```
Finally, update app.component.ts with the deleteItem function which uses the specified id:
```
import { AddItemAction, DeleteItemAction } from './store/actions/shopping.actions';

export class AppComponent implements OnInit {

  // Omitted

  deleteItem(id: string) {
    this.store.dispatch(new DeleteItemAction(id));
  }
}
```

##Summary
In this article we firstly looked at how to add @ngrx/store to a new project. We then made a small shopping list style application that used the power of @ngrx to manage the state of our list.
