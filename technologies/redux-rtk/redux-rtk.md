# Redux & Redux Toolkit (RTK)

#technology

Redux is a predictable state management library for JavaScript apps, commonly used with React. 

Redux Toolkit (RTK) is the official, recommended approach to write Redux logic, providing simplified APIs and best practices out of the box.

---

## Resources

- **Deep Dives**  
	- [Redux Fundamentals Tutorial](https://redux.js.org/tutorials/fundamentals/part-1-overview)  
	- [RTK Usage Guide](https://redux-toolkit.js.org/introduction/getting-started)  

- **Docs & References**  
	- [Redux Docs](https://redux.js.org/)  
	- [Redux Toolkit Docs](https://redux-toolkit.js.org/)  

---

## Core Concepts

- **Single Source of Truth**: The entire app state is stored in a single Redux store object.
- **Reducers**: Pure functions that specify how state changes in response to actions.
- **Actions**: Plain objects describing what happened, dispatched to trigger state changes.
- **Dispatch**: Method to send actions to the store.
- **Selectors**: Functions to read and derive state from the store.
- **Redux Toolkit (RTK)**:  
  - Provides `configureStore()` for easy store setup with good defaults.  
  - Includes `createSlice()` to define reducers and actions together.  
  - Supports `createAsyncThunk()` for handling async logic like API calls.  
  - Uses Immer internally to let you write “mutating” immutable update logic.  
- **Middleware**: Enhances dispatching process (e.g., thunk for async, logger for debugging).
- **DevTools Integration**: Time-travel debugging and state inspection with Redux DevTools.

---
