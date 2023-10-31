---
layout: post
title:  The Tortured Object
categories: [Code, Software Engineering]
---

As I was learning about design patterns and anti-patterns, I came across an anti-pattern that I hadn’t seen online. So, I decided to write about it so others can learn more about it: the ***tortured object***. I know the <em>tortured object</em> sounds weird, but it'll make sense as you keep reading. The tortured object is an anti-pattern that involves a failing design pattern in which a factory pattern can be implemented to simplify the problem. 

One way to identify this anti-pattern is when one encounters code duplication. One of the widely taught fundamental principles of software development is DRY, which stands for 'Don’t Repeat Yourself.' As Andy Hunt and Dave Thomas said, 'Every piece of knowledge must have a single, unambiguous, authoritative representation within a system' (Hunt & Thomas, 1999). Although this is commonly taught and preached, it isn’t commonly practiced. Duplicate code has several issues (if it wasn’t already obvious). One of the major problems is its effect on maintainability. If you need to modify code in multiple places, it may impact your velocity. In some cases, code duplication can also affect the performance of the program.

The tortured object anti-pattern also makes extension difficult, as it does not adhere to the open-closed SOLID principle. This principle states that code should be open to extension but closed to modification. This means that we should be able to add functionality to our code without modifying the existing code. For example, let's use the common example of a `FlightInfoRetriever` class where there can be different airlines with various clients and different ticket numbers. Here’s an example of some code that doesn’t adhere to the open-closed principle.


```python
class FlightInfoRetriever:
    def __init__(self, airline, ticket_number):
        self.airline = airline
        Self.ticket_number = ticket_number
    
    def get_flight_info(self, token=None):
        if self.airline == 'AirCanada':
            client = getClient(AIR_CANADA_URL) 
        elif self.airline == 'United':
            client = getClient(UNITED_URL, token)
        elif self.airline == 'AirFrance':
            client = getClient(AIR_FRANCE_URL)
        # More elif conditions for other airlines...
        Return client.get_info(self.ticket_number)


# Client code
ac = FlightInfoRetriever('AirCanada')
ac.get_flight_info()

united = FlightInfoRetriever('United')
united.get_flight_info(getToken())

af = FlightInfoRetriever('AirFrance')
af.get_flight_info()
```

In this code, you can see that if you want to add more airline clients, you would have to modify the code inside the `get_flight_info` method, which may break tests and increase the complexity of the code by becoming a conditional hell. Imagine having to modify every single test anytime a new airline is added; that doesn't make any sense at all. While it may not be obvious in this case, there is a tortured object in this code. This means there are signs that there should be an object instantiated for the various airlines. Usually, the easiest way to spot a tortured object anti-pattern is to check if there are methods or classes with repeating prefixes or suffixes. For example, let’s take the above code and improve it to follow the open-closed principle, and it may become a bit more obvious:


```python
class FlightInfo:
    def get_flight_info(self, ticket_number, token=None):
        pass  # Abstract method

class AirCanadaInfo(FlightInfo):
def __init__(self, ticket_number):
        self.ticket_number = ticket_number

    def get_flight_info(self):
        return getClient(AIR_CANADA_URL).get_info(self.ticket_number)

class UnitedInfo(FlightInfo):
def __init__(self, ticket_number):
        self.ticket_number = ticket_number

    def get_flight_info(self, token):
        return getClient(UNITED_URL, token).get_info(self.ticket_number)

class AirFranceInfo(FlightInfo):
def __init__(self, ticket_number):
        self.ticket_number = ticket_number

    def get_flight_info(self):
        return getClient(AIR_FRANCE_URL).get_info(self.ticket_number)

# Client code
ac = AirCanadaInfo(ticket_number)
ac.get_flight_info()


united = UnitedInfo(ticket_number, getToken())
united.get_flight_info()

af = AirFranceInfo(ticket_number)
af.get_flight_info()
```

Now you can start to see the repeating suffixes (AirCanadaInfo, UnitedInfo, AirFranceInfo), and what looked like repeating code before still looked like repeating code after the refactor. You can see that when creating instances of the various airlines it can become quite cumbersome. This indicates that there are more simplifications to come. Sandy Metz mentions in her brilliant talk [“All the Little Things”](https://youtu.be/8bZh5LMaSmE) that explains this kind of scenario (watch the talk, it’s brilliant and the inspiration for this article!) - the intermediate steps from complex code to simple code may actually increase code complexity. You just have to keep believing that your refactoring, following object-oriented principles, will eventually lead to simpler code. 

So, taking a look at this code, you can see that there is a tortured object. The antidote to this anti-pattern is the factory design pattern. All anti-patterns are like venom; they’re dangerous and can hurt you. However, most of them have antivenom that can be used to treat the venom. In this case, we can use a factory to simplify our code and adhere to the DRY principle as well as SOLID principles:

```python
class FlightInfoFactory:
    @staticmethod
    def create(airline, ticket_number):
        if airline == ‘AirCanada’:
            return AirCanadaInfo(ticket_number)
        elif airline == ‘United’:
            return UnitedInfo(ticket_number, getUnitedToken())
        elif airline == 'AirFrance':
            return AirFranceInfo(ticket_number)
        else:
            raise ValueError("Invalid airline")

# Client code using the factory

ac = FlightInfoFactory.create(‘AirCanada’, ticket_number)
ac.get_flight_info()

united = FlightInfoFactory.create(‘United’, ticket_number)
united.get_flight_info()

af = FlightInfoFactory.create(‘AirFrance’, ticket_number)
af.get_flight_info()

```

In this example, you can see that creating different types of airline clients is now simplified by using the `FlightInfoFactory` class, which creates the various types of airline clients based on the input. We have successfully eliminated the tortured object; our factory now instantiates the necessary objects depending on the input. This is easier than calling the various classes yourself for each airline that you have to instantiate. Now, testing becomes a lot easier as well. Anytime in the future, if the functionality is extended to include more airlines, you would not have to modify any of the existing tests; you would either extend a test or add new tests. In this case, the client code doesn’t need to know the various subclasses, and this decoupling actually improves scalability and maintainability. A great example of this can be demonstrated by the Gilded Rose Kata, which Sandy Metz discusses in her earlier-mentioned talk.

So in conclusion, if you can recognize the tortured object anti-pattern you can use the factory design pattern to help simplify your code. Using the different object-oriented principles and software engineering principles you can improve the quality, scalability, and the maintainability of your code. I hope you took something away from this!

<em>Special thanks to Chris Johnston for introducing me to the topic!</em>
