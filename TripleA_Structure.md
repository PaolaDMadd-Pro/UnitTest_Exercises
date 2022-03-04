 ### Triple 'A' Test structure and test methods name conventions:

Here few basic rules:
1. Syntax
- I public void
    - A testing method is always void.
- II name contains the methodname we want to test + scenario + expected behaviour.
    -  methodNameWewantToTest_Scenario_Expectedbehavior()

2. Arrange, Act, Assert 
- this is the structure that follows the testing method body 

        [Test]
        public void CanBeCancelledBy_UserIsAdmin_ReturnsTrue()
        {
            //Arrange is where we initalize the object

            var reservation = new Reservation();

            //Act the method we are going to test

            var result = reservation.CanBeCancelledBy(new User{IsAdmin = true });

            //Assert 
            Assert.IsTrue(result);
            //can be wtritten also in 2 other ways:

            //Assert.That(result, Is.True);
            //Assert.That(result == true);

        }

Depending on what we want to test we might adopt a different approach.
###### State Based testing
The above code is also known as **State-Based testing** because we test the state changes of the applications.

###### Interaction Testing
Sometimes we have to deal with external resources and we need to verify that the class we are testing interacts with another class properly. This is what we call
**Interaction Testing**.
In the example here below the `orderService` should store the `order` in a external storage (database or cloud)

```
    public class OrderService
    {
        public void PlaceOrder(Order Order)
        {
            _storage.Store(Order):
            ...
        }
    }
``` 
So what we are going to test is that the `OrderService object` is interacting properly with the `storage object property`.
We'll do this checking that our code calls the store method of the storage object with the right argument.

#### Conclusions:

Is better to reserve the Interaction testing approach only for dealing with external resources; this because with interaction testing, your tests start to couple
during the implementation.\
What need to be asserted is just to verify that the right method is called with the right argumnents.
As you refactor and restrcture your code, it is possible you move some of these methods around, doing that you may break one or more tests.\
It is worthy to emphasize, that your test shoud test the external behaviour and not the implementation, therefore prefer statebased testing to interaction testing; 
and use interaction testing only when dealing with external resources.

### Testing Interaction between 2 objects

```
 [TestFixture]
    public class OrderServiceTests
    {

        [Test]
        public void PlaceOrder_WhenCalled_StoreTheOrder()
        {
// Arrange
            var storage = new Mock<IStorage>();
            var service = new OrderService(storage.Object);
            var order = new Order();
// Act
            service.PlaceOrder(order);
// Assert 
            storage.Verify(s => s.Store(order));

        }
    }

```

We are programming the mock object with `storage.Verify(s => s.Store(order));`.\
In order to test the interaction between 2 objects we use the `.Verify()` method of mock object.
In this way we are testing if any given method is called with the right arguments or not.

#### wants more? read also [Dependency Injection](https://en.wikipedia.org/wiki/Object_graph)