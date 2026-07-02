# Structs and Classes

## Structs

Plain data containers that live on the stack:

```kestrel
struct Point {
    float64 x
    float64 y
}

Point p = Point { x: 1.0, y: 2.0 }
printf("{p.x}, {p.y}\n")
```

## Classes

Classes support inheritance and live on the heap (managed by NOVA):

```kestrel
class Animal {
    pub str name

    virtual func speak() -> void {
        println("...")
    }
}

class Dog : Animal {
    override func speak() -> void {
        println("Woof! I'm {this.name}")
    }
}
```

Use `parent` to call the parent class method:

```kestrel
class Puppy : Dog {
    override func speak() -> void {
        parent.speak()
        println("*wags tail*")
    }
}
```

## Abstract classes

```kestrel
class Shape {
    abstract func area() -> float64

    virtual func describe() -> str {
        return "a shape"
    }
}

class Circle : Shape {
    pub float64 radius

    override func area() -> float64 {
        return 3.14159 * this.radius * this.radius
    }
}
```

## Visibility

Fields and methods are **private by default**. Use `pub` or `protected`:

```kestrel
class BankAccount {
    float64 balance                 // private (default)

    pub func deposit(float64 amount) -> void {
        balance += amount
    }

    pub func get_balance() -> float64 {
        return balance
    }

    protected func audit_log() -> void {
        // only this class and subclasses
    }
}
```

## Traits

```kestrel
trait Printable {
    func print() -> void
}

impl Printable for Point {
    func print() -> void {
        printf("({this.x}, {this.y})\n")
    }
}
```

## Adding traits to existing types

```kestrel
impl Printable for Circle {
    func print() -> void {
        printf("Circle r={this.radius}\n")
    }
}
```
