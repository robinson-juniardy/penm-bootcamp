# MODUL PEMBELAJARAN BULAN 3
## Fullstack Project (NestJS + React Integration)

### TUJUAN PEMBELAJARAN
Setelah menyelesaikan modul ini, peserta akan mampu:
- Mengintegrasikan frontend React dengan backend NestJS
- Mengimplementasikan sistem authentication lengkap
- Membangun aplikasi fullstack yang production-ready
- Melakukan deployment aplikasi ke platform cloud

---

## MINGGU 9

### Hari 1: Membangun API Authentication (JWT, Login, Register)

#### Materi Pembelajaran:
1. **JWT (JSON Web Token)**
   - Konsep JWT dan cara kerjanya
   - Access token vs Refresh token
   - JWT security best practices

2. **Authentication Module**
   - Passport.js integration
   - Local strategy untuk login
   - JWT strategy untuk protected routes

#### Setup Dependencies:
```bash
# Backend (NestJS)
npm install @nestjs/passport @nestjs/jwt passport passport-local passport-jwt bcryptjs
npm install -D @types/passport-local @types/passport-jwt @types/bcryptjs
```

#### Praktik Backend:
```typescript
// auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { LocalStrategy } from './local.strategy';
import { JwtStrategy } from './jwt.strategy';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: process.env.JWT_SECRET || 'secretKey',
      signOptions: { expiresIn: '24h' },
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, LocalStrategy, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}

// auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcryptjs';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async validateUser(email: string, password: string): Promise<any> {
    const user = await this.usersService.findByEmail(email);
    if (user && await bcrypt.compare(password, user.password)) {
      const { password, ...result } = user;
      return result;
    }
    return null;
  }

  async login(user: any) {
    const payload = { email: user.email, sub: user.id, role: user.role };
    return {
      access_token: this.jwtService.sign(payload),
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role,
      },
    };
  }

  async register(createUserDto: any) {
    const existingUser = await this.usersService.findByEmail(createUserDto.email);
    if (existingUser) {
      throw new UnauthorizedException('Email sudah terdaftar');
    }

    const hashedPassword = await bcrypt.hash(createUserDto.password, 10);
    const user = await this.usersService.create({
      ...createUserDto,
      password: hashedPassword,
    });

    return this.login(user);
  }
}

// auth/auth.controller.ts
import { Controller, Post, Body, UseGuards, Request } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { AuthService } from './auth.service';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('register')
  async register(@Body() createUserDto: any) {
    return this.authService.register(createUserDto);
  }

  @UseGuards(AuthGuard('local'))
  @Post('login')
  async login(@Request() req) {
    return this.authService.login(req.user);
  }

  @UseGuards(AuthGuard('jwt'))
  @Post('profile')
  getProfile(@Request() req) {
    return req.user;
  }
}

// auth/jwt.strategy.ts
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET || 'secretKey',
    });
  }

  async validate(payload: any) {
    return { 
      userId: payload.sub, 
      email: payload.email, 
      role: payload.role 
    };
  }
}
```

### Hari 2: Integrasi Auth di React (Login/Logout)

#### Materi Pembelajaran:
1. **Authentication Context**
   - React Context untuk global state
   - Token management
   - Automatic logout pada token expiry

2. **Protected Routes**
   - Route guards
   - Redirect setelah login
   - Persistent login state

#### Praktik Frontend:
```jsx
// contexts/AuthContext.js
import { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext();

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [token, setToken] = useState(localStorage.getItem('token'));

  useEffect(() => {
    if (token) {
      // Verify token dengan backend
      verifyToken();
    } else {
      setLoading(false);
    }
  }, [token]);

  const verifyToken = async () => {
    try {
      const response = await fetch('http://localhost:3000/auth/profile', {
        headers: {
          'Authorization': `Bearer ${token}`
        }
      });
      
      if (response.ok) {
        const userData = await response.json();
        setUser(userData);
      } else {
        logout();
      }
    } catch (error) {
      logout();
    } finally {
      setLoading(false);
    }
  };

  const login = async (email, password) => {
    try {
      const response = await fetch('http://localhost:3000/auth/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ email, password }),
      });

      if (response.ok) {
        const data = await response.json();
        setToken(data.access_token);
        setUser(data.user);
        localStorage.setItem('token', data.access_token);
        return { success: true };
      } else {
        const error = await response.json();
        return { success: false, message: error.message };
      }
    } catch (error) {
      return { success: false, message: 'Network error' };
    }
  };

  const register = async (userData) => {
    try {
      const response = await fetch('http://localhost:3000/auth/register', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(userData),
      });

      if (response.ok) {
        const data = await response.json();
        setToken(data.access_token);
        setUser(data.user);
        localStorage.setItem('token', data.access_token);
        return { success: true };
      } else {
        const error = await response.json();
        return { success: false, message: error.message };
      }
    } catch (error) {
      return { success: false, message: 'Network error' };
    }
  };

  const logout = () => {
    setToken(null);
    setUser(null);
    localStorage.removeItem('token');
  };

  const value = {
    user,
    token,
    login,
    register,
    logout,
    loading,
    isAuthenticated: !!user,
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

// components/LoginForm.js
import { useState } from 'react';
import { useAuth } from '../contexts/AuthContext';
import { useNavigate, Link } from 'react-router-dom';

function LoginForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  
  const { login } = useAuth();
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError('');

    const result = await login(formData.email, formData.password);
    
    if (result.success) {
      navigate('/dashboard');
    } else {
      setError(result.message);
    }
    
    setLoading(false);
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full space-y-8">
        <div>
          <h2 className="mt-6 text-center text-3xl font-extrabold text-gray-900">
            Masuk ke Akun Anda
          </h2>
        </div>
        
        <form className="mt-8 space-y-6" onSubmit={handleSubmit}>
          {error && (
            <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded">
              {error}
            </div>
          )}
          
          <div>
            <input
              type="email"
              required
              className="appearance-none rounded-md relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"
              placeholder="Email"
              value={formData.email}
              onChange={(e) => setFormData({...formData, email: e.target.value})}
            />
          </div>
          
          <div>
            <input
              type="password"
              required
              className="appearance-none rounded-md relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"
              placeholder="Password"
              value={formData.password}
              onChange={(e) => setFormData({...formData, password: e.target.value})}
            />
          </div>

          <div>
            <button
              type="submit"
              disabled={loading}
              className="group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50"
            >
              {loading ? 'Memproses...' : 'Masuk'}
            </button>
          </div>
          
          <div className="text-center">
            <Link to="/register" className="text-indigo-600 hover:text-indigo-500">
              Belum punya akun? Daftar di sini
            </Link>
          </div>
        </form>
      </div>
    </div>
  );
}
```

---

## MINGGU 10

### Hari 1: Protected Routes & User Session

#### Materi Pembelajaran:
1. **Route Protection**
   - Private routes component
   - Role-based access control
   - Redirect logic

2. **Session Management**
   - Token refresh mechanism
   - Auto logout pada inactivity
   - Session persistence

#### Praktik:
```jsx
// components/ProtectedRoute.js
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../contexts/AuthContext';

function ProtectedRoute({ children, requiredRole = null }) {
  const { isAuthenticated, user, loading } = useAuth();
  const location = useLocation();

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="animate-spin rounded-full h-32 w-32 border-b-2 border-indigo-500"></div>
      </div>
    );
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }

  return children;
}

// App.js dengan Protected Routes
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './contexts/AuthContext';
import ProtectedRoute from './components/ProtectedRoute';
import LoginForm from './components/LoginForm';
import RegisterForm from './components/RegisterForm';
import Dashboard from './pages/Dashboard';
import AdminPanel from './pages/AdminPanel';
import Profile from './pages/Profile';

function App() {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          <Route path="/login" element={<LoginForm />} />
          <Route path="/register" element={<RegisterForm />} />
          
          <Route path="/dashboard" element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          } />
          
          <Route path="/profile" element={
            <ProtectedRoute>
              <Profile />
            </ProtectedRoute>
          } />
          
          <Route path="/admin" element={
            <ProtectedRoute requiredRole="admin">
              <AdminPanel />
            </ProtectedRoute>
          } />
          
          <Route path="/" element={<Navigate to="/dashboard" />} />
        </Routes>
      </Router>
    </AuthProvider>
  );
}
```

### Hari 2: CRUD Data dengan Auth (contoh: Postingan Blog)

#### Materi Pembelajaran:
1. **Authenticated API Calls**
   - Authorization headers
   - API service dengan token
   - Error handling untuk expired tokens

2. **CRUD Operations**
   - Create, Read, Update, Delete dengan authentication
   - User-specific data filtering
   - Permission-based operations

#### Praktik Backend:
```typescript
// posts/posts.controller.ts
import { Controller, Get, Post, Body, Patch, Param, Delete, UseGuards, Request } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { PostsService } from './posts.service';
import { CreatePostDto } from './dto/create-post.dto';
import { UpdatePostDto } from './dto/update-post.dto';

@Controller('posts')
@UseGuards(AuthGuard('jwt'))
export class PostsController {
  constructor(private readonly postsService: PostsService) {}

  @Post()
  create(@Body() createPostDto: CreatePostDto, @Request() req) {
    return this.postsService.create({
      ...createPostDto,
      authorId: req.user.userId,
    });
  }

  @Get()
  findAll(@Request() req) {
    // Admin bisa lihat semua, user biasa hanya miliknya
    if (req.user.role === 'admin') {
      return this.postsService.findAll();
    }
    return this.postsService.findByAuthor(req.user.userId);
  }

  @Get(':id')
  findOne(@Param('id') id: string, @Request() req) {
    return this.postsService.findOne(+id, req.user.userId, req.user.role);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updatePostDto: UpdatePostDto, @Request() req) {
    return this.postsService.update(+id, updatePostDto, req.user.userId, req.user.role);
  }

  @Delete(':id')
  remove(@Param('id') id: string, @Request() req) {
    return this.postsService.remove(+id, req.user.userId, req.user.role);
  }
}
```

#### Praktik Frontend:
```jsx
// services/apiService.js
class ApiService {
  constructor() {
    this.baseURL = 'http://localhost:3000';
  }

  getAuthHeaders() {
    const token = localStorage.getItem('token');
    return {
      'Content-Type': 'application/json',
      ...(token && { 'Authorization': `Bearer ${token}` })
    };
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: this.getAuthHeaders(),
      ...options
    };

    try {
      const response = await fetch(url, config);
      
      if (response.status === 401) {
        // Token expired, redirect to login
        localStorage.removeItem('token');
        window.location.href = '/login';
        return;
      }

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      return await response.json();
    } catch (error) {
      throw error;
    }
  }

  // Posts API methods
  async getPosts() {
    return this.request('/posts');
  }

  async getPost(id) {
    return this.request(`/posts/${id}`);
  }

  async createPost(postData) {
    return this.request('/posts', {
      method: 'POST',
      body: JSON.stringify(postData)
    });
  }

  async updatePost(id, postData) {
    return this.request(`/posts/${id}`, {
      method: 'PATCH',
      body: JSON.stringify(postData)
    });
  }

  async deletePost(id) {
    return this.request(`/posts/${id}`, {
      method: 'DELETE'
    });
  }
}

export default new ApiService();

// components/PostList.js
import { useState, useEffect } from 'react';
import { useAuth } from '../contexts/AuthContext';
import apiService from '../services/apiService';
import PostCard from './PostCard';

function PostList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const { user } = useAuth();

  useEffect(() => {
    loadPosts();
  }, []);

  const loadPosts = async () => {
    try {
      setLoading(true);
      const data = await apiService.getPosts();
      setPosts(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  const handleDeletePost = async (id) => {
    if (window.confirm('Yakin ingin menghapus post ini?')) {
      try {
        await apiService.deletePost(id);
        setPosts(posts.filter(post => post.id !== id));
      } catch (err) {
        alert('Gagal menghapus post: ' + err.message);
      }
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h2 className="text-2xl font-bold">My Posts</h2>
        <Link 
          to="/posts/create"
          className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
        >
          Buat Post Baru
        </Link>
      </div>

      {posts.length === 0 ? (
        <p className="text-gray-500">Belum ada post. Buat yang pertama!</p>
      ) : (
        <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
          {posts.map(post => (
            <PostCard
              key={post.id}
              post={post}
              onDelete={handleDeletePost}
              canEdit={post.authorId === user.id || user.role === 'admin'}
            />
          ))}
        </div>
      )}
    </div>
  );
}
```

---

## MINGGU 11

### Hari 1: Deployment Backend (Render/Vercel/Heroku)

#### Materi Pembelajaran:
1. **Persiapan Deployment**
   - Environment variables
   - Production configuration
   - Database setup untuk production

2. **Platform Deployment**
   - Render deployment
   - Railway deployment
   - Environment configuration

#### Praktik:
```typescript
// main.ts - Production configuration
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // CORS configuration untuk production
  app.enableCors({
    origin: process.env.FRONTEND_URL || 'http://localhost:3000',
    credentials: true,
  });
  
  app.useGlobalPipes(new ValidationPipe());
  
  const port = process.env.PORT || 3000;
  await app.listen(port, '0.0.0.0');
  
  console.log(`Application is running on: ${await app.getUrl()}`);
}
bootstrap();

// package.json scripts
{
  "scripts": {
    "build": "nest build",
    "start": "node dist/main",
    "start:prod": "node dist/main"
  }
}
```

#### Environment Variables:
```bash
# .env.production
DATABASE_URL=your_production_database_url
JWT_SECRET=your_super_secret_jwt_key
FRONTEND_URL=https://your-frontend-domain.com
PORT=3000
```

### Hari 2: Deployment Frontend (Netlify/Vercel)

#### Materi Pembelajaran:
1. **Build Optimization**
   - Production build configuration
   - Environment variables untuk frontend
   - Build performance optimization

2. **Deployment Platforms**
   - Netlify deployment
   - Vercel deployment
   - Custom domain setup

#### Praktik:
```javascript
// .env.production (React)
REACT_APP_API_URL=https://your-backend-api.com
REACT_APP_APP_NAME=Blog App

// src/config/api.js
const config = {
  apiUrl: process.env.REACT_APP_API_URL || 'http://localhost:3000',
  appName: process.env.REACT_APP_APP_NAME || 'Blog App',
};

export default config;

// netlify.toml
[build]
  publish = "build"
  command = "npm run build"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

# vercel.json
{
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}
```

---

## MINGGU 12

### Hari 1: Final Project - Fullstack Blog App (Bagian 1)

#### Project Specifications:
1. **Backend Features:**
   - User authentication (register/login)
   - CRUD operations untuk posts
   - Role-based access (admin/user)
   - Comment system
   - File upload untuk gambar

2. **Frontend Features:**
   - Responsive design
   - Authentication flow
   - Post management
   - Comment system
   - User profile management

#### Implementation:
```typescript
// Backend: comments.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, CreateDateColumn } from 'typeorm';
import { User } from '../users/user.entity';
import { Post } from '../posts/post.entity';

@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  @ManyToOne(() => User, user => user.comments)
  author: User;

  @ManyToOne(() => Post, post => post.comments)
  post: Post;

  @CreateDateColumn()
  createdAt: Date;
}
```

```jsx
// Frontend: BlogPost.js
import { useState, useEffect } from 'react';
import { useParams } from 'react-router-dom';
import { useAuth } from '../contexts/AuthContext';
import apiService from '../services/apiService';
import CommentSection from '../components/CommentSection';

function BlogPost() {
  const { id } = useParams();
  const { user } = useAuth();
  const [post, setPost] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadPost();
  }, [id]);

  const loadPost = async () => {
    try {
      const data = await apiService.getPost(id);
      setPost(data);
    } catch (error) {
      console.error('Error loading post:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading) return <div>Loading...</div>;
  if (!post) return <div>Post not found</div>;

  return (
    <article className="max-w-4xl mx-auto px-4 py-8">
      <header className="mb-8">
        <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
        <div className="flex items-center text-gray-600 mb-4">
          <span>By {post.author.name}</span>
          <span className="mx-2">•</span>
          <span>{new Date(post.createdAt).toLocaleDateString('id-ID')}</span>
        </div>
        {post.image && (
          <img 
            src={post.image} 
            alt={post.title}
            className="w-full h-64 object-cover rounded-lg"
          />
        )}
      </header>
      
      <div className="prose max-w-none mb-8">
        <p className="text-lg leading-relaxed">{post.content}</p>
      </div>
      
      <CommentSection postId={post.id} />
    </article>
  );
}
```

### Hari 2: Final Project - Fullstack Blog App (Bagian 2)

#### Final Integration & Testing:
1. **Testing Checklist:**
   - Authentication flow
   - CRUD operations
   - Permission system
   - Responsive design
   - Error handling

2. **Documentation:**
   - API documentation
   - Setup instructions
   - Deployment guide
   - User manual

#### Project Deliverables:
```markdown
# Blog App - Final Project

## Features Implemented
✅ User Authentication (Register/Login/Logout)
✅ Protected Routes & Role-based Access
✅ Post Management (CRUD)
✅ Comment System
✅ File Upload untuk Images
✅ Responsive Design
✅ Error Handling
✅ Production Deployment

## Tech Stack
- **Backend:** NestJS, TypeORM, PostgreSQL, JWT
- **Frontend:** React, React Router, Tailwind CSS
- **Deployment:** Render (Backend), Netlify (Frontend)

## Live Demo
- Frontend: https://your-blog-app.netlify.app
- Backend API: https://your-api.render.com

## Repository
- GitHub: https://github.com/username/fullstack-blog-app
```

---

## EVALUASI FINAL BULAN 3

### Kriteria Penilaian:
1. **Backend Implementation (25%)**
   - Authentication system
   - API design dan security
   - Database design

2. **Frontend Implementation (25%)**
   - User interface dan UX
   - State management
   - API integration

3. **Fullstack Integration (25%)**
   - Authentication flow
   - Data synchronization
   - Error handling

4. **Deployment & Documentation (25%)**
   - Successful deployment
   - Code quality
   - Documentation completeness

### Checklist Kompetensi Final:
- [ ] Dapat mengimplementasikan JWT authentication
- [ ] Dapat membangun protected routes
- [ ] Dapat mengintegrasikan frontend dengan backend
- [ ] Dapat melakukan CRUD operations dengan auth
- [ ] Dapat deploy aplikasi ke production
- [ ] Dapat membuat dokumentasi yang lengkap
- [ ] Dapat menangani error dan edge cases
- [ ] Dapat mengoptimasi performance aplikasi

---

## RESOURCES TAMBAHAN

### Documentation:
- [JWT.io](https://jwt.io/) - JWT debugger
- [Render Documentation](https://render.com/docs)
- [Netlify Documentation](https://docs.netlify.com/)

### Best Practices:
1. Implementasikan proper error handling
2. Gunakan environment variables untuk konfigurasi
3. Implementasikan rate limiting untuk API
4. Gunakan HTTPS untuk production
5. Implementasikan proper logging
6. Buat backup strategy untuk database

### Security Checklist:
- [ ] Password hashing dengan bcrypt
- [ ] JWT secret yang kuat
- [ ] CORS configuration yang benar
- [ ] Input validation dan sanitization
- [ ] Rate limiting untuk API endpoints
- [ ] HTTPS untuk production

---

*Modul ini merupakan culmination dari pembelajaran 3 bulan, di mana peserta akan mengintegrasikan semua pengetahuan yang telah dipelajari untuk membangun aplikasi fullstack yang production-ready.*
