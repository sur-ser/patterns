// --- Saga Pattern Implementation ---
// Sagas coordinate and manage transactions across multiple services or steps.
// They listen for events and execute a series of commands or compensations in case of failures.

// --- Events ---
export interface Event {
  readonly type: string;
}

export class UserCreatedEvent implements Event {
  readonly type = "USER_CREATED";
  constructor(public readonly userId: string) {}
}

export class UserCreationFailedEvent implements Event {
  readonly type = "USER_CREATION_FAILED";
  constructor(public readonly userId: string, public readonly reason: string) {}
}

// --- Commands ---
export interface Command {
  readonly type: string;
}

export class CreateUserCommand implements Command {
  readonly type = "CREATE_USER";
  constructor(public readonly name: string, public readonly email: string) {}
}

export class RollbackUserCreationCommand implements Command {
  readonly type = "ROLLBACK_USER_CREATION";
  constructor(public readonly userId: string) {}
}

// --- Saga ---
class UserCreationSaga {
  private steps: { execute: Command; compensate?: Command }[] = [];

  addStep(execute: Command, compensate?: Command) {
    this.steps.push({ execute, compensate });
  }

  async run(commandBus: CommandBus): Promise<void> {
    const executedSteps: { execute: Command; compensate?: Command }[] = [];
    try {
      for (const step of this.steps) {
        await commandBus.execute(step.execute);
        executedSteps.push(step);
      }
    } catch (error) {
      console.error("Saga step failed, triggering compensation", error);
      for (const step of executedSteps.reverse()) {
        if (step.compensate) {
          await commandBus.execute(step.compensate);
        }
      }
    }
  }
}

// --- Command Handlers ---
export class CreateUserHandler implements CommandHandler<CreateUserCommand> {
  constructor(private readonly repository: UserRepository) {}
  async handle(command: CreateUserCommand): Promise<void> {
    console.log(`Creating user: ${command.name}, ${command.email}`);
    const user = new User(command.name, command.email);
    await this.repository.save(user);
  }
}

export class RollbackUserCreationHandler implements CommandHandler<RollbackUserCreationCommand> {
  constructor(private readonly repository: UserRepository) {}
  async handle(command: RollbackUserCreationCommand): Promise<void> {
    console.log(`Rolling back user creation for ID: ${command.userId}`);
    await this.repository.delete(command.userId);
  }
}

// --- Repository Updates ---
class InMemoryUserRepository implements UserRepository {
  private users: Map<string, User> = new Map();

  async save(user: User): Promise<void> {
    this.users.set(user.email, user);
  }

  async delete(userId: string): Promise<void> {
    this.users.delete(userId);
  }
}

// --- Example Usage ---
(async () => {
  const repository = new InMemoryUserRepository();
  const commandBus = new CommandBus();

  // Register Command Handlers
  commandBus.register("CREATE_USER", new CreateUserHandler(repository));
  commandBus.register("ROLLBACK_USER_CREATION", new RollbackUserCreationHandler(repository));

  // Create Saga
  const saga = new UserCreationSaga();
  const userId = "user123";

  saga.addStep(new CreateUserCommand("John Doe", "john@example.com"), new RollbackUserCreationCommand(userId));

  // Run Saga
  await saga.run(commandBus);
})();
