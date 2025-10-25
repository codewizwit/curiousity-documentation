# NestJS

_Angular's architecture, but for the backend — bringing structure, scalability, and sanity to Node.js._

## Overview

If you've ever worked with Angular and thought "I wish my backend had this kind of organization," NestJS is your answer. It brings Angular's best architectural patterns — decorators, dependency injection, modules, and clear separation of concerns — to Node.js backend development.

NestJS isn't just "inspired by" Angular. It deliberately mirrors Angular's patterns so that full-stack TypeScript development feels consistent across the entire application. Same mental model, front to back.

---

## The Restaurant Metaphor

**Express/Koa (bare Node.js frameworks):**
You're running a food truck. You're the chef, the cashier, the cleaner, and the manager. Everything's in one space. It's flexible and fast to start, but as you grow, chaos ensues. Where do orders go? How do you organize ingredients? Who's responsible for what?

**NestJS:**
You're running a professional restaurant with a clear organizational chart:
- **Controllers** = Front-of-house staff (take orders, send responses)
- **Services** = Kitchen staff (business logic, food prep)
- **Modules** = Restaurant departments (kitchen, bar, bakery)
- **Providers/DI** = Supply chain (ingredients delivered where needed)
- **Guards** = Bouncers (authentication, authorization)
- **Interceptors** = Quality control (transform requests/responses)
- **Pipes** = Prep stations (validate and transform incoming data)

Everyone knows their role. New staff onboard easily because the structure is clear. Scaling means adding more modules, not rewriting the entire operation.

---

## Why NestJS?

### 1. **TypeScript-First**
Not just "supports TypeScript" — it's built for TypeScript from the ground up. Full type safety across your entire backend.

### 2. **Familiar for Angular Developers**
If you know Angular, you already know 60% of NestJS. Same decorators, same DI, same module system.

### 3. **Opinionated Architecture**
No more "how should I structure this?" debates. NestJS provides clear conventions while remaining flexible when you need it.

### 4. **Built-in Everything**
- Dependency injection out of the box
- Testing utilities included
- Validation with class-validator
- Configuration management
- Database integration (TypeORM, Prisma, Mongoose)
- GraphQL and WebSocket support
- Microservices architecture

### 5. **Scalability by Default**
The module system naturally supports:
- Monolithic apps
- Microservices
- Serverless functions
- Anything in between

---

## Angular vs NestJS: Side-by-Side Comparison

### The Parallel Architecture

| Concept | Angular (Frontend) | NestJS (Backend) | Purpose |
|---------|-------------------|------------------|---------|
| **Module** | `@NgModule` | `@Module` | Organize related code into cohesive blocks |
| **Component** | `@Component` | `@Controller` | Handle user interactions / HTTP requests |
| **Service** | `@Injectable` | `@Injectable` | Business logic and data operations |
| **Routing** | `RouterModule` | `@Get()`, `@Post()` decorators | Map URLs to handlers |
| **Dependency Injection** | Constructor injection | Constructor injection | Share instances across the app |
| **Guards** | Route guards (`CanActivate`) | `@UseGuards()` | Protect routes (auth, permissions) |
| **Interceptors** | HTTP interceptors | `@UseInterceptors()` | Transform requests/responses |
| **Pipes** | Pipes (`transform`) | `@UsePipes()` | Validate and transform data |

### Code Comparison

**Angular Component:**
```typescript
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html'
})
export class UserListComponent {
  constructor(private userService: UserService) {}

  ngOnInit() {
    this.users = this.userService.getUsers();
  }
}
```

**NestJS Controller:**
```typescript
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get()
  getUsers() {
    return this.userService.getUsers();
  }
}
```

**See the pattern?** Same constructor injection, same service pattern, same clear separation of concerns.

---

## Core Concepts

### 1. Controllers

Controllers handle incoming HTTP requests and return responses. They're like Angular components, but instead of rendering views, they return data.

```typescript
@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Get()
  findAll(): string {
    return this.catsService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.catsService.findOne(id);
  }

  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return this.catsService.create(createCatDto);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return this.catsService.update(id, updateCatDto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.catsService.remove(id);
  }
}
```

**Key Decorators:**
- `@Controller('path')` - Define route prefix
- `@Get()`, `@Post()`, `@Put()`, `@Delete()` - HTTP methods
- `@Param('id')` - Route parameters
- `@Body()` - Request body
- `@Query()` - Query parameters
- `@Headers()` - Request headers

---

### 2. Providers (Services)

Services contain business logic. Just like Angular services, they're injectable and reusable.

```typescript
@Injectable()
export class CatsService {
  private cats: Cat[] = [];

  findAll(): Cat[] {
    return this.cats;
  }

  findOne(id: string): Cat {
    return this.cats.find(cat => cat.id === id);
  }

  create(cat: Cat): Cat {
    this.cats.push(cat);
    return cat;
  }

  update(id: string, cat: Cat): Cat {
    const index = this.cats.findIndex(c => c.id === id);
    this.cats[index] = { ...this.cats[index], ...cat };
    return this.cats[index];
  }

  remove(id: string): void {
    this.cats = this.cats.filter(cat => cat.id !== id);
  }
}
```

**Why Services?**
- Separation of concerns (controllers stay thin)
- Testability (easy to mock)
- Reusability (share logic across controllers)
- Single Responsibility Principle

---

### 3. Modules

Modules organize your application into logical groups. Each module encapsulates related controllers, services, and other providers.

```typescript
@Module({
  imports: [DatabaseModule, ConfigModule],
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]  // Make available to other modules
})
export class CatsModule {}
```

**Module Benefits:**
- **Clear boundaries** - Each module has a specific purpose
- **Lazy loading** - Load modules only when needed
- **Encapsulation** - Private vs public APIs via exports
- **Testability** - Mock entire modules easily

**App Structure Example:**
```
app.module.ts (root)
├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   └── user.entity.ts
├── auth.module.ts
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── jwt.strategy.ts
│   └── guards/
│       └── jwt-auth.guard.ts
└── database.module.ts
    └── database.providers.ts
```

---

### 4. Dependency Injection

NestJS uses Angular's dependency injection pattern. Providers are automatically instantiated and injected where needed.

```typescript
// Service declares it's injectable
@Injectable()
export class CatsService { }

// Module registers the provider
@Module({
  providers: [CatsService]
})
export class CatsModule {}

// Controller receives it via constructor injection
@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}
  // catsService is automatically injected
}
```

**DI Scopes:**
- `DEFAULT` - Singleton (one instance across the app)
- `REQUEST` - New instance per request
- `TRANSIENT` - New instance every time it's injected

---

### 5. Middleware, Guards, Interceptors, and Pipes

NestJS provides a robust request/response pipeline, just like Angular's HTTP interceptors but more granular.

#### Middleware
Executes before the route handler. Used for logging, CORS, body parsing, etc.

```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`[${req.method}] ${req.url}`);
    next();
  }
}
```

#### Guards
Determine if a request should be handled (authentication, authorization).

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return this.validateRequest(request);
  }
}

// Use it
@Controller('cats')
@UseGuards(AuthGuard)
export class CatsController {}
```

#### Interceptors
Transform the response or add extra logic before/after handler execution.

```typescript
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({ data, timestamp: new Date().toISOString() }))
    );
  }
}
```

#### Pipes
Validate and transform input data.

```typescript
@Post()
create(@Body(ValidationPipe) createCatDto: CreateCatDto) {
  return this.catsService.create(createCatDto);
}
```

**Request Flow:**
```
Incoming Request
    ↓
Middleware
    ↓
Guards (can block)
    ↓
Interceptors (before)
    ↓
Pipes (validation/transformation)
    ↓
Route Handler
    ↓
Interceptors (after)
    ↓
Response
```

---

## Real-World Example: Building a REST API

### Step 1: Create a Module

```bash
nest generate module users
nest generate controller users
nest generate service users
```

### Step 2: Define the Entity (DTO)

```typescript
// create-user.dto.ts
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}
```

### Step 3: Build the Service

```typescript
@Injectable()
export class UsersService {
  private users: User[] = [];

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = {
      id: Date.now().toString(),
      ...createUserDto,
      createdAt: new Date()
    };
    this.users.push(user);
    return user;
  }

  async findAll(): Promise<User[]> {
    return this.users;
  }

  async findOne(id: string): Promise<User> {
    const user = this.users.find(u => u.id === id);
    if (!user) {
      throw new NotFoundException(`User #${id} not found`);
    }
    return user;
  }
}
```

### Step 4: Build the Controller

```typescript
@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Post()
  @UsePipes(ValidationPipe)
  async create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Get()
  async findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
}
```

### Step 5: Register in Module

```typescript
@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService]  // If other modules need it
})
export class UsersModule {}
```

**That's it!** You now have a fully functional REST API with validation, error handling, and dependency injection.

---

## NestJS + Database (TypeORM Example)

```typescript
// user.entity.ts
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @CreateDateColumn()
  createdAt: Date;
}

// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>
  ) {}

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.usersRepository.create(createUserDto);
    return this.usersRepository.save(user);
  }

  async findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }
}

// users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService]
})
export class UsersModule {}
```

---

## Testing in NestJS

Testing follows Angular's patterns too. NestJS includes built-in testing utilities.

```typescript
describe('CatsController', () => {
  let controller: CatsController;
  let service: CatsService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [CatsController],
      providers: [
        {
          provide: CatsService,
          useValue: {
            findAll: jest.fn().mockResolvedValue([]),
            create: jest.fn()
          }
        }
      ]
    }).compile();

    controller = module.get<CatsController>(CatsController);
    service = module.get<CatsService>(CatsService);
  });

  it('should return an array of cats', async () => {
    expect(await controller.findAll()).toEqual([]);
    expect(service.findAll).toHaveBeenCalled();
  });
});
```

**Key Testing Features:**
- `Test.createTestingModule()` - Same as Angular's TestBed
- Built-in mocking utilities
- E2E testing with Supertest
- Same Jest integration as Angular

---

## When to Use NestJS

**✅ Great for:**
- REST APIs with clear structure
- GraphQL backends
- Microservices architectures
- Teams familiar with Angular
- Projects requiring scalability
- TypeScript-first development
- Enterprise applications

**⚠️ Maybe overkill for:**
- Simple CRUD apps (Express might be faster to spin up)
- Serverless functions with cold-start sensitivity
- Teams with zero TypeScript experience

---

## NestJS vs Other Node.js Frameworks

| Framework | Philosophy | When to Use |
|-----------|-----------|-------------|
| **Express** | Minimalist, unopinionated | Small apps, microservices, when you want total control |
| **Koa** | Modern Express, middleware-focused | Similar to Express but cleaner async/await |
| **Fastify** | Performance-first | When raw speed is critical |
| **NestJS** | Opinionated, Angular-inspired | Enterprise apps, teams wanting structure |
| **AdonisJS** | Laravel-inspired MVC | Coming from PHP/Laravel background |

**NestJS shines when:**
- Your team already knows Angular
- You need enterprise-grade architecture
- Scalability and maintainability > initial speed
- You want TypeScript everywhere

---

## Key Benefits Summary

### 1. **Consistency Across the Stack**
If you're using Angular on the frontend, NestJS on the backend means:
- Same decorators (`@Injectable`, `@Module`)
- Same dependency injection patterns
- Same testing philosophy
- Shared TypeScript types between frontend and backend

### 2. **Scalability Built-In**
- Modular architecture prevents "big ball of mud"
- Clear separation of concerns
- Easy to split into microservices later
- Dependency injection makes refactoring safer

### 3. **Developer Experience**
- Excellent documentation
- CLI for scaffolding (`nest generate`)
- Strong TypeScript support
- Built-in Swagger/OpenAPI integration
- Great error messages

### 4. **Flexibility**
Despite being opinionated, NestJS is flexible:
- Use any ORM (TypeORM, Prisma, Sequelize, Mongoose)
- Express or Fastify underneath
- GraphQL, REST, WebSockets, gRPC
- Monolith or microservices

### 5. **Production-Ready**
- Battle-tested at scale
- Active community and ecosystem
- Regular updates and long-term support
- Used by major companies

---

## Getting Started

```bash
# Install NestJS CLI
npm i -g @nestjs/cli

# Create new project
nest new my-app

# Generate resources
nest generate module users
nest generate controller users
nest generate service users

# Run development server
npm run start:dev
```

---

## Resources

- [NestJS Documentation](https://docs.nestjs.com/)
- [NestJS CLI](https://docs.nestjs.com/cli/overview)
- [NestJS Courses](https://courses.nestjs.com/)
- [Awesome NestJS](https://github.com/nestjs/awesome-nestjs) - Curated resources
- [NestJS Discord](https://discord.gg/nestjs)

---

## The Bottom Line

**If you know Angular, you already know how to think in NestJS.**

NestJS brings the structure, patterns, and developer experience of Angular to the backend. It's not just "Node.js with decorators" — it's a complete architectural philosophy that scales from prototypes to enterprise applications.

The learning curve is gentle if you're coming from Angular, and the payoff is huge: consistent patterns across your entire stack, better code organization, and a framework that grows with your application.

---

_Backend development doesn't have to be chaotic. NestJS proves that structure and flexibility can coexist._ 🌭

← [Back to Home](../README.md) | [Angular](../angular/README.md) | [REST APIs](../rest-apis/README.md)
