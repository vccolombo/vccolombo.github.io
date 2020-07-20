---
title: "Do I need so many gets and sets in my class?"
excerpt: "People prefer things that are easy to use. Use abstractions to hide the hard stuff from the users and allow everyone to enjoy what you create."
date: 2020-07-19T16:57:30-03:00
categories:
  - Blog
tags:
  - OOP
---

There are two reasons to make class attributes private: **safety** and **abstraction**.

## Safety

**Safety** is not about people seeing your code or doing things that can lead to vulnerabilities. It is about avoiding that the user performs unintended actions that can lead to weird behaviors in the class logic, not expected by its creator. If all attributes were public and directly accessed, a bug in the class logic could be anywhere in the code. However, with private attributes, you are most likely sure that the bug is inside the class, making it easier to find it.

As an example, a simple mistype like `if (account.balance = transferAmount)` instead of `if (account.balance == transferAmount)` somewhere in the code could make money vanish from your users' accounts. With private attributes, you would explicitly call `getBalance()`, avoiding this situation. If the money started vanishing even with private attributes, you already know where to look.

## Abstraction

The other reason is **abstraction**, which is the main topic of this post.

The idea of abstraction is that the user of the class should be able to interact with the object in a friendly, simple way, without needing to know the hard concepts behind it. What is being hidden is not the attribute itself (sometimes it is, but this is out of scope here), but the hard concepts behind it.

### Hide the hard stuff

Let's walk through an example. If you were to create a class `Car`, how would you implement it? Like this, maybe:

```cpp
class Car {
  float speed;

  public:
    void setSpeed(float speed) {
      this->speed = speed;
    }

    float getSpeed() {
      return this->speed;
    };
};
```

With just encapsulation into mind, this could be fine. If you wanna go fast, just increase the speed. However, is this how it really works? Do you tell your car "hey, go 50km/h now"? A lot of cars have cruise control nowadays, but manually this is not how you drive a car. If you want to go faster, you press the acceleration pedal. If you want to slow down, you press the brake pedal. Also, the speed does not just go to the value you want immediately. A better approach would be:

```cpp
class Car {
  float speed;

  // ...
  // a lot of methods that control the internals of the car
  // like engine, breaks, oil, gas, etc
  // ...

  public:
    void accelerate(float pedalPressure) {
      // really hard algorithm that an engineer created 
      // to make the car accelerate when you press a pedal
    }

    void slowDown(float pedalPressure) {
      // really hard algorithm that an engineer created 
      // to make the car break when you press a pedal
    }
};
```

See how the only thing the driver needs to know is that he uses a pedal to accelerate and another to break? He does not need to know that when he presses the pedal more gas enters the engine, making it produce more power, and finally making the wheels spin faster (I don't know a lot about cars, but this is good enough to understand). Or that the brake pedal actually moves a part in the car that enters in contact with the wheel, making it slow down, and has internal systems that avoid the wheel to lock if the driver brakes too hard. The technical concepts are hidden, allowing any person without a mechanical engineer Ph.D. to drive a car.

Another thing is the get methods. Let's change the example. Now we will use the class of a digital clock, simplified to just the part that shows the time:

```cpp
class Clock {
  int time; // in seconds

  public:
    int getTime() {
      return this->time;
    }
};
```

See that the time is just being stored as an integer. However, digital clocks do not show the time in seconds, but in a HH:MM format. So the abstraction implementation of this class should be:

```cpp
class Clock {
  int time; // in seconds

  string formatTime(int time) {
    // method return the time formatted in HH:MM
  }

  public:
    string getTime() {
      return formatTime(this->time);
    }
};
```

The user does not want to know how many seconds passed since the day started. He wants to know the hour and the minute of the day, so he won't be late to work. This is valid to the `Car` example too. Internally, the speed is calculated using the angular velocity of the wheel. Only when the speed is presented in the velocimeter, the system changes it to km/h and display it.

### If my class stores an attribute in the same way the user interacts with it, can I make the attribute public?

You shouldn't. If later your class changes to store this attribute differently, it should not impact the user. If the attributes were to be accessed directly, a change in the class would require a refactor of all the code that uses this class. Protect yourself from your future self.

Also, because of safety, which I talked about at the beginning of the post. A bug in the class behavior could be caused anywhere in the code if you expose your attributes this way. 

## Overview

Users of a class should not need to know its intricate internals. Abstraction guarantees that everyone without domain knowledge can use the object easily (both in real life and in programming). Also, use private attributes to avoid class users to make unintended modifications that can lead to unexpected behavior. Be sure to lead them through the happy path you planned for the class.
