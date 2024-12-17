// 1. **Command**: Represents an action in the system
// 2. **Query**: Represents a request to read data
// 3. **Handlers**: Execute logic for commands and queries
// 4. **Bus**: Dispatches commands and queries
// 5. **Repository**: Data abstraction layer
// 6. **Domain Models**: Represent business entities

// --- Commands ---
export interface Command {
  readonly type: string;
}

export class CreateUserCommand implements Command {
  readonly type = "CREATE_USER";
  constructor(public readonly name: string, public readonly email: string) {}
}

// --- Queries ---
export interface Query {
  readonly type: string;
}

export class GetUserByEmailQuery implements Query {
  readonly type = "GET_USER_BY_EMAIL";
  constructor(public readonly email: string) {}
}

// --- Handlers ---
export interface CommandHandler<T extends Command> {
  handle(command: T): Promise<void>;
}

export interface QueryHandler<T extends Query, R> {
  handle(query: T): Promise<R>;
}

import { User } from "./domain/User";

export class CreateUserHandler implements CommandHandler<CreateUserCommand> {
  constructor(private readonly repository: UserRepository) {}
  async handle(command: CreateUserCommand): Promise<void> {
    const user = new User(command.name, command.email);
    await this.repository.save(user);
  }
}

export class GetUserByEmailHandler implements QueryHandler<GetUserByEmailQuery, User | null> {
  constructor(private readonly repository: UserRepository) {}
  async handle(query: GetUserByEmailQuery): Promise<User | null> {
    return await this.repository.findByEmail(query.email);
  }
}

// --- Bus Implementation ---
class CommandBus {
  private handlers = new Map<string, CommandHandler<any>>();

  register<T extends Command>(commandType: string, handler: CommandHandler<T>) {
    this.handlers.set(commandType, handler);
  }

  async execute<T extends Command>(command: T): Promise<void> {
    const handler = this.handlers.get(command.type);
    if (!handler) throw new Error(`No handler registered for ${command.type}`);
    await handler.handle(command);
  }
}

class QueryBus {
  private handlers = new Map<string, QueryHandler<any, any>>();

  register<T extends Query, R>(queryType: string, handler: QueryHandler<T, R>) {
    this.handlers.set(queryType, handler);
  }

  async execute<T extends Query, R>(query: T): Promise<R> {
    const handler = this.handlers.get(query.type);
    if (!handler) throw new Error(`No handler registered for ${query.type}`);
    return await handler.handle(query);
  }
}

// --- Repository ---
interface UserRepository {
  save(user: User): Promise<void>;
  findByEmail(email: string): Promise<User | null>;
}

class InMemoryUserRepository implements UserRepository {
  private users: User[] = [];
  async save(user: User): Promise<void> {
    this.users.push(user);
  }
  async findByEmail(email: string): Promise<User | null> {
    return this.users.find((u) => u.email === email) || null;
  }
}

// --- Domain ---
export class User {
  constructor(public readonly name: string, public readonly email: string) {}
}

// --- Application Setup ---
const repository = new InMemoryUserRepository();
const commandBus = new CommandBus();
const queryBus = new QueryBus();

commandBus.register("CREATE_USER", new CreateUserHandler(repository));
queryBus.register("GET_USER_BY_EMAIL", new GetUserByEmailHandler(repository));

// --- Example Usage ---
(async () => {
  const createUserCommand = new CreateUserCommand("John Doe", "john@example.com");
  await commandBus.execute(createUserCommand);

  const query = new GetUserByEmailQuery("john@example.com");
  const user = await queryBus.execute(query);
  console.log(user); // Output: User { name: 'John Doe', email: 'john@example.com' }
})();
