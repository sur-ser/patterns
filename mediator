// --- Mediator Pattern Implementation ---
// The Mediator pattern facilitates communication between multiple objects (Colleagues) via a central Mediator.
// This promotes loose coupling by preventing objects from referring to each other directly.

// --- Mediator Interface ---
interface Mediator {
  notify(sender: object, event: string): void;
}

// --- Concrete Mediator ---
class UserMediator implements Mediator {
  private notificationService: NotificationService;

  constructor(notificationService: NotificationService) {
    this.notificationService = notificationService;
  }

  async notify(sender: object, event: string): Promise<void> {
    if (event === "UserCreated") {
      console.log("Mediator handling UserCreated event.");
      const user = sender as User;
      await this.notificationService.sendWelcomeEmail(user.email);
    } else if (event === "UserDeleted") {
      console.log("Mediator handling UserDeleted event.");
      const user = sender as User;
      await this.notificationService.sendAccountClosureEmail(user.email);
    }
  }
}

// --- Colleague Classes ---
class UserRepository {
  private users: Map<string, User> = new Map();
  private mediator: Mediator;

  constructor(mediator: Mediator) {
    this.mediator = mediator;
  }

  async save(user: User): Promise<void> {
    this.users.set(user.email, user);
    await this.mediator.notify(user, "UserCreated");
  }

  async deleteByEmail(email: string): Promise<void> {
    const user = this.users.get(email);
    if (user) {
      this.users.delete(email);
      await this.mediator.notify(user, "UserDeleted");
    }
  }
}

class NotificationService {
  async sendWelcomeEmail(email: string): Promise<void> {
    console.log(`Sending welcome email to ${email}`);
  }

  async sendAccountClosureEmail(email: string): Promise<void> {
    console.log(`Sending account closure email to ${email}`);
  }
}

// --- Domain ---
export class User {
  constructor(public readonly name: string, public readonly email: string) {}
}

// --- Example Usage ---
(async () => {
  const notificationService = new NotificationService();
  const mediator = new UserMediator(notificationService);
  const userRepository = new UserRepository(mediator);

  const user = new User("Jane Doe", "jane@example.com");
  await userRepository.save(user); // Triggers "UserCreated" event

  await userRepository.deleteByEmail("jane@example.com"); // Triggers "UserDeleted" event
})();
