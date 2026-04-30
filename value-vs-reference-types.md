# A Deep Dive into Value vs. Reference Types

Irrespective of which programming language you call home, I'm sure you've heard about **Value Types** and **Reference Types**. Today, we’re going to deep dive into understanding what they actually are, not just the surface behaviour, but the why behind it.

## The First Thing That Comes to Mind

The first difference between **Value Types** and **Reference Types** that comes straight to mind is what happens when you copy them or assign them to a new variables?

When you copy a **Value Type**, the data is physically duplicated from one variable to another. Both of them will have their own independent copy of the value, meaning each one can do whatever it wants without affecting the other.

With a **Reference Type**, it's a different story. When you copy a **Reference Type** you only copy the address (the "map"), where the data is actualy stored, between the two variables. So when you make a change through one of them, that change shows up in the other one too. End of the day they only have one data to work with.

Let's see this in code:

```swift
// Value Type Example (Struct)
struct Wallet {
    var cash: Int
}

var myWallet = Wallet(cash: 100)
var yourWallet = myWallet  // Data is COPIED
yourWallet.cash = 50

print(myWallet.cash)   // 100 - untouched!
print(yourWallet.cash) // 50

// Reference Type Example (Class)
class SharedAccount {
    var balance: Int = 100
}

let accountA = SharedAccount()
let accountB = accountA  // Only the REFERENCE is copied
accountB.balance = 50

print(accountA.balance) // 50 - wait, it changed!
print(accountB.balance) // 50
```

## Ok, But Why Does This Happen?

We know this behaviour, but have you ever stopped and wondered why this difference exists in the first place? Let's dive deeper.

In simple words, **Value Types** and **Reference Types** define how data is stored, copied, and shared in the memory.

**Value Types** are stored in the **Stack**, whereas **Reference Types** are stored in the **Heap**, and the address where they are stored is saved in the **Stack**.

Now if that's the case who decided that a class is a reference type and a struct is a value type? Can we make one act like the other?

The answer lies with the **Compiler**. The people who built your programming language wrote the logic inside the compiler that decides how each type gets stored in memory and how it gets copied. You're abstracted away from all of that. you just use the keywords they gave you (`class`, `struct`, `enum`) and the compiler handles the rest.

And if you wanted to make your own? Well, if you're building your own programming language, sure, go ahead. You get to define the rules. But in an existing language, you work within the boundaries it gives you.

## Enter the Warehouse: Stack and Heap

At this point you might be wondering. what even is a Stack or a Heap?

Let me explain. They are, at their core, just storage techniques. The machine itself doesn't know about any of this, all it sees is 1s and 0s. Then what are they? lets dive in.

Imagine your memory as a giant warehouse full of empty slots, where each slot can hold a box. If a box is present, that's a 1. If it's empty, that's a 0.

### The Stack

In the Stack, we take one specific row of the warehouse and put a strict rule on it:

- Boxes can only be added from the top
- Boxes can only be removed from the top (Last In, First Out)

Because of this rule, every box on the Stack has a fixed, known size. That means if you know where the top is, you can instantly find any box below it just by counting down. It's fast, predictable, and efficient.

The downside? The Stack has a limited size. If you try to store too much in it, you hit the famous Stack Overflow - yes, that's where the website got its name.

### The Heap

When your data is too big or unpredictable in size, the Stack just won't cut it. That's where the Heap comes in.

Think of the Heap as the wide open floor of the warehouse. We find a big enough empty space, dump all our data there, and then write down the address of that location on a small sticky note.

Where does that sticky note go? Back on the Stack, for easy access.

## Putting It All Together

Can you see it now?

**Value Types** live entirely on the **Stack**. When you copy them, you're just making a new box and placing it on top.

**Reference Types** live on the **Heap**, but the sticky note pointing to them lives on the **Stack**. When you "copy" a reference type, you're just copying the sticky note, not the giant pile of data it points to. That's why both variables end up seeing the same thing.

It's not magic. It's not an arbitrary rule. It's just the natural consequence of where the data lives.

## Ok, But Who Cleans Up the Heap?

Here's a question that naturally follows, if Reference Types keep piling up on the Heap, who's responsible for cleaning them up when we're done with them? lets dive into this.

Because if nobody does, your app just keeps consuming more and more memory until it crashes. That's called a memory leak and it's every developer's nightmare.

Different languages solve this differently, but let's look at the two most common approaches.

### ARC - Swift's Approach

Swift uses something called **Automatic Reference Counting**, or **ARC**. The idea is simple, every object on the **Heap** carries an invisible counter. Every time a new variable points to that object, the counter goes up. Every time a variable goes out of scope or stops pointing to it, the counter goes down. The moment that counter hits zero, Swift immediately frees the memory.

```swift
class Dog {
    let name: String
    init(_ name: String) { self.name = name }
    deinit { print("\(name) was deallocated") }
}

var dog1: Dog? = Dog("Rex")  // counter: 1
var dog2 = dog1              // counter: 2
dog1 = nil                   // counter: 1
dog2 = nil                   // counter: 0 -> "Rex was deallocated"
```

The beautiful thing about **ARC** is that it's immediate and predictable, you know exactly when memory gets freed.

The one thing to watch out for? If two objects hold a reference to each other, their counters never reach zero and they never get cleaned up. Swift gives us `weak` and `unowned` references to break these kinds of cycles.

### Garbage Collection - Java, C#, Go

Other languages take a different approach entirely. Instead of counting references, a **Garbage Collector** runs in the background, periodically looks at everything on the **Heap**, figures out what's no longer reachable from your program, and cleans it all up in one go.

The upside is you never have to think about retain cycles or memory management at all. The downside is those cleanup pauses, even small ones, can cause unexpected hiccups in performance-sensitive apps. Modern garbage collectors have gotten really good at minimising this, but it's still a tradeoff worth knowing about.

## Wait - Doesn't Copying Value Types Get Expensive?

Here's something that should be bugging you at this point.

If Value Types get fully copied every time you assign them - what happens when you have a large array with thousands of items? Does Swift really copy all of that data every single time?

If we copy all the the data every single time when pass it or assign to new variable, that would make Value Types completely impractical for collections. And yet, `Array`, `Dictionary` and `String` in Swift are all Value Types. So what's going on? lets see how this work

The answer is a clever trick called **Copy-on-Write**, or **CoW** for short.

The idea is: don't actually copy the data until someone tries to change it.

When you assign an array to a new variable, Swift doesn't immediately duplicate all that data. Instead, both variables quietly share the same data behind the scenes. The moment one of them tries to modify it only then does Swift make the actual copy.

```swift
var a = [1, 2, 3, 4, 5]
var b = a          // No copy yet - both share the same data

b.append(6)        // NOW the copy happens, right at this moment

print(a) // [1, 2, 3, 4, 5] - untouched
print(b) // [1, 2, 3, 4, 5, 6]
```

If you never mutate `b`, the copy never happens at all. You get the safety of Value Type behaviour without paying the cost of copying data you never needed to change.

That's the best of both worlds, and it's one of those things that seems like magic until you understand what's happening under the hood.

## The Full Picture

Let's bring it all together:

- **Value Types** live on the **Stack** - fast, predictable, automatically cleaned up when they go out of scope
- **Reference Types** live on the **Heap** - flexible, shared, but need memory management via **ARC** or a **Garbage Collector**
- Large **Value Types** like arrays use **Copy-on-Write** to avoid unnecessary copying until the moment it's actually needed

It's not a set of arbitrary rules to make your life difficult, it's a carefully designed system balancing speed, flexibility, and safety. Once you see it for what it is, the behaviour that used to seem confusing starts to make complete sense.

Next time a property changes and you didn't expect it to, you know exactly why. Someone copied the sticky note! :smile:
