# Session 4 — Notification System LLD

## Learning approach

The goal is to learn LLD through evolving requirements rather than memorizing patterns.

Flow:

Problem
→ First thought
→ Requirement change
→ Design problem
→ Improve design
→ Discover concept/pattern

---

## 1. Initial requirement

Requirement:

> When an event occurs, send a notification to a user.

Initially, only email notification was required.

First design:

```ts
class Notification {
    send(emailId: string, message: string) {
        // send email
    }
}
```

### Concept: Start Simple

Do not introduce managers, factories, repositories, or patterns without a requirement that justifies them.

---

# 2. Email + SMS

New requirement:

> We also need SMS notifications.

Adding `sendSMS()` to the existing class would require modifying existing code.

This led to the **Open/Closed Principle**.

### Concept: Open/Closed Principle (OCP)

Software entities should be:

- open for extension
- closed for modification

Instead of:

```ts
class Notification {
    sendEmail() {}
    sendSMS() {}
}
```

we created a common contract.

---

# 3. Notification abstraction

```ts
interface Notification {
    send(message: string): void;
}
```

Concrete implementations:

```ts
class EmailNotification implements Notification {
    send(message: string): void {
        // send email
    }
}

class SmsNotification implements Notification {
    send(message: string): void {
        // send SMS
    }
}
```

### Concept: Abstraction

`Notification` defines **what** every notification must do:

```text
send()
```

It does not define **how** email/SMS is actually sent.

### Concept: Polymorphism

Different concrete objects can be used through the same interface:

```ts
function notify(
    notification: Notification,
    message: string
) {
    notification.send(message);
}
```

The caller doesn't need to care whether the implementation is email or SMS.

---

# 4. Object creation problem

The application should not have to repeatedly do:

```ts
new EmailNotification()
new SmsNotification()
```

We introduced a Factory.

## NotificationFactory

```ts
class NotificationFactory {
    static create(type: NotificationType): Notification {
        if (type === NotificationType.EMAIL) {
            return new EmailNotification();
        }

        if (type === NotificationType.SMS) {
            return new SmsNotification();
        }

        throw new Error("Unsupported notification type");
    }
}
```

### Concept: Factory Pattern

Factory responsibility:

> Create and return the appropriate object.

The caller asks:

```ts
const notification =
    NotificationFactory.create("email");
```

The caller does not need to know:

```ts
new EmailNotification()
```

The Factory owns the object-creation decision.

---

# 5. Factory and OCP

We noticed a limitation.

Adding WhatsApp initially required modifying the Factory:

```ts
if (type === "whatsapp") {
    return new WhatsAppNotification();
}
```

We considered using a map:

```ts
const creators = {
    email: () => new EmailNotification(),
    sms: () => new SmsNotification()
};
```

But the map would still need modification when adding WhatsApp.

So we evolved toward registration.

```ts
class NotificationFactory {
    register(
        type: string,
        creator: () => Notification
    ) {
        // store creator
    }

    create(type: string): Notification {
        // use registered creator
    }
}
```

Startup can register implementations:

```ts
factory.register(
    "email",
    () => new EmailNotification()
);

factory.register(
    "sms",
    () => new SmsNotification()
);

factory.register(
    "whatsapp",
    () => new WhatsAppNotification()
);
```

Now adding a new notification can happen through registration without changing the Factory implementation.

---

# 6. Application startup

We decided the application should create/configure the Factory once during startup.

Conceptually:

```text
Application Startup
        |
        v
NotificationFactory
        |
        +-- register Email
        +-- register SMS
        +-- register WhatsApp
        |
        v
Factory passed to required components
```

We explicitly rejected making Email/SMS implementations Singleton just because we wanted one Factory.

### Concept: Singleton vs Dependency Injection

Singleton:

```text
Anyone
  |
  v
getInstance()
  |
  v
same global instance
```

Dependency Injection:

```text
Application Startup
        |
        v
create object
        |
        v
pass object to dependent class
```

We preferred dependency injection because the system may eventually need multiple differently configured instances/providers.

---

# 7. Dependency Injection

Example:

```ts
class OrderService {
    constructor(
        private factory: NotificationFactory
    ) {}
}
```

The important idea is:

> A class receives its dependency from outside instead of constructing it internally.

This makes dependencies explicit and makes testing easier.

---

# 8. Composition Root

We identified that application startup is the appropriate place to connect concrete implementations.

Concept:

> The Composition Root is where the application's objects and dependencies are wired together.

For example:

```ts
const factory = new NotificationFactory();

factory.register(
    "email",
    () => new EmailNotification()
);

factory.register(
    "sms",
    () => new SmsNotification()
);
```

Business code should not need to construct concrete notification implementations directly.

---

# 9. Dependency Inversion Principle (DIP)

We discussed DIP carefully.

The key idea:

> High-level business logic should depend on abstractions rather than directly depending on low-level implementation details.

Example:

```text
OrderService
     |
     v
Notification abstraction
     ^
     |
+----+----------------+
|                     |
Email              SMS
```

The business layer does not need to directly depend on:

```ts
EmailNotification
SmsNotification
```

The Composition Root can still know about concrete implementations.

### Important clarification

DIP does **not** mean:

> No part of the application may ever know about concrete classes.

Somewhere the concrete objects must be created.

The Composition Root is an appropriate place for that knowledge.

Rule to remember:

> Business logic depends on abstractions; the Composition Root connects abstractions to concrete implementations.

---

# 10. Important correction about OrderService

We questioned this design:

```ts
class OrderService {
    constructor(
        private factory: NotificationFactory
    ) {}

    notify() {
        const notification = this.factory.create("email");
        notification.send();
    }
}
```

This is not automatically a better design.

Why?

`OrderService` still knows:

```text
NotificationFactory
"email"
```

It no longer creates `EmailNotification` directly, but it still knows about notification creation.

The better question is:

> What does OrderService actually need from the notification system?

It needs to **send a notification**, not necessarily to **create notification objects**.

We considered a higher-level abstraction:

```ts
interface NotificationService {
    notify(
        type: NotificationType,
        message: string
    ): void;
}
```

Then:

```ts
class OrderService {
    constructor(
        private notificationService: NotificationService
    ) {}

    notify() {
        this.notificationService.notify(
            NotificationType.EMAIL,
            "Order placed"
        );
    }
}
```

The implementation could internally use the Factory:

```text
OrderService
     |
     v
NotificationService
     |
     v
NotificationFactory
     |
     v
EmailNotification
     |
     v
send()
```

However, we deliberately did **not** finalize this extra abstraction.

### Lesson

> Do not introduce an abstraction merely because it seems architecturally "clean."

It needs a meaningful responsibility.

---

# Final concepts discovered

During this session we naturally encountered:

1. **Responsibility**
   - Each class should have a clear reason to exist.

2. **Abstraction**
   - `Notification` defines the common contract.

3. **Polymorphism**
   - Email/SMS can be used through `Notification`.

4. **Open/Closed Principle**
   - New notification types should be addable without constantly modifying stable code.

5. **Factory Pattern**
   - Centralizes object creation.

6. **Dependency Injection**
   - Dependencies are supplied from outside.

7. **Singleton**
   - Considered and rejected as the default solution because multiple differently configured instances may be required.

8. **Composition Root**
   - Application startup wires concrete implementations together.

9. **Dependency Inversion Principle**
   - Business logic should depend on abstractions rather than concrete implementation details.

---

# Core takeaway

The most important lesson from this session:

> Don't start with a pattern. Start with the requirement.

We went from:

```text
Notification.send()
```

to:

```text
Notification interface
        ↓
Email / SMS
        ↓
Factory
        ↓
Registration
        ↓
Dependency Injection
        ↓
Composition Root
        ↓
DIP
```

Each design concept appeared because a new requirement exposed a problem in the previous design.

That is the approach we should continue using for future LLD problems.
