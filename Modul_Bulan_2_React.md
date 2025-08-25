# MODUL PEMBELAJARAN BULAN 2
## React Fundamentals (Frontend Development)

### TUJUAN PEMBELAJARAN
Setelah menyelesaikan modul ini, peserta akan mampu:
- Memahami konsep dasar React dan komponen-komponennya
- Membuat aplikasi frontend modern dengan React
- Mengelola state dan lifecycle dalam aplikasi React
- Mengintegrasikan React dengan API backend

---

## MINGGU 5

### Hari 1: Intro React, JSX & Component Dasar

#### Materi Pembelajaran:
1. **Pengenalan React**
   - Apa itu React dan Virtual DOM
   - Keuntungan menggunakan React
   - Perbandingan dengan framework lain (Vue, Angular)

2. **JSX (JavaScript XML)**
   - Sintaks JSX dan aturan penulisan
   - Embedding expressions dalam JSX
   - JSX vs HTML biasa

3. **Component Dasar**
   - Functional Components
   - Class Components (pengenalan)
   - Component composition

#### Praktik:
```jsx
// App.js
function App() {
  const name = "Penm Academy";
  const currentYear = new Date().getFullYear();
  
  return (
    <div className="App">
      <h1>Selamat datang di {name}</h1>
      <p>Tahun: {currentYear}</p>
    </div>
  );
}

// Welcome.js
function Welcome({ name, age }) {
  return (
    <div>
      <h2>Halo, {name}!</h2>
      <p>Umur: {age} tahun</p>
    </div>
  );
}
```

#### Setup Project:
```bash
# Buat project React baru
npx create-react-app my-react-app
cd my-react-app
npm start
```

#### Tugas:
- Setup project React baru
- Buat komponen Welcome dengan props
- Eksplorasi struktur folder React

### Hari 2: Props & State Management Dasar

#### Materi Pembelajaran:
1. **Props (Properties)**
   - Passing data ke components
   - Props destructuring
   - Default props
   - Props validation dengan PropTypes

2. **State Management Dasar**
   - Konsep state dalam React
   - Local component state
   - Immutable state updates

#### Praktik:
```jsx
// UserCard.js
import PropTypes from 'prop-types';

function UserCard({ name, email, avatar, isOnline = false }) {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
      <span className={isOnline ? 'online' : 'offline'}>
        {isOnline ? 'Online' : 'Offline'}
      </span>
    </div>
  );
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  email: PropTypes.string.isRequired,
  avatar: PropTypes.string,
  isOnline: PropTypes.bool
};

// App.js
function App() {
  const users = [
    { id: 1, name: 'John Doe', email: 'john@example.com', isOnline: true },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com', isOnline: false }
  ];

  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} {...user} />
      ))}
    </div>
  );
}
```

#### Tugas:
- Buat komponen UserCard dengan props
- Implementasikan PropTypes untuk validation
- Render list of users dengan map()

---

## MINGGU 6

### Hari 1: Hooks (useState, useEffect)

#### Materi Pembelajaran:
1. **useState Hook**
   - Mengelola state dalam functional components
   - State updates dan re-rendering
   - Multiple state variables

2. **useEffect Hook**
   - Side effects dalam React
   - Component lifecycle dengan hooks
   - Cleanup functions

#### Praktik:
```jsx
// Counter.js
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  const increment = () => setCount(count + step);
  const decrement = () => setCount(count - step);
  const reset = () => setCount(0);

  return (
    <div>
      <h2>Counter: {count}</h2>
      <input 
        type="number" 
        value={step} 
        onChange={(e) => setStep(Number(e.target.value))}
        placeholder="Step"
      />
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// Timer.js
import { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isActive, setIsActive] = useState(false);

  useEffect(() => {
    let interval = null;
    if (isActive) {
      interval = setInterval(() => {
        setSeconds(seconds => seconds + 1);
      }, 1000);
    } else if (!isActive && seconds !== 0) {
      clearInterval(interval);
    }
    return () => clearInterval(interval);
  }, [isActive, seconds]);

  return (
    <div>
      <h2>Timer: {seconds}s</h2>
      <button onClick={() => setIsActive(!isActive)}>
        {isActive ? 'Pause' : 'Start'}
      </button>
      <button onClick={() => setSeconds(0)}>Reset</button>
    </div>
  );
}
```

### Hari 2: Event Handling & Forms

#### Materi Pembelajaran:
1. **Event Handling**
   - onClick, onChange, onSubmit events
   - Event object dan preventDefault
   - Event delegation

2. **Forms dalam React**
   - Controlled components
   - Form validation
   - Multiple input handling

#### Praktik:
```jsx
// ContactForm.js
import { useState } from 'react';

function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: '',
    category: 'general'
  });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };

  const validateForm = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) {
      newErrors.name = 'Nama harus diisi';
    }
    
    if (!formData.email.trim()) {
      newErrors.email = 'Email harus diisi';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Format email tidak valid';
    }
    
    if (!formData.message.trim()) {
      newErrors.message = 'Pesan harus diisi';
    }
    
    return newErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validateForm();
    
    if (Object.keys(newErrors).length === 0) {
      console.log('Form submitted:', formData);
      // Reset form
      setFormData({
        name: '',
        email: '',
        message: '',
        category: 'general'
      });
    } else {
      setErrors(newErrors);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Nama:</label>
        <input
          type="text"
          name="name"
          value={formData.name}
          onChange={handleChange}
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>
      
      <div>
        <label>Email:</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <label>Kategori:</label>
        <select name="category" value={formData.category} onChange={handleChange}>
          <option value="general">Umum</option>
          <option value="support">Dukungan</option>
          <option value="feedback">Feedback</option>
        </select>
      </div>
      
      <div>
        <label>Pesan:</label>
        <textarea
          name="message"
          value={formData.message}
          onChange={handleChange}
          rows="4"
        />
        {errors.message && <span className="error">{errors.message}</span>}
      </div>
      
      <button type="submit">Kirim</button>
    </form>
  );
}
```

---

## MINGGU 7

### Hari 1: Routing dengan React Router

#### Materi Pembelajaran:
1. **React Router Setup**
   - Install dan konfigurasi React Router
   - BrowserRouter vs HashRouter
   - Route configuration

2. **Navigation**
   - Link vs NavLink
   - Programmatic navigation
   - Route parameters

#### Setup:
```bash
npm install react-router-dom
```

#### Praktik:
```jsx
// App.js
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';
import Contact from './pages/Contact';
import UserProfile from './pages/UserProfile';
import NotFound from './pages/NotFound';

function App() {
  return (
    <Router>
      <nav>
        <ul>
          <li><Link to="/">Home</Link></li>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/contact">Contact</Link></li>
          <li><Link to="/user/123">User Profile</Link></li>
        </ul>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
        <Route path="/user/:id" element={<UserProfile />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </Router>
  );
}

// pages/UserProfile.js
import { useParams, useNavigate } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  const navigate = useNavigate();
  
  const goBack = () => {
    navigate(-1);
  };
  
  const goHome = () => {
    navigate('/');
  };

  return (
    <div>
      <h2>User Profile</h2>
      <p>User ID: {id}</p>
      <button onClick={goBack}>Kembali</button>
      <button onClick={goHome}>Home</button>
    </div>
  );
}
```

### Hari 2: Integrasi API (fetch data dari NestJS)

#### Materi Pembelajaran:
1. **HTTP Requests dalam React**
   - Fetch API vs Axios
   - Async/await dalam useEffect
   - Loading states dan error handling

2. **Data Fetching Patterns**
   - Fetch on mount
   - Conditional fetching
   - Data caching strategies

#### Praktik:
```jsx
// hooks/useApi.js
import { useState, useEffect } from 'react';

function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// components/BookList.js
import { useState } from 'react';
import useApi from '../hooks/useApi';

function BookList() {
  const [refreshKey, setRefreshKey] = useState(0);
  const { data: books, loading, error } = useApi(`http://localhost:3000/books?refresh=${refreshKey}`);

  const refresh = () => {
    setRefreshKey(prev => prev + 1);
  };

  const deleteBook = async (id) => {
    try {
      const response = await fetch(`http://localhost:3000/books/${id}`, {
        method: 'DELETE'
      });
      
      if (response.ok) {
        refresh(); // Refresh data setelah delete
      }
    } catch (error) {
      console.error('Error deleting book:', error);
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h2>Daftar Buku</h2>
      <button onClick={refresh}>Refresh</button>
      
      {books && books.length > 0 ? (
        <ul>
          {books.map(book => (
            <li key={book.id}>
              <h3>{book.title}</h3>
              <p>Penulis: {book.author}</p>
              <p>ISBN: {book.isbn}</p>
              <button onClick={() => deleteBook(book.id)}>
                Hapus
              </button>
            </li>
          ))}
        </ul>
      ) : (
        <p>Tidak ada buku tersedia</p>
      )}
    </div>
  );
}

// components/AddBook.js
import { useState } from 'react';

function AddBook({ onBookAdded }) {
  const [formData, setFormData] = useState({
    title: '',
    author: '',
    isbn: ''
  });
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    try {
      const response = await fetch('http://localhost:3000/books', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(formData)
      });
      
      if (response.ok) {
        setFormData({ title: '', author: '', isbn: '' });
        onBookAdded(); // Callback untuk refresh data
      }
    } catch (error) {
      console.error('Error adding book:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <h3>Tambah Buku Baru</h3>
      <input
        type="text"
        placeholder="Judul"
        value={formData.title}
        onChange={(e) => setFormData({...formData, title: e.target.value})}
        required
      />
      <input
        type="text"
        placeholder="Penulis"
        value={formData.author}
        onChange={(e) => setFormData({...formData, author: e.target.value})}
        required
      />
      <input
        type="text"
        placeholder="ISBN"
        value={formData.isbn}
        onChange={(e) => setFormData({...formData, isbn: e.target.value})}
        required
      />
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Menambah...' : 'Tambah Buku'}
      </button>
    </form>
  );
}
```

---

## MINGGU 8

### Hari 1: UI & Styling (Tailwind/Chakra UI)

#### Materi Pembelajaran:
1. **CSS dalam React**
   - CSS Modules
   - Styled Components
   - CSS-in-JS libraries

2. **UI Framework Integration**
   - Setup Tailwind CSS
   - Setup Chakra UI
   - Component styling best practices

#### Setup Tailwind CSS:
```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

#### Praktik dengan Tailwind:
```jsx
// components/Card.js
function Card({ title, description, image, onAction }) {
  return (
    <div className="max-w-sm rounded-lg overflow-hidden shadow-lg bg-white hover:shadow-xl transition-shadow duration-300">
      <img 
        className="w-full h-48 object-cover" 
        src={image} 
        alt={title}
      />
      <div className="px-6 py-4">
        <h3 className="font-bold text-xl mb-2 text-gray-800">
          {title}
        </h3>
        <p className="text-gray-600 text-base">
          {description}
        </p>
      </div>
      <div className="px-6 pt-4 pb-6">
        <button 
          onClick={onAction}
          className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full transition-colors duration-200"
        >
          Lihat Detail
        </button>
      </div>
    </div>
  );
}

// components/Layout.js
function Layout({ children }) {
  return (
    <div className="min-h-screen bg-gray-100">
      <header className="bg-white shadow-sm">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex justify-between items-center py-6">
            <h1 className="text-3xl font-bold text-gray-900">
              My App
            </h1>
            <nav className="space-x-8">
              <a href="/" className="text-gray-500 hover:text-gray-900">
                Home
              </a>
              <a href="/about" className="text-gray-500 hover:text-gray-900">
                About
              </a>
            </nav>
          </div>
        </div>
      </header>
      
      <main className="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
        {children}
      </main>
      
      <footer className="bg-gray-800 text-white py-8">
        <div className="max-w-7xl mx-auto px-4 text-center">
          <p>&copy; 2024 My App. All rights reserved.</p>
        </div>
      </footer>
    </div>
  );
}
```

### Hari 2: Mini Project - Todo App dengan React

#### Project Requirements:
1. **Fitur yang harus dibuat:**
   - Tambah todo baru
   - Mark todo sebagai completed
   - Edit todo yang sudah ada
   - Hapus todo
   - Filter todo (All, Active, Completed)
   - Persist data ke localStorage

#### Praktik:
```jsx
// App.js - Todo App
import { useState, useEffect } from 'react';
import TodoForm from './components/TodoForm';
import TodoList from './components/TodoList';
import TodoFilter from './components/TodoFilter';

function App() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');

  // Load todos from localStorage
  useEffect(() => {
    const savedTodos = localStorage.getItem('todos');
    if (savedTodos) {
      setTodos(JSON.parse(savedTodos));
    }
  }, []);

  // Save todos to localStorage
  useEffect(() => {
    localStorage.setItem('todos', JSON.stringify(todos));
  }, [todos]);

  const addTodo = (text) => {
    const newTodo = {
      id: Date.now(),
      text,
      completed: false,
      createdAt: new Date().toISOString()
    };
    setTodos([...todos, newTodo]);
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  const editTodo = (id, newText) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, text: newText } : todo
    ));
  };

  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });

  const stats = {
    total: todos.length,
    active: todos.filter(t => !t.completed).length,
    completed: todos.filter(t => t.completed).length
  };

  return (
    <div className="min-h-screen bg-gray-100 py-8">
      <div className="max-w-md mx-auto bg-white rounded-lg shadow-md p-6">
        <h1 className="text-2xl font-bold text-center mb-6">
          Todo App
        </h1>
        
        <TodoForm onAddTodo={addTodo} />
        
        <div className="mt-6">
          <TodoFilter 
            currentFilter={filter} 
            onFilterChange={setFilter}
            stats={stats}
          />
        </div>
        
        <TodoList
          todos={filteredTodos}
          onToggleTodo={toggleTodo}
          onDeleteTodo={deleteTodo}
          onEditTodo={editTodo}
        />
        
        {todos.length === 0 && (
          <p className="text-center text-gray-500 mt-8">
            Belum ada todo. Tambahkan yang pertama!
          </p>
        )}
      </div>
    </div>
  );
}
```

#### Deliverables:
- Todo App yang fully functional
- Responsive design dengan Tailwind CSS
- Local storage persistence
- Clean code dengan proper component structure

---

## EVALUASI BULAN 2

### Kriteria Penilaian:
1. **Pemahaman Konsep React (30%)**
   - Components dan JSX
   - Props dan State
   - Hooks (useState, useEffect)

2. **Implementasi Teknis (40%)**
   - Event handling dan forms
   - API integration
   - Routing dengan React Router

3. **Todo App Project (30%)**
   - Kelengkapan fitur
   - UI/UX design
   - Code quality

### Checklist Kompetensi:
- [ ] Dapat membuat functional components
- [ ] Memahami props dan state management
- [ ] Dapat menggunakan hooks dengan benar
- [ ] Dapat handle forms dan events
- [ ] Dapat mengintegrasikan dengan API
- [ ] Memahami routing dalam React
- [ ] Dapat styling dengan CSS framework

---

## RESOURCES TAMBAHAN

### Dokumentasi:
- [React Official Documentation](https://react.dev/)
- [React Router Documentation](https://reactrouter.com/)
- [Tailwind CSS Documentation](https://tailwindcss.com/)

### Tools:
- React Developer Tools (Browser Extension)
- VS Code dengan React extensions
- JSON Server untuk mock API

### Best Practices:
1. Gunakan functional components dengan hooks
2. Pisahkan logic ke custom hooks
3. Implementasikan proper error handling
4. Gunakan PropTypes untuk type checking
5. Optimasi performance dengan useMemo/useCallback

---

*Modul ini dirancang untuk memberikan pemahaman mendalam tentang React dan pengembangan frontend modern. Pastikan untuk praktik secara konsisten dan membangun project yang berkualitas.*
