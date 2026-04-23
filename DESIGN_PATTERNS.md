# Design Patterns

This article will explain 7 of my most favorite design patterns that I think every developer should be aware of.

Design patterns are solutions to common problems that many developers face when writing code. They can help you write more readable, scalable, and maintainable code.
Most design patterns fall under one of three categories: creational, structural, and behavioral.

A lot of frameworks and libraries use the patterns I will be describing in this article. In fact, you have probably used some of these patterns in your favorite libraries without even realizing it.

# Creational patterns

Creational patterns are design patterns that are focused how objects are created.

## 1. Builder

The builder pattern is a creational pattern where complex objects are constructed step-by-step. It makes object creation easier to work with, and it makes your code a lot more readable.

Think of the builder pattern like a recipe - when you're following a recipe, you're not throwing in all the ingredients all at once; you're adding each ingredient step-by-step to come to your final result.

### Example

Let's imagine there is a JavaScript object that represents a pizza:

```js
{
  toppings: ['cheese', 'pepperoni'],
  sauce: 'tomato'
}
```

Instead of creating the object using the object literal syntax, we could create a builder class that provides setters to create the pizza step-by-step:

```js
class Pizza {
  constructor() {
    this.toppings = [];
    this.sauce = null;
  }

  addTopping(toppingName) {
    this.toppings.push(toppingName);
    return this;
  } 

  setSauce(sauceType) {
    this.sauce = sauceType;
    return this;
  } 
}

```

Now, we can build the same pizza, but instead of creating a plain object literal, we can construct the pizza step-by-step using the setters we defined in our builder class instead.

```js
const pizza = new Pizza()
  .setSauce('tomato')
  .addTopping('cheese')
  .addTopping('pepperoni');
```

Now our code just reads like plain English; we want to make a new pizza, put tomato sauce on it, then add cheese and pepperoni.

> In real-life scenarios, objects created using the builder pattern will usually be more complicated than the one in this example.

## 2. Singleton

The singleton pattern is a creational pattern that is quite straightforward; it is a pattern where a class can only have one instance. It's commonly used to manage access to shared resources, such as a database connection or a file.
There's not much else to say about the singleton pattern, so I'm just going to move on to the example.

### Example

Let's create a `Logger` class. This class will be responsible for logging messages and saving them to the console:

```js
class Logger {
  constructor() {
    this.logs = [];
  }

  log(message) {
    console.log(message);
    this.logs.push(message);
  }

  warn(message) {
    this.log(`[WARNING] ${message}`); 
  }
}

```

In the example above, our `Logger` class doesn't implement the singleton pattern, but we can change that:

```js
class Logger {
  static instance;
 
  constructor() {
    if (Logger.instance) return Logger.instance;

    this.logs = [];
    Logger.instance = this;
  }

  static getInstance() {
    if (Logger.instance) {
      return Logger.instance;
    } else {
      return new Logger();
    }
  }

  log(message) {
    console.log(message);
    this.logs.push(message);
  }

  warn(message) {
    this.log(`[WARNING] ${message}`); 
  }
}

```

Now, the class will check whether there is already an existing instance of it. If there is, it will return the instance that already exists, and if there isn't, then a new instance of `Logger` will be created and set as the value of the static `instance` property.

## 3. Factory

The factory pattern is a creational design pattern that handles the instantiation of classes via a single shared instance.

### Example

To make this simpler to understand, let's take a look at an example.
We have two classes - a Windows button class and a Linux button class. The Windows button will be red with rounded corners, and the Linux button will be blue with sharp corners.
We have a variable named `platform`, and we want to create an instance of the `WindowsButton` class only if the platform is set to `'windows'`.

This gets pretty repetitive without the factory pattern:

```js
const platform = 'windows';

class Button {
  display() {
    // here's the code to display the button
  }
}

class WindowsButton extends Button {
  constructor() {
    super();
    this.color = 'red';
    this.corners = 'rounded';
  }
}

class LinuxButton extends Button {
  constructor() {
    super();
    this.color = 'orange';
    this.corners = 'sharp';
  }
}

const button = platform === 'windows' ? new WindowsButton() : new LinuxButton();
const button2 = platform === 'windows' ? new WindowsButton() : new LinuxButton();

button.display();
button2.display();

```

Not only is the code above repetitive, but it is also very difficult to scale and maintain. If there were 20 of these conditional checks all over the codebase and you wanted to add a new MacOS-themed button, you would have to update the code in each of those 20 places, which is a nightmare to maintain.

Now, let's rewrite the code above with the factory pattern:

```js
const platform = 'windows';

class Button {
  display() {
    // here's the code to display the button
  }
}

class WindowsButton extends Button {
  constructor() {
    super();
    this.color = 'red';
    this.corners = 'rounded';
  }
}

class LinuxButton extends Button {
  constructor() {
    super();
    this.color = 'orange';
    this.corners = 'sharp';
  }
}

class ButtonFactory {
  static createButton(platform) {
    if (platform === 'windows') {
      return new WindowsButton();  
    } else if (platform === 'linux') {
      return new LinuxButton();
    } else {
      throw new Error('unrecognized platform');
    }
  }
}

const button = ButtonFactory.createButton('windows');
const button2 = ButtonFactory.createButton('linux');

button.display();
button2.display();

```

We're now implementing a class called ButtonFactory, which deals with the creation of button classes.
Now, if we want to add a new type of button, we don't have to modify the code in 20 different places; instead, we can just update a single method.

# Structural patterns

Structural patterns deal with how objects relate to each other.

## 4. Facade

Facade is basically just a fancy term for encapsulation. It is a design pattern where you hide the inner complexities of a system by creating a simplified interface that hides the underlying details and procedures.

In a way, you could say that any framework or library is a facade; Discord.JS acts as a facade that provides a simplified interface to interact with Discord's HTTP API, Pixi.JS acts as a facade that provides a simplified interface for the HTML Canvas API, and jQuery acts as a facade that provides a simplified interface for DOM manipulation.

## 5. Adapter

The adapter pattern is a structural design pattern that lets two objects with incompatible interfaces interact with eachother. 

It's like a USB adapter; for example, if you're using a normie laptop that has a USB port but not an ethernet port, then you might use an ethernet-to-USB adapter that lets you use an ethernet cable despite your laptop not having built-in ethernet cables. It's the same idea with the adapter design pattern.

### Example

Let's say there's a program that downloads the current temperature from an online weather API. The only problem is that your program expects the temperature to be in degrees, but the API only provides the weather in farenheit. Someone who doesn't know about the adapter pattern might do something like this:

```js
class WeatherApi {
  static async getTemperature() {
    const response = await fetch('https://weatherapi.com/api/temperature'); // not a real url btw
    const text = await response.text();
    return text; 
  }

  static async getRainForecast() {
    const response = await fetch('https://weatherapi.com/api/rainForecast'); // not a real url btw
    const text = await response.text();
    return text; 
  }
}

const farenheitToCelcius = (farenheit) => {
  return (farenheit - 32) / 1.8;
}

(async () => {
  const temperature = farenheitToCelcius(await WeatherApi.getTemperature());
  console.log(`temperature: ${temperature}`);

  setTimeout(() => {
    const newTemperature = farenheitToCelcius(await WeatherApi.getTemperature());
    console.log(`temperature difference after 1 hour: ${newTemperature}`);
  }, 60 * 60 * 1000);
})();
```

But this is not a feasible solution. A better alternative would be to create an adapter class for the weather API to make the response compatible with your program's expectations.

```js
class WeatherApi {
  static async getTemperature() {
    const response = await fetch('https://weatherapi.com/api/temperature'); // not a real url btw
    const text = await response.text();
    return text; 
  }

  static async getRainForecast() {
    const response = await fetch('https://weatherapi.com/api/rainForecast'); // not a real url btw
    const text = await response.text();
    return text; 
  }
}

class WeatherApiAdapter extends WeatherApi {
  static async getTemperature() {
    const farenheit = await WeatherApi.getTemperature();
    const celcius = (farenheit - 32) / 1.8;
    return celcius; 
  }
}


(async () => {
  const temperature = await WeatherApiAdapter.getTemperature();
  console.log(`temperature: ${temperature}`);

  setTimeout(() => {
    const newTemperature = await WeatherApiAdapter.getTemperature();
    console.log(`temperature difference after 1 hour: ${newTemperature}`);
  }, 60 * 60 * 1000);
})();
```

Now, we can call the `getTemperature` method directly instead of having to deal with the conversion in our actual code. We're basically creating an adapter for the incompatible API.

# Behavioral design patterns

Behavioral design patterns focus on how objects communicate with each other.

## 6. Chain of Responsibility

The chain of responsibility pattern is a common behavioral design pattern.

The chain of responsibility is similar to a linked list; the chain of responsibility pattern consists of multiple handlers which have references to the next handler. When data is passed to a handler, that handler performs some operation on the data, and it decides whether or not to pass it on to the next handler.

If you've worked with Express.js, then you might be familiar with middleware. Express.JS middleware is an example of the chain of responsibility pattern.

## 7. Observer Pattern

The observer pattern is another common behavioral pattern used to create a one-to-many relationship between objects.

It consists of a central object called the subject (or the publisher) and multiple objects that are dependent on the subject. The subject will automatically notify all of its dependent objects of any changes.

### Example

```js
class YouTuber {
  constructor() {
    this.subscribers = new Set();
  }

  subscribe(func) {
    this.subscribers.add(func);
  }

  unsubscribe(func) {
    this.subscribers.delete(func);
  }
  
  notify(message) {
    this.subscribers.forEach(func => func(message));
  }
}

const youtuber = new YouTuber();
youtuber.subscribe(message => console.log('my favorite youtuber just posted a video'));
youtuber.subscribe(message => console.log('new video from the youtuber'));
youtuber.subscribe(message => console.log(`there's a new video from my favorite youtuber ${message}`));

youtuber.notify('asdfasdf');

/*
Output:

my favorite youtuber just posted a video
new video from the youtuber
there's a new video from my favorite youtuber asdfasdf
*/
```

The code above implements the observer pattern. There is a publisher (the `youtuber` object), and multiple observers (subscribers). The observers can observe the notifications from the `youtuber` object, and all the observers will be updated with any new information.

# When to use each pattern

Although you've just learned about 7 design patterns, I don't think you should over-use them. Use these patterns where they are appropriate, and don't over-complicate your code by using them where they are not necessary. Over-using design patterns can actually make your code more difficult to read and maintain.
