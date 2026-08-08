# First Design Exploration — User & Permissions

## Why We Did This

This was my first LLD exploration.

The goal was **not** to learn design patterns or memorize SOLID definitions.

The goal was to understand how a design evolves when requirements change.

---

# 1. Starting Point

We started with a simple requirement:

> Design a `User`.

My initial thought was:

```ts
class User {
    name;
    age;

    updateUser(key, value) {
        this[key] = value;
    }
}
```

The basic idea was:

```text
User
 ├── name
 ├── age
 ├── other properties
 │
 ├── updateUser()
 └── getUser()
```

At this point, the design was very simple.

---

# 2. Requirement: Some Properties Should Not Be Modifiable

We introduced properties like:

```text
name
age
email
password
role
isAdmin
```

But not every property should be freely modifiable.

For example:

```text
name     → can modify
age      → can modify
isAdmin  → cannot modify
role     → cannot modify
```

My first idea was to maintain something like:

```ts
notAllowedToModify = ["isAdmin", "role"];
```

and check it before updating.

---

# 3. Requirement: Different Roles Have Different Permissions

Then the requirement changed.

Different roles can modify different properties.

For example:

```text
User
 └── name, age

Admin
 └── name, age, role, accountStatus

System
 └── password
```

Instead of maintaining one `notAllowedToModify` list, I proposed mapping roles to the properties they can modify:

```text
User   → [name, age]
Admin  → [name, age, role, accountStatus]
System → [password]
```

This was the first point where we started seeing that **permission is its own area of responsibility**.

---

# 4. Requirement: Permissions Come From a Database

Then we considered:

> What if permissions are stored in a database?

I realized that `User` should not contain:

```text
database connection logic
database fetching logic
permission fetching logic
```

Otherwise, changing:

```text
PostgreSQL → MongoDB
```

would require changing the `User` class.

So we started separating responsibilities.

Conceptually:

```text
User
 ↓
Permission logic
 ↓
Database
```

The important realization was:

> A `User` should not know how permissions are stored.

---

# 5. Database Abstraction

We then considered multiple databases:

```text
PostgreSQL
MongoDB
Cassandra
```

We didn't want application code to depend directly on PostgreSQL.

Instead, we created a contract:

```ts
interface Database {
    save(data: unknown): void;
    find(id: string): unknown;
    update(id: string, data: unknown): void;
    delete(id: string): void;
}
```

Then different databases implement the contract:

```ts
class PostgresDatabase implements Database {
    save(data: unknown) {
        // PostgreSQL-specific implementation
    }

    find(id: string) {
        // PostgreSQL-specific implementation
    }

    update(id: string, data: unknown) {
        // PostgreSQL-specific implementation
    }

    delete(id: string) {
        // PostgreSQL-specific implementation
    }
}
```

And:

```ts
class MongoDatabase implements Database {
    save(data: unknown) {
        // MongoDB-specific implementation
    }

    find(id: string) {
        // MongoDB-specific implementation
    }

    update(id: string, data: unknown) {
        // MongoDB-specific implementation
    }

    delete(id: string) {
        // MongoDB-specific implementation
    }
}
```

Now a new database can be added:

```ts
class CassandraDatabase implements Database {
    // Cassandra-specific implementation
}
```

without modifying the existing application code.

---

# 6. What Does `implements` Mean?

We learned:

```ts
class MongoDatabase implements Database
```

means that `MongoDatabase` promises to follow the `Database` contract.

If the interface says:

```ts
interface Database {
    save(): void;
    find(): void;
}
```

then the implementing class must provide those methods.

TypeScript checks this for us.

So:

```text
interface
    ↓
defines the contract

implements
    ↓
class promises to follow it

TypeScript
    ↓
checks the promise
```

---

# 7. Abstraction

We then learned about abstraction.

Example:

```ts
interface Payment {
    pay(amount: number): void;
}
```

The caller only needs to know:

```ts
payment.pay(500);
```

It doesn't need to know whether the implementation uses:

```text
UPI
Credit Card
PayPal
Net Banking
```

The implementation details are hidden.

### Mental model

> **Abstraction means exposing what is needed while hiding unnecessary implementation details.**

An interface is one TypeScript mechanism that can help us achieve abstraction.

They are related, but they are not the same thing.

---

# 8. Polymorphism

We saw:

```ts
interface Payment {
    pay(amount: number): void;
}

class UPI implements Payment {
    pay(amount: number) {
        console.log("UPI payment");
    }
}

class CreditCard implements Payment {
    pay(amount: number) {
        console.log("Credit Card payment");
    }
}
```

Then:

```ts
function makePayment(payment: Payment) {
    payment.pay(500);
}
```

We can pass:

```ts
makePayment(new UPI());
```

or:

```ts
makePayment(new CreditCard());
```

The function stays the same.

But the implementation of `pay()` changes depending on the object.

### Mental model

> **Same method call, different implementation depending on the actual object.**

---

# 9. Dependency Inversion

We compared:

```ts
class PermissionService {
    constructor(private db: PostgresDatabase) {}
}
```

with:

```ts
class PermissionService {
    constructor(private db: Database) {}
}
```

The second design is more flexible.

Why?

Because `PermissionService` doesn't actually need PostgreSQL.

It needs something that provides the database operations defined by `Database`.

Therefore:

```text
PermissionService
       ↓
    Database
       ↑
   ┌───┴────┐
   │        │
Postgres   Mongo
```

If we add Cassandra:

```text
PermissionService
       ↓
    Database
       ↑
   ┌───┼────────┐
   │   │        │
Postgres Mongo Cassandra
```

`PermissionService` doesn't need to change.

### Mental model

> **High-level business logic should not be tightly coupled to low-level implementation details. Both can work through an abstraction.**

We learned this from the problem rather than memorizing the formal SOLID definition.

---

# 10. An Important Design Lesson

At one point, we introduced a `PermissionService`.

I questioned why we needed that class.

That was a good observation.

We realized that we should not create classes just because names like:

```text
UserService
PermissionService
PaymentService
```

sound natural.

Instead, ask:

> **What responsibility does this class have?**

If we cannot identify a meaningful responsibility, we probably don't need the class yet.

### Important rule

> **Design from responsibilities, not from class names.**

Don't start with:

```text
"I need a service."
```

Start with:

```text
"What needs to happen?"
"Who should be responsible for it?"
```

---

# 11. Current Mental Model

Our permission design eventually became:

```text
User
 ↓
Permission
 ↑
 ├── RoleBasedPermission
 └── ExternalPermission
```

And permission implementations may use:

```text
Permission Implementation
          ↓
       Database
       ↑      ↑
   Postgres  Mongo
```

We deliberately did **not** add unnecessary classes unless a requirement gave us a reason.

---

# 12. Concepts Learned

| Concept                      | What I Understand                                          |
| ---------------------------- | ---------------------------------------------------------- |
| Interface                    | A contract that implementations must follow                |
| `implements`                 | A class promises to follow an interface                    |
| Abstraction                  | Hide implementation details and expose what is needed      |
| Polymorphism                 | Same method call can execute different implementations     |
| Dependency Inversion         | Depend on abstractions instead of concrete implementations |
| Separation of Responsibility | Keep unrelated responsibilities separate                   |
| Design from Responsibilities | Don't create classes without a reason                      |

---

# 13. The Most Important Takeaway

The most important thing I learned today is **not** SOLID or interfaces.

It is this:

```text
Don't start with:
"What classes should I create?"

Start with:
"What problem am I solving?"

Then:
"What changes might happen?"

Then:
"What is becoming difficult?"

Then:
"Who should be responsible for this?"

Then:
"Can I make the design less tightly coupled?"
```

The design should **evolve from the problem**.

---

# 14. Questions I Should Be Able to Answer

Before moving forward, I should be able to explain these in my own words:

### Interface

* What problem does an interface solve?
* Why do different implementations follow the same interface?

### Abstraction

* What implementation details are being hidden?
* Why is hiding them useful?

### Polymorphism

* Why can the same method call behave differently?

### Dependency Inversion

* Why is depending on `Database` better than depending directly on `PostgresDatabase`?

### Design

* Why shouldn't we create a class unless it has a clear responsibility?
* Why shouldn't we put database logic inside `User`?

---

# 15. Next Step

Start a real LLD problem.

The next problem should be small enough that I can first attempt the design myself.

We should follow:

```text
Problem
   ↓
My understanding
   ↓
My first design
   ↓
Code
   ↓
Requirement changes
   ↓
What breaks?
   ↓
Improve design
   ↓
Discover principles/patterns
   ↓
Final implementation
```

**The goal is to develop design skills, not memorize solutions.**
