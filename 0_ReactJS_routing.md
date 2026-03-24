# Adding client-side routing in ReactJS

```sh
npm install react-router-dom
```

## Example
```ts
// App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter> // Need this
      <div className="app-container">
        <Routes>
          <Route path="/" element={<Dashboard />} />
          <Route path="/news/:ticker" element={<NewsPage />} />
        </Routes>
      </div>
    </BrowserRouter>
  );
}

export default App;

// Navigate
import { useParams, useNavigate } from 'react-router-dom';

function ExampleCard(){
    // Get ticker from url path: `/news/:ticker`
    const { ticker } = useParams<{ ticker: string }>();
    const navigate = useNavigate();
    return (
        <button className="back-button" onClick={() => navigate('/')}>
    )
} 
```