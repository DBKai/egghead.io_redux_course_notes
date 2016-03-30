# 21. Извлечение Презентационных Компонентов (AddTodo, Footer, FilterLink)

Мы отрефакторили `TodoApp` используя компонент `TodoList`.

We've refactored `TodoApp` to use a `TodoList` component.

Давайте продолжим работать над разделением представления от поведения.

Let's keep working on separating the looks from the behavior.


#### Current `TodoApp` Code:
```JavaScript
class TodoApp extends Component {
  render () {
    const {
      todos,
      visibilityFilter
    } = this.props;

    const visibleTodos = getVisibleTodos(
      todos,
      visibilityFilter
    );

    return (
      <div>
        <input ref={node => {
          this.input = node;
        }} />
        <button onClick={() => {
          store.dispatch({
            type: 'ADD_TODO',
            text: this.input.value,
            id: nextTodoId++
          });
          this.input.value = '';
        }}>
          Add Todo
        </button>
        <TodoList 
          todos={visibleTodos}
          onTodoClick={id =>
            store.dispatch({
              type: 'TOGGLE_TODO',
              id
            })
          } />

        <p>
          Show:
          {' '}
          <FilterLink
            filter="SHOW_ALL"
            currentFilter={visibilityFilter}
          >
            All
          </FilterLink>
          {', '}
          <FilterLink
            filter='SHOW_ACTIVE'
            currentFilter={visibilityFilter}
          >
            Active
          </FilterLink>
          {', '}
          <FilterLink
            filter='SHOW_COMPLETED'
            currentFilter={visibilityFilter}
          >
            Completed
          </FilterLink>
        </p>
      </div>
    );
  }
}
```

#### Извлечение Input и Button из `AddTodo`

Мы будем объединять input и button в один новый компонент под названием `AddTodo`.

We will combine the input and the button into one new component called `AddTodo`.

Функциональные компоненты не имеют экземпляров, поэтому, вместо того, чтобы использовать `this`, мы будем использовать переменную,
под названием `input` что мы будем закрывать над, так что мы можем написать к ней внутри функции.

Functional components don't have instances, so instead of using `this`, we will use a variable called `input` that we will
close over so we can write to it inside of the function.

Так как мы хотим, чтобы `AddTodo` был презентационным компонентом, мы заведем кнопку с функцией `onAddClick()`
с параметром `input` значения в качестве параметра. Мы так же сделаем свойство `onAddClick` так, чтобы компонент,
который использует `AddTodo`, мог указать, что происходит, когда кнопка "Add Todo" нажата.

Since we want `AddTodo` to be a presentational component, we will have the button call an `onAddClick()` function
with `input`'s value as its parameter. We also make `onAddClick` a prop so that the component that uses `AddTodo`
can specify what happens with the "Add Todo" button is clicked.

```JavaScript
const AddTodo = ({
  onAddClick
}) => {
  let input;

  return (
    <div>
      <input ref={node => {
        input = node;
      }} />
      <button onClick={() => {
        onAddClick(input.value);
        input.value = '';
      }}>
        Add Todo
      </button>
    </div>
  );
};
```

Теперь нам нужно обновить компонент контейнера `TodoApp` заменой `<input>` и `<button>` вхождений
с нашим новым компонентом `AddTodo`.

Now we need to update the `TodoApp` container component by replacing the `<input>` and `<button>` entries with 
our new `AddTodo` component.

Мы будем так же уточнять нашу функцию `onAddClick` отправкой события с типом `'ADD_TODO'` с соответствующим `text` и следующим `id`.

We will also specify our `onAddClick` function to dispatch an action of type `'ADD_TODO'` along with the corresponding `text` and next `id`.

```JavaScript

// inside `TodoApp`'s `render` method

return (
  <div>
    <AddTodo
      onAddClick={text =>
        store.dispatch({
          type: 'ADD_TODO',
          id: nextTodoId++,
          text  
        })
      }
    />

```

#### Извлечение `FilterLink` Элементов Footer
Сейчас мы будем создавать новый функциональный компонент под названием `Footer`. Поскольку каждый `FilterLink`
должен знать `visibilityFilter`, мы сделаем это свойство.

Now we will create a new functional component called `Footer`. Since each `FilterLink` needs to know the
`visibilityFilter`, we will make that a prop.

Мы хотим сделать `Filter` и `FilterLink` презентационными компонентами, но в этом текущем представлении каждый
содержащийся `FilterLink` вызывает `store.dispatch()`. Этот вызов будет заменён вызовом `onClick` что будет
принимать один параметр с фильтром. Мы так же добавим `onClick` свойство `FilterLink`.

We want the `Filter` and `FilterLink` to be presentational components, but in its current implementation each
of the `FilterLink`s contain a `store.dispatch()` call. This call will be replaced by an `onClick` call that
will take a single parameter with the filter. We also add `onClick` to `FilterLink`'s props.

Поскольку `onClick` теперь свойство для `FilterLink`, нам нужно указать это каждый раз, когда `FilterLink`
используется в нашем `Footer`

Since `onClick` is now a prop for `FilterLink`, we need to specify it every time that `FilterLink` is used in our `Footer`.
Adding `onClick={onFilterClick}` makes sure that `onClick` makes it to `FilterLink` as a prop.

```JavaScript
// FilterLink был построен в предыдущем разделе
const FilterLink = ({
  filter,
  currentFilter,
  children,
  onClick
}) => {
  if (filter === currentFilter) {
    return <span>{children}</span>
  }

  return (
    <a href='#'
      onClick={e => {
        e.preventDefault();
        onClick(filter);
      }}
    >
      {children}
    </a>
  );
};

const Footer = ({
  visibilityFilter,
  onFilterClick
}) => (
  <p>
    <FilterLink
      filter='SHOW_ALL'
      currentFilter={visibilityFilter}
      onClick={onFilterClick}
    >
      All
    </FilterLink>
    {', '}
    <FilterLink
      filter='SHOW_ACTIVE'
      currentFilter={visibilityFilter}
      onClick={onFilterClick}
    >
      Active
    </FilterLink>
      {', '}
    <FilterLink
      filter='SHOW_COMPLETED'
      currentFilter={visibilityFilter}
      onClick={onFilterClick}
    >
      Completed
    </FilterLink>
  </p>
);
```

#### Добавление `Footer` к `TodoApp`
При добавлении компонента `Footer` в `TodoApp`, нам нужно проверить два свойства. Сначала, `visibilityFilter`
подсветка активной ссылки. Второе свойство `onFilterClick`, которое будет отправлять действие с типом
`'SET_VISIBILITY_FILTER'`  вместе с `filter` будучи нажатым.

When adding the `Footer` component into `TodoApp`, we need to pass two props. First, `visibilityFilter` to highlight
the active link. The second prop is `onFilterClick`, which will dispatch an action of type `'SET_VISIBILITY_FILTER'`
along with the `filter` being clicked.

```JavaScript

// inside `TodoApp`'s `render` method

return (
  <div>
    // `<AddTodo>` component
    // `<TodoList>` component
    <Footer 
      visibilityFilter={visibilityFilter}
      onFilterClick={filter =>
        store.dispatch({
          type: 'SET_VISIBILITY_FILTER'
          filter
        })
      }
    />
  </div>

```

#### Изменение `TodoApp` в функцию
Существует возможномть изменить `TodoApp` из класса в функцию.

It is possible to change `TodoApp` from a class into a function. 

Это позволяет устранить деструктуризацию `todos` и `visibilityFilter` из `this.props` внутри функции`render`.
Вместо этого, мы можем сделать это внутри аргумента в функции `TodoApp`.

This allows us to eliminate the destructuring of `todos` and `visibilityFilter` from `this.props` inside the `render`
function. Instead, we can do this inside the argument to the `TodoApp` function.

Мы также можем покончить с объявлением `render()`.

We can also do away with the `render()` declaration.

Поскольку `visibleTodos` используется только в одном месте, мы можем переместить это объявление в объявление
свойств `TodoList` `todos`.

Since the `visibleTodos` are only used in a single place, we can move its declaration into the `TodoList` `todos`
prop declaration.

```JavaScript
const TodoApp = ({
  todos,
  visibilityFilter
}) =>
  return (
    <div>
      <AddTodo
        onAddClick={text =>
          store.dispatch({
            type: 'ADD_TODO',
            id: nextTodoId++,
            text
          })
        }
      />
      <TodoList
        todos={
          getVisibleTodos(
            todos,
            visibilityFilter
          )
        }
        onTodoClick={id =>
          store.dispatch({
            type: 'TOGGLE_TODO',
            id
          })
        }
      />
      <Footer 
        visibilityFilter={visibilityFilter}
        onFilterClick={filter =>
          store.dispatch({
            type: 'SET_VISIBILITY_FILTER'
            filter
          })
        }
      />
    </div>
  );
```

#### Recap of the Data Flow
We've now finished the initial refactor of our application into a single container component with many presentational
components inside of it.

[At 4:10 in the video, Dan walks us through the current code & data flow in the application.](https://egghead.io/lessons/javascript-redux-extracting-presentational-components-addtodo-footer-filterlink)

Separation of presentational components isn't required in Redux, but it's a good pattern to follow because it
decouples our rendering from Redux. This way, if we choose to move to another framework like Relay, we can keep our
presentation components as-is.

One of the downsides to having separate presentational components is that we have to move around a lot of props,
including the callbacks. However, we can easily solve this problem by introducing many intermediate container components,
which we will start on in the next section.




