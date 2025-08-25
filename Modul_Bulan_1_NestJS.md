# MODUL PEMBELAJARAN BULAN 1
## NestJS Fundamentals (Backend Development)

### TUJUAN PEMBELAJARAN
Setelah menyelesaikan modul ini, peserta akan mampu:
- Memahami konsep dasar NestJS dan arsitektur backend modern
- Membuat aplikasi backend dengan NestJS dari awal
- Mengimplementasikan CRUD API yang lengkap
- Menerapkan best practices dalam pengembangan backend

---

## MINGGU 1

### Hari 1: Intro NestJS & Setup Project

#### Materi Pembelajaran:
1. **Pengenalan NestJS**
   - Apa itu NestJS dan mengapa menggunakannya
   - Perbandingan dengan framework lain (Express, Fastify)
   - Arsitektur NestJS (Modular, Decorator-based)

2. **Setup Development Environment**
   - Install Node.js dan npm/yarn
   - Install NestJS CLI
   - Membuat project baru

3. **Struktur Project NestJS**
   - Folder structure dan file penting
   - main.ts sebagai entry point
   - app.module.ts sebagai root module

#### Praktik:
```bash
# Install NestJS CLI
npm i -g @nestjs/cli

# Buat project baru
nest new my-first-nestjs-app

# Jalankan aplikasi
npm run start:dev
```

#### Tugas:
- Setup project NestJS baru
- Eksplorasi struktur folder
- Jalankan aplikasi dan akses http://localhost:3000

### Hari 2: Membuat Controller & Service Dasar

#### Materi Pembelajaran:
1. **Controller dalam NestJS**
   - Konsep Controller dan routing
   - Decorator @Controller, @Get, @Post
   - Request dan Response handling

2. **Service dan Business Logic**
   - Konsep Service layer
   - Decorator @Injectable
   - Dependency Injection dasar

#### Praktik:
```typescript
// books.controller.ts
@Controller('books')
export class BooksController {
  @Get()
  findAll() {
    return 'Semua buku';
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `Buku dengan ID: ${id}`;
  }
}

// books.service.ts
@Injectable()
export class BooksService {
  private books = [
    { id: 1, title: 'Belajar NestJS', author: 'John Doe' }
  ];

  findAll() {
    return this.books;
  }
}
```

#### Tugas:
- Buat controller untuk mengelola buku
- Buat service untuk business logic
- Test endpoint dengan Postman/Thunder Client

---

## MINGGU 2

### Hari 1: Dependency Injection & Module

#### Materi Pembelajaran:
1. **Dependency Injection**
   - Konsep DI dalam NestJS
   - Provider dan Consumer
   - Constructor injection

2. **Module System**
   - Decorator @Module
   - Providers, Controllers, Imports, Exports
   - Feature modules

#### Praktik:
```typescript
// books.module.ts
@Module({
  controllers: [BooksController],
  providers: [BooksService],
  exports: [BooksService]
})
export class BooksModule {}

// app.module.ts
@Module({
  imports: [BooksModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### Hari 2: CRUD API (Bagian 1 – Create & Read)

#### Materi Pembelajaran:
1. **HTTP Methods**
   - GET untuk Read operations
   - POST untuk Create operations
   - Request body dan validation

2. **Data Transfer Objects (DTO)**
   - Konsep DTO
   - Class-based DTO
   - Interface vs Class

#### Praktik:
```typescript
// create-book.dto.ts
export class CreateBookDto {
  title: string;
  author: string;
  isbn: string;
}

// books.controller.ts
@Controller('books')
export class BooksController {
  constructor(private booksService: BooksService) {}

  @Post()
  create(@Body() createBookDto: CreateBookDto) {
    return this.booksService.create(createBookDto);
  }

  @Get()
  findAll() {
    return this.booksService.findAll();
  }
}
```

---

## MINGGU 3

### Hari 1: CRUD API (Bagian 2 – Update & Delete)

#### Materi Pembelajaran:
1. **Update Operations**
   - PUT vs PATCH methods
   - Partial updates
   - Update DTO

2. **Delete Operations**
   - DELETE method
   - Soft delete vs Hard delete
   - Error handling

#### Praktik:
```typescript
// update-book.dto.ts
export class UpdateBookDto {
  title?: string;
  author?: string;
  isbn?: string;
}

// books.controller.ts
@Put(':id')
update(@Param('id') id: string, @Body() updateBookDto: UpdateBookDto) {
  return this.booksService.update(+id, updateBookDto);
}

@Delete(':id')
remove(@Param('id') id: string) {
  return this.booksService.remove(+id);
}
```

### Hari 2: Validation & DTO

#### Materi Pembelajaran:
1. **Input Validation**
   - class-validator library
   - Validation decorators
   - ValidationPipe

2. **Error Handling**
   - Built-in exceptions
   - Custom exceptions
   - Exception filters

#### Praktik:
```typescript
// create-book.dto.ts
import { IsString, IsNotEmpty, IsISBN } from 'class-validator';

export class CreateBookDto {
  @IsString()
  @IsNotEmpty()
  title: string;

  @IsString()
  @IsNotEmpty()
  author: string;

  @IsISBN()
  isbn: string;
}

// main.ts
app.useGlobalPipes(new ValidationPipe());
```

---

## MINGGU 4

### Hari 1: Middleware, Guards & Interceptors

#### Materi Pembelajaran:
1. **Middleware**
   - Konsep middleware
   - Custom middleware
   - Global vs route-specific

2. **Guards**
   - Authentication guards
   - Authorization guards
   - Role-based access

3. **Interceptors**
   - Request/Response transformation
   - Logging interceptor
   - Cache interceptor

#### Praktik:
```typescript
// logger.middleware.ts
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.url}`);
    next();
  }
}

// auth.guard.ts
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

### Hari 2: Project Mini - API Buku dengan NestJS

#### Project Requirements:
1. **Fitur yang harus dibuat:**
   - CRUD lengkap untuk buku
   - Validation untuk semua input
   - Error handling yang proper
   - Logging middleware
   - Basic authentication

2. **Struktur Data Buku:**
   ```typescript
   interface Book {
     id: number;
     title: string;
     author: string;
     isbn: string;
     publishedYear: number;
     genre: string;
     createdAt: Date;
     updatedAt: Date;
   }
   ```

#### Deliverables:
- Source code lengkap
- API documentation
- Postman collection untuk testing
- README dengan instruksi setup

---

## EVALUASI BULAN 1

### Kriteria Penilaian:
1. **Pemahaman Konsep (30%)**
   - Arsitektur NestJS
   - Dependency Injection
   - Module system

2. **Implementasi Teknis (40%)**
   - CRUD API yang berfungsi
   - Validation dan error handling
   - Code quality dan struktur

3. **Project Mini (30%)**
   - Kelengkapan fitur
   - Documentation
   - Best practices

### Checklist Kompetensi:
- [ ] Dapat membuat project NestJS dari awal
- [ ] Memahami konsep Controller dan Service
- [ ] Dapat mengimplementasikan CRUD API
- [ ] Memahami Dependency Injection
- [ ] Dapat menggunakan validation dan error handling
- [ ] Memahami middleware, guards, dan interceptors

---

## RESOURCES TAMBAHAN

### Dokumentasi:
- [NestJS Official Documentation](https://docs.nestjs.com/)
- [NestJS CLI Documentation](https://docs.nestjs.com/cli/overview)

### Tools:
- Postman/Thunder Client untuk API testing
- VS Code dengan NestJS extensions
- Node.js dan npm/yarn

### Best Practices:
1. Gunakan TypeScript dengan strict mode
2. Implementasikan proper error handling
3. Gunakan validation untuk semua input
4. Pisahkan business logic ke service layer
5. Dokumentasikan API dengan Swagger

---

*Modul ini dirancang untuk memberikan fondasi yang kuat dalam pengembangan backend dengan NestJS. Pastikan untuk mengikuti setiap praktik dan menyelesaikan semua tugas untuk pemahaman yang optimal.*
