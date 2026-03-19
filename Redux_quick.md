# Redux
Managing a global state with 10 (or even 100) user settings is exactly what Redux was built to handle.

```ts
import { createSlice, configureStore } from '@reduxjs/toolkit';

// Your global context
const initialState = {
  theme: 'dark',
  language: 'en',
  notificationsEnabled: true,
  autoSave: false,
  timezone: 'UTC',
  fontSize: 'medium',
  dataSaverMode: false,
  // ... and so on
};

// A "slice" contains your initial state, and the reducers (functions) that dictate how that state changes.
const userSettingsSlice = createSlice({
  name: 'userSettings',
  initialState,
  reducers: {
    // A single, dynamic reducer to update ANY setting
    updateSetting: (state, action) => {
      const { key, value } = action.payload;
      // Because RTK uses Immer under the hood, we can directly mutate the state
      state[key] = value;
    },
    // Optional: A bulk update if fetching from an API
    setAllSettings: (state, action) => {
      return { ...state, ...action.payload };
    }
  },
});


import userSettingsReducer from './userSettingsSlice';

export const store = configureStore({
  reducer: {
    userSettings: userSettingsSlice.reducer,
    // other reducers go here
  },
});

export const { updateSetting, setAllSettings } = userSettingsSlice.actions;




function Main(){
    // Provider type from react-redux
    return (
        <Provider store={store}>
            <App />
        </Provider>
    )
}
```


```ts
// SettingsPanel.jsx
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { updateSetting } from './userSettingsSlice';

const SettingsPanel = () => {
  const dispatch = useDispatch();
  
  // Good: Select ONLY what this component needs to render
  const theme = useSelector((state) => state.userSettings.theme);
  const notificationsEnabled = useSelector((state) => state.userSettings.notificationsEnabled);

  /*
  When you use useSelector, Redux runs that selector function every time an action is dispatched. 
  It then compares the new result of your selector to the old result using strict equality (===). 
  If the result is different, it forces your component to re-render.

  Summary: useSelector uses "===" to see discrepency then re-render

  Even if you are not modifying the props you are using (let's say "language")
  ===> `oldUserSettings !== newUserSettings`

  Hence, don't useSelector on object.
  */
  // Bad: const settings = useSelector((state) => state.userSettings); 

  const handleThemeChange = (e) => {
    dispatch(updateSetting({ key: 'theme', value: e.target.value }));
  };

  const handleToggleNotifications = () => {
    dispatch(updateSetting({ key: 'notificationsEnabled', value: !notificationsEnabled }));
  };

  return (
    <div>
      <h3>User Preferences</h3>
      
      <label>
        Theme:
        <select value={theme} onChange={handleThemeChange}>
          <option value="light">Light</option>
          <option value="dark">Dark</option>
        </select>
      </label>

      <label>
        <input 
          type="checkbox" 
          checked={notificationsEnabled} 
          onChange={handleToggleNotifications} 
        />
        Enable Notifications
      </label>
    </div>
  );
};

export default SettingsPanel;
```

## With localStorage
```ts
// store.js
import { configureStore } from '@reduxjs/toolkit';
import userSettingsReducer from './userSettingsSlice';

// 1. Function to load state from LocalStorage
const loadFromLocalStorage = () => {
  try {
    const serializedState = localStorage.getItem('userSettings');
    if (serializedState === null) return undefined;
    return JSON.parse(serializedState);
  } catch (e) {
    console.warn(e);
    return undefined;
  }
};

// 2. Configure the store, passing in the preloaded state
export const store = configureStore({
  reducer: {
    userSettings: userSettingsReducer,
  },
  preloadedState: {
    userSettings: loadFromLocalStorage(), // Inject saved data on startup
  }
});

// 3. Subscribe to the store to save changes
store.subscribe(() => {
  try {
    // Only save the userSettings slice
    const settingsState = store.getState().userSettings;
    localStorage.setItem('userSettings', JSON.stringify(settingsState));
  } catch (e) {
    console.warn(e);
  }
});
```