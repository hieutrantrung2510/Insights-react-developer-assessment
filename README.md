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

```jsx
import { useEffect, useState } from "react";


const USER_URL = "https://jsonplaceholder.typicode.com/users/1";

type Geo = {
    lat: string;
    lng: string;
};

type Address = {
    street: string;
    suite: string;
    city: string;
    zipcode: string;
    geo: Geo;
};

type Company = {
    name: string;
    catchPhrase: string;
    bs: string;
};

type User = {
    id: number;
    name: string;
    username: string;
    email: string;
    address: Address;
    phone: string;
    website: string;
    company: Company;
};

function UserProfile() {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState<boolean>(true);
    const [error, setError] = useState<string>("");

    const [nameInput, setNameInput] = useState("");
    const [saving, setSaving] = useState(false);
    const [saveError, setSaveError] = useState("");

    const handleSave = async () => {
        if (!user) return;
        if (!nameInput) {
            setSaveError("Name can not be empty");
            return;
        }

        setSaving(true);
        setSaveError("");

        const previousUser = user;

        setUser({ ...user, name: nameInput });

        try {
            const response = await fetch(USER_URL, {
                method: "PATCH",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({ name: nameInput }),
            });

            if (!response.ok) {
                throw new Error("Failed to update user");
            }

        } catch (err) {

            // rollback
            setUser(previousUser);
            setNameInput(previousUser.name);

            setSaveError(
                err instanceof Error ? err.message : "Failed to update"
            );

        } finally {
            setSaving(false);
        }
    }

    useEffect(() => {
        const controller = new AbortController();

        const fetchUser = async () => {
            try {
                const response = await fetch(USER_URL, {
                    signal: controller.signal
                });

                if (!response.ok) {
                    throw new Error("Failed to fetch user");
                }

                const data: User = await response.json();
                setUser(data);
                setNameInput(data.name);
            }
            catch (err) {
                if (err instanceof DOMException && err.name === "AbortError") {
                    return;
                }
                setError(err instanceof Error ? err.message : "Something went wrong");
            }
            finally {
                setLoading(false);
            }
        };

        fetchUser();

        return () => {
            controller.abort();
        };
    }, []);

    return (
        <div>
            {loading && <p>Loading...</p>}
            {error && <p>{error}</p>}
            {user &&
                <div>
                    <p>{user.name}</p>
                    <input
                        value={nameInput}
                        onChange={(e) => setNameInput(e.target.value)}
                        disabled={saving}
                    />

                    <button onClick={handleSave} disabled={saving}>
                        {saving ? "Saving..." : "Save"}
                    </button>

                    {saveError && <p>{saveError}</p>}
                </div>
            }
        </div>
    );
}

export default UserProfile;
```

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

1. Modal.tsx

```jsx
import React, { useEffect, useRef } from "react";

type ModalProps = {
    isOpen: boolean;
    onClose: () => void;
    children: React.ReactNode;
    closeButton?: React.ReactNode;
};

const FOCUSABLE_SELECTORS = [
    'a[href]',
    'button:not([disabled])',
    'textarea:not([disabled])',
    'input:not([disabled])',
    'select:not([disabled])',
    '[tabindex]:not([tabindex="-1"])',
].join(",");

function getFocusableElements(container: HTMLElement): HTMLElement[] {
    return Array.from(
        container.querySelectorAll<HTMLElement>(FOCUSABLE_SELECTORS)
    );
}

function Modal({ isOpen, onClose, children, closeButton }: ModalProps) {
    const modalRef = useRef<HTMLDivElement | null>(null);
    const previousFocusedElementRef = useRef<HTMLElement | null>(null);

    useEffect(() => {
        if (!isOpen) return;

        const originalOverflow = document.body.style.overflow;
        document.body.style.overflow = "hidden";

        return () => {
            document.body.style.overflow = originalOverflow;
        };
    }, [isOpen]);

    useEffect(() => {
        if (!isOpen) return;

        previousFocusedElementRef.current =
            document.activeElement as HTMLElement | null;

        modalRef.current?.focus();

        return () => {
            previousFocusedElementRef.current?.focus();
        };
    }, [isOpen]);

    useEffect(() => {
        if (!isOpen) return;

        const handleKeyDown = (event: KeyboardEvent) => {
            if (event.key === "Escape") {
                onClose();
                return;
            }

            if (event.key !== "Tab") return;

            const modalElement = modalRef.current;
            if (!modalElement) return;

            const focusableElements = getFocusableElements(modalElement);

            if (focusableElements.length === 0) {
                event.preventDefault();
                modalElement.focus();
                return;
            }

            const firstElement = focusableElements[0];
            const lastElement = focusableElements[focusableElements.length - 1];
            const activeElement = document.activeElement as HTMLElement | null;

            if (event.shiftKey) {
                if (activeElement === firstElement) {
                    event.preventDefault();
                    lastElement.focus();
                }
            } else {
                if (activeElement === lastElement) {
                    event.preventDefault();
                    firstElement.focus();
                }
            }
        };

        document.addEventListener("keydown", handleKeyDown);

        return () => {
            document.removeEventListener("keydown", handleKeyDown);
        };
    }, [isOpen, onClose]);

    if (!isOpen) return null;

    return (
        <div
            style={styles.backdrop}
            onClick={(e) => {
                if (e.target === e.currentTarget) {
                    onClose();
                }
            }}
        >
            <div
                ref={modalRef}
                style={styles.modal}
                role="dialog"
                aria-modal="true"
                tabIndex={-1}
            >
                <div style={styles.header}>
                    {closeButton ? (
                        <div style={styles.customCloseWrapper} onClick={onClose}>
                            {closeButton}
                        </div>
                    ) : (
                        <button
                            style={styles.defaultCloseButton}
                            onClick={onClose}
                            aria-label="Close modal"
                        >
                            ×
                        </button>
                    )}
                </div>

                <div>{children}</div>
            </div>
        </div>
    );
}

const styles: Record<string, React.CSSProperties> = {
    backdrop: {
        position: "fixed",
        inset: 0,
        background: "rgba(0,0,0,0.5)",
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        padding: "24px",
    },
    modal: {
        background: "white",
        padding: "20px",
        borderRadius: "8px",
        minWidth: "320px",
        maxWidth: "600px",
        width: "100%",
        outline: "none",
    },
    header: {
        display: "flex",
        justifyContent: "flex-end",
        marginBottom: "12px",
    },
    defaultCloseButton: {
        border: "none",
        background: "transparent",
        fontSize: "24px",
        cursor: "pointer",
    },
    customCloseWrapper: {
        cursor: "pointer",
        display: "inline-flex",
    },
};

export default Modal;
```

2. App.tsx (Usage example)

```jsx
import { useState } from "react";
import Modal from "./Modal";

function App() {
  const [isOpen, setIsOpen] = useState(false);

  const handleDelete = () => {
    alert("Item deleted");
    setIsOpen(false);
  };

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>
        Delete Item
      </button>

      <Modal
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
      >
        <h3>Confirm Delete</h3>

        <p>Are you sure you want to delete this item?</p>

        <button
          onClick={handleDelete}
          style={{ background: "red", color: "white" }}
        >
          Delete
        </button>

        <button
          onClick={() => setIsOpen(false)}
          style={{ marginLeft: 10 }}
        >
          Cancel
        </button>
      </Modal>
    </div>
  );
}

export default App;
```
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

In my opinion, this code only has missing keys and missing error handling issues
