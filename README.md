# React Developer Assessment

## Section 1: Core React Concepts

### Question 1.1

Explain the difference between `useState` and `useRef`. When would you use each?

`useState` stores reactive state and causes the component to re-render when updated, so I would use it to store data that affects the UI.
`useRef` stores a mutable value that persists across renders but does not trigger re-renders, so I would use it for DOM references or storing values like timers or previous state.

### Question 1.2

Fix the following component that has multiple issues:

```jsx
import React from 'react';

function UserList({ users }) {
  const [selectedUser, setSelectedUser] = useState();

  return (
    <div>
      <h2>Users</h2>
      {users.map((user) => (
        <div onClick={() => setSelectedUser(user.id)}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}

      {selectedUser && (
        <div>
          <h3>Selected: {selectedUser.name}</h3>
        </div>
      )}
    </div>
  );
}
```

Issues:
1. `useState` is not imported
2. keys should be added in users.map()
3. To set a clear initial value, `useState(null)` should be used instead of `useState()` which sets an undefined state
4. `setSelectedUser` originally stores `user.id` but later `selectedUser.name` is being read => `setSelectedUser(user)`

Fixed component:

```jsx
import React from 'react';
import { useState } from 'react';

function UserList({ users }) {
  const [selectedUser, setSelectedUser] = useState(null);

  return (
    <div>
      <h2>Users</h2>
      {users.map((user) => (
        <div key={user.id} onClick={() => setSelectedUser(user)}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}

      {selectedUser && (
        <div>
          <h3>Selected: {selectedUser.name}</h3>
        </div>
      )}
    </div>
  );
}
```

### Question 1.3

Create a custom hook called `useToggle` that:

- Takes an initial boolean value as parameter
- Returns the current state and a function to toggle it
- Include a reset function that sets it back to the initial value

---

```jsx
import { useState } from "react";

function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = () => {
    setValue((prev) => !prev);
  };

  const reset = () => {
    setValue(initialValue);
  };

  return [value, toggle, reset];
}

export default useToggle;
```

## Section 2: State Management & Effects

Create a `UserProfile` component that:

- Fetches user data from an API endpoint on mount
- Shows loading state while fetching
- Handles and displays errors appropriately
- Allows editing user name with optimistic updates
- Cancels the request if component unmounts before completion

**API Endpoint:** `https://jsonplaceholder.typicode.com/users/1`

---

## Section 3: Component Design & Props

Design and implement a flexible `Modal` component with these features:

- Open/close based on a prop
- Accept custom content as children
- Has a backdrop that close the modal when clicked
- Prevent body scroll when open
- Support custom close button
- Accessible (keyboard navigation, focus management)

Provide usage examples showing different ways to use your Modal component.

---

## Section 4: Code Review

Review the following component and provide feedback:

```jsx
function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [filter, setFilter] = useState('');

  useEffect(() => {
    fetch('/api/products')
      .then((res) => res.json())
      .then((data) => {
        setProducts(data);
        setLoading(false);
      });
  }, []);

  const filteredProducts = products.filter((p) => p.name.toLowerCase().includes(filter.toLowerCase()));

  return (
    <div>
      <input value={filter} onChange={(e) => setFilter(e.target.value)} placeholder="Search products..." />

      {loading ? (
        <div>Loading...</div>
      ) : (
        <div>
          {filteredProducts.map((product) => (
            <div style={{ border: '1px solid #ccc', margin: '10px', padding: '10px' }}>
              <h3>{product.name}</h3>
              <p>${product.price}</p>
              <img src={product.image} style={{ width: '100px' }} />
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

## Submission Guidelines

1. Provide all code solutions in a single file or organized folder structure
2. Include comments explaining your reasoning for complex solutions
3. Push your code to the Github public repo and send us the link via email
