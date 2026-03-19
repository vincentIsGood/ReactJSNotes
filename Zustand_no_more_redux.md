# Zustand for the win
Dump that Redux and start Zustand

```ts
npm install zustand
```

```ts
// store.js
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useSettingsStore = create(
  persist(
    (set) => ({
      // 1. Your initial state (the 10 settings)
      theme: 'dark',
      language: 'en',
      notificationsEnabled: true,
      fontSize: 'medium',
      // ... add the rest here

      // 2. The "Actions" (directly inside the store!)
      updateSetting: (key, value) => 
        set((state) => ({ [key]: value })),

      resetSettings: () => 
        set({ theme: 'dark', language: 'en', notificationsEnabled: true, ... }),
    }),
    {
      name: 'user-settings-storage', // Unique name for LocalStorage
    }
  )
);

// SettingsComponent.jsx
import { useSettingsStore } from './store';

const SettingsComponent = () => {
  // Select specific values (still best practice for performance!)
  const theme = useSettingsStore((state) => state.theme);                  // value
  const updateSetting = useSettingsStore((state) => state.updateSetting);  // function for update

  return (
    <div className={`app-${theme}`}>
      <select 
        value={theme} 
        onChange={(e) => updateSetting('theme', e.target.value)}
      >
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
    </div>
  );
};
```