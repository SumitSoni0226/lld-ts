# Coffee Machine — LLD Learning Session

## 1. Objective

Build a Coffee Machine using TypeScript while learning **Low-Level Design through design evolution**.

The goal is not to jump directly to a perfect architecture.

Instead:

```text
Requirement
    ↓
Initial thought
    ↓
Design
    ↓
New requirement
    ↓
Something becomes difficult
    ↓
Understand why
    ↓
Improve design
```

---

# 2. Initial Idea

The Coffee Machine's primary responsibility is to make coffee.

Initial thought:

```ts
class CoffeeMachine {
    makeCoffee() {}
}
```

But a real coffee machine can make different types of coffee:

```text
Espresso
Cappuccino
Latte
...
```

Initially, the thought was to create a `Coffee` interface with different implementations.

However, we realized:

> **Making coffee is the responsibility of CoffeeMachine, while different coffee types should describe how a particular coffee is prepared.**

Therefore, the abstraction should be around the **recipe**, not the coffee itself.

---

# 3. CoffeeRecipe

A recipe should provide:

1. Ingredients required
2. Steps required to prepare the coffee

```ts
interface CoffeeRecipe {
    getIngredients(): IngredientRequirement[];
    getSteps(): string[];
}
```

Example:

```ts
class CappuccinoRecipe implements CoffeeRecipe {
    getIngredients(): IngredientRequirement[] {
        return [
            {
                ingredient: "coffeeBeans",
                quantity: 10,
                unit: "grams"
            },
            {
                ingredient: "water",
                quantity: 30,
                unit: "ml"
            },
            {
                ingredient: "milk",
                quantity: 100,
                unit: "ml"
            }
        ];
    }

    getSteps(): string[] {
        return [
            "Grind coffee beans",
            "Extract coffee",
            "Steam milk",
            "Combine coffee and milk"
        ];
    }
}
```

---

# 4. IngredientRequirement

A recipe needs to describe what ingredients it requires.

```ts
type IngredientRequirement = {
    ingredient: string;
    quantity: number;
    unit: string;
};
```

Example:

```ts
{
    ingredient: "coffeeBeans",
    quantity: 10,
    unit: "grams"
}
```

---

# 5. Inventory

Before making coffee, we need to know whether the required ingredients are available.

Example:

```text
Coffee beans → 100 grams
Water        → 500 ml
Milk         → 200 ml
Sugar        → 50 grams
```

Initially, the thought was to create a service that checks ingredient availability.

This led to `InventoryManager`.

---

# 6. InventoryManager

`InventoryManager` manages **inventory rules**.

Its responsibilities:

```text
Are the required ingredients available?
        ↓
Consume the ingredients
```

Initial design:

```ts
class InventoryManager {
    constructor(
        private repository: InventoryRepository
    ) {}

    areIngredientsAvailable(
        ingredients: IngredientRequirement[]
    ): boolean {
        // ...
    }

    consumeIngredients(
        ingredients: IngredientRequirement[]
    ): void {
        // ...
    }
}
```

---

# 7. Why InventoryManager?

We could have allowed `CoffeeMachine` to directly access the database.

But then `CoffeeMachine` would need to know:

```text
How inventory is stored
How to query inventory
How to update inventory
```

That would mix responsibilities.

Instead:

```text
CoffeeMachine
      ↓
InventoryManager
      ↓
InventoryRepository
      ↓
Actual storage
```

The distinction:

> **InventoryManager handles inventory rules.**

> **InventoryRepository handles inventory persistence/storage.**

---

# 8. InventoryRepository

We realized inventory storage can change.

Initially:

```text
In-memory
```

Later:

```text
PostgreSQL
MongoDB
Cassandra
...
```

We don't want `InventoryManager` to change whenever the database changes.

Therefore we created an interface:

```ts
interface InventoryRepository {
    getIngredient(name: string): Ingredient | null;
    updateIngredient(ingredient: Ingredient): void;
}
```

The manager depends on this contract instead of a concrete database.

---

# 9. Ingredient

We created an `Ingredient` type to represent the current inventory.

```ts
type Ingredient = {
    ingredient: string;
    quantity: number;
    unit: string;
};
```

Conceptually:

```text
IngredientRequirement
→ "I need 10 grams of coffee beans"

Ingredient
→ "I currently have 100 grams of coffee beans"
```

They currently have the same structure but represent different concepts.

---

# 10. In-Memory Inventory Repository

For the first implementation, we decided to store inventory using a `Map`.

Conceptually:

```text
"coffeeBeans"
      ↓
{
    ingredient: "coffeeBeans",
    quantity: 100,
    unit: "grams"
}
```

Implementation:

```ts
class InMemoryInventoryRepository
    implements InventoryRepository {

    private inventory = new Map<string, Ingredient>();

    constructor(ingredients: Ingredient[]) {
        for (const ingredient of ingredients) {
            this.inventory.set(
                ingredient.ingredient,
                ingredient
            );
        }
    }

    getIngredient(name: string): Ingredient | null {
        return this.inventory.get(name) ?? null;
    }

    updateIngredient(ingredient: Ingredient): void {
        this.inventory.set(
            ingredient.ingredient,
            ingredient
        );
    }
}
```

---

# 11. Initial Inventory

The repository receives initial inventory through its constructor.

```ts
const repository = new InMemoryInventoryRepository([
    {
        ingredient: "coffeeBeans",
        quantity: 100,
        unit: "grams"
    },
    {
        ingredient: "water",
        quantity: 500,
        unit: "ml"
    },
    {
        ingredient: "milk",
        quantity: 200,
        unit: "ml"
    }
]);
```

This gives us:

```text
InMemoryInventoryRepository
        ↓
      Map
        ↓
┌──────────────────────────────┐
│ coffeeBeans → 100 grams      │
│ water       → 500 ml         │
│ milk        → 200 ml         │
└──────────────────────────────┘
```

---

# 12. Checking Ingredient Availability

Suppose a recipe requires:

```ts
[
    {
        ingredient: "coffeeBeans",
        quantity: 10,
        unit: "grams"
    },
    {
        ingredient: "water",
        quantity: 30,
        unit: "ml"
    }
]
```

For every required ingredient, `InventoryManager` gets the current inventory.

For one iteration:

```ts
const inventory = this.repository.getIngredient(
    ingredient.ingredient
);

if (
    !inventory ||
    inventory.quantity < ingredient.quantity
) {
    return false;
}
```

Complete implementation:

```ts
areIngredientsAvailable(
    ingredients: IngredientRequirement[]
): boolean {
    for (const ingredient of ingredients) {
        const inventory = this.repository.getIngredient(
            ingredient.ingredient
        );

        if (
            !inventory ||
            inventory.quantity < ingredient.quantity
        ) {
            return false;
        }
    }

    return true;
}
```

Logic:

```text
For every required ingredient:

    Get current inventory

    If ingredient doesn't exist
        → false

    If available quantity < required quantity
        → false

If everything is available
    → true
```

---

# 13. Consuming Ingredients

After coffee is made, inventory needs to be reduced.

Example:

```text
Before:

Coffee beans → 100g
Water        → 500ml

Consumed:

Coffee beans → 10g
Water        → 30ml

After:

Coffee beans → 90g
Water        → 470ml
```

The logic:

```text
1. Get current ingredient
2. Reduce quantity
3. Update repository
```

Implementation:

```ts
consumeIngredients(
    ingredients: IngredientRequirement[]
): void {
    for (const ingredient of ingredients) {
        const inventory = this.repository.getIngredient(
            ingredient.ingredient
        );

        inventory.quantity =
            inventory.quantity - ingredient.quantity;

        this.repository.updateIngredient(inventory);
    }
}
```

---

# 14. CoffeeMaker

We identified another responsibility:

> `CoffeeMachine` should coordinate the process, while something else should actually make the coffee.

This led to:

```text
CoffeeMachine
    ↓
coordinates
    ↓
CoffeeMaker
    ↓
actually prepares coffee
```

We considered making `CoffeeMaker` an interface because there could be different implementations.

For example:

```text
CoffeeMaker
    ↑
    ├── NormalCoffeeMaker
    └── PremiumCoffeeMaker
```

Contract:

```ts
interface CoffeeMaker {
    make(recipe: CoffeeRecipe): CoffeeMakingResult;
}
```

---

# 15. CoffeeMakingResult

Initially, we wanted the maker to return:

```text
success
reason
ingredients consumed
```

We considered:

```ts
type CoffeeMakingResult = {
    success: boolean;
    reason?: string;
    ingredientsConsumed: Ingredient[];
};
```

The reason for `ingredientsConsumed` was a possible failure scenario:

```text
Step 1 → Grind beans       ✅
Step 2 → Extract coffee    ✅
Step 3 → Steam milk        ❌
```

Some ingredients might already have been consumed before failure.

However, this introduced additional complexity.

We realized that this was becoming difficult to reason about because we were trying to solve multiple design problems at once.

So we intentionally **simplified the current version**.

For the first version:

```ts
type CoffeeMakingResult = {
    success: boolean;
    reason?: string;
};
```

We will introduce partial consumption later as a **new requirement**.

This is important to our learning approach:

> Don't design for hypothetical complexity before the requirement exists.

---

# 16. Restaurant Analogy

A useful mental model was a restaurant with multiple kitchens.

```text
Restaurant
    │
    ├── Reception
    │
    ├── Veg Kitchen
    │
    └── Non-Veg Kitchen
```

A customer orders food.

The reception coordinates the order.

The kitchen actually prepares the food.

Mapping:

```text
Restaurant
    → CoffeeMachine

Reception
    → CoffeeMachine coordination

Kitchen
    → CoffeeMaker

Different kitchens
    → Different CoffeeMaker implementations
```

Therefore:

> **CoffeeMachine coordinates. CoffeeMaker actually makes the coffee.**

---

# 17. Current Design

At the end of this session, our design is:

```text
                         CoffeeMachine
                              │
             ┌────────────────┼────────────────┐
             ↓                ↓                ↓
       CoffeeRecipe    InventoryManager    CoffeeMaker
                              │
                              ↓
                    InventoryRepository
                              │
                              ↓
              InMemoryInventoryRepository
                              │
                              ↓
                             Map
```

Responsibilities:

```text
CoffeeMachine
→ Coordinates the complete process

CoffeeRecipe
→ Provides ingredients and preparation steps

InventoryManager
→ Applies inventory rules

InventoryRepository
→ Provides storage abstraction

InMemoryInventoryRepository
→ Stores inventory in memory

CoffeeMaker
→ Actually makes coffee
```

---

# 18. Current Code

The pieces we have designed so far:

```ts
type Ingredient = {
    ingredient: string;
    quantity: number;
    unit: string;
};

type IngredientRequirement = {
    ingredient: string;
    quantity: number;
    unit: string;
};

interface CoffeeRecipe {
    getIngredients(): IngredientRequirement[];
    getSteps(): string[];
}

interface InventoryRepository {
    getIngredient(name: string): Ingredient | null;
    updateIngredient(ingredient: Ingredient): void;
}

class InMemoryInventoryRepository
    implements InventoryRepository {

    private inventory = new Map<string, Ingredient>();

    constructor(ingredients: Ingredient[]) {
        for (const ingredient of ingredients) {
            this.inventory.set(
                ingredient.ingredient,
                ingredient
            );
        }
    }

    getIngredient(name: string): Ingredient | null {
        return this.inventory.get(name) ?? null;
    }

    updateIngredient(ingredient: Ingredient): void {
        this.inventory.set(
            ingredient.ingredient,
            ingredient
        );
    }
}

class InventoryManager {
    constructor(
        private repository: InventoryRepository
    ) {}

    areIngredientsAvailable(
        ingredients: IngredientRequirement[]
    ): boolean {
        for (const ingredient of ingredients) {
            const inventory = this.repository.getIngredient(
                ingredient.ingredient
            );

            if (
                !inventory ||
                inventory.quantity < ingredient.quantity
            ) {
                return false;
            }
        }

        return true;
    }

    consumeIngredients(
        ingredients: IngredientRequirement[]
    ): void {
        for (const ingredient of ingredients) {
            const inventory = this.repository.getIngredient(
                ingredient.ingredient
            );

            inventory.quantity =
                inventory.quantity - ingredient.quantity;

            this.repository.updateIngredient(inventory);
        }
    }
}

type CoffeeMakingResult = {
    success: boolean;
    reason?: string;
};

interface CoffeeMaker {
    make(recipe: CoffeeRecipe): CoffeeMakingResult;
}
```

---

# 19. What We Learned

### 1. Start with responsibilities

Instead of immediately asking:

> Which design pattern should I use?

Ask:

> What responsibility does this object have?

---

### 2. Don't make one class do everything

Instead of:

```text
CoffeeMachine
    ↓
does everything
```

we separated:

```text
CoffeeMachine
CoffeeRecipe
InventoryManager
InventoryRepository
CoffeeMaker
```

---

### 3. Interface when implementations can vary

We introduced:

```ts
interface InventoryRepository
```

because storage can vary:

```text
InMemory
PostgreSQL
MongoDB
Cassandra
```

Similarly, `CoffeeMaker` can have multiple implementations.

---

### 4. Manager vs Repository

Important distinction:

```text
InventoryManager
→ inventory business rules

InventoryRepository
→ persistence/storage
```

---

### 5. Don't over-engineer

We considered additional managers but rejected them because they didn't have meaningful responsibilities.

For example:

```text
CoffeeMakerManager
CoffeeMachineManager
```

were unnecessary.

---

### 6. Let complexity emerge

We initially tried to solve partial ingredient consumption during a failed coffee-making process.

That made the problem difficult to follow.

Instead, we simplified the first version.

Later, we can introduce the requirement:

> "If coffee fails halfway, tell me exactly what was consumed."

Then we can evolve the design.

This is the core of this learning journey.

---

# 20. Things Still Not Fully Understood

These concepts have not yet been formally learned:

```text
Abstraction
Polymorphism
Dependency Inversion Principle
SOLID principles
Design Patterns
```

The goal is to encounter the need for them naturally during the exercises rather than memorize definitions first.

---

# 21. Session Status

### Completed

* Initial CoffeeMachine idea
* CoffeeRecipe
* IngredientRequirement
* Ingredient
* InventoryManager
* InventoryRepository
* InMemoryInventoryRepository
* Inventory availability checking
* Ingredient consumption
* CoffeeMaker responsibility
* Basic CoffeeMakingResult

### Paused At

```text
CoffeeMaker
    ↓
make(recipe)
    ↓
actually prepare coffee
```

We will continue from here in the next session.

---

# 22. Next Session

The next step is to implement:

```ts
class NormalCoffeeMaker implements CoffeeMaker
```

using the simplified requirement:

```ts
type CoffeeMakingResult = {
    success: boolean;
    reason?: string;
};
```

We will **not** introduce partial consumption yet.

That will come later as a requirement change.

---

# Core Principle

> **Don't memorize the solution. Understand why the design evolved into the solution.**

And more importantly:

> **Don't solve tomorrow's requirements today. Let the design evolve when the requirements actually change.**
