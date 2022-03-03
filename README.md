notes and examples

### My Guide & Notes
<br/>

##### 3 ways of implementing dependecy injections:

1. via method parameter
2. via constructors
3. via properties

<br/>
<br/>

##### 1. Method Parameter

in `VideoService.cs` file
``` 
 public string ReadVideoTitle(IFileReader fileReader)
        {
            var str = fileReader.Read("video.txt");
            
            //do something
        }
```


in `program.cs` file we have passed an instance of a class that implement the FileReader.
But in real life we don't initialize new fileReader, we inject the framework instead.

``` 
 public class Program
    {
        public static void Main()
        {
            var service = new VideoService();
            var title = service.ReadVideoTitle(new FileReader());

        }
    }
```


in the `unitTest project`we recreate the same structure as the app project (in our case, adding the mocking folder)
we also create a class `videoServiceTests` here we will implement a mock function as follows:

```
    [TestFixture]
    public class VideoServicesTests
    {
        [Test]
        public void ReadVideoTitle_EmptyFile_ReturnsErrorMsg()
        {
            var service = new VideoService();
            var result = service.ReadVideoTitle(new FakeFileReader());

            Assert.That(result, Does.Contain("Error").IgnoreCase);
        }
    }
```

**N.B.** 
This method has some challenges, for example if you need to modify its signature will change,
this means that we need to make the change in all other places it's been used.
Also some injection framework can not inject methods via parameters.
So this approach requires to take into count 
 
- If it's allowed by the framework used in the app.
- How many times the method you're using is present in your app.
<br/>
<br/>

##### 2. Property

The second apprach is using property and inintialize it, with a construstor as follows:
in `VideoService.cs` file 
``` 
    public IFileReader FileReader { get; set; }

        public VideoService()
        {
            FileReader = new FileReader();
        } 
```

in the `program` file we need to remove the parameter(from the previous example)

``` 
   public class Program
    {
        public static void Main()
        {
            var service = new VideoService();
            var title = service.ReadVideoTitle();

        }
    }
 ```
as well as in `VideoServiceTests` file and we need to initialize the file reader property with a "fake" property 
in our case we created a class in our UnitTests project called `FakeFileReader`, as follows:
```
public class FakeFileReader : IFileReader //by  convention can be named Stub or Fake or Mock + interface name
    {
        public string Read(string path)
        {
            return " ";
        }
    }
```

```
 [TestFixture]
    public class VideoServicesTests
    {
        [Test]
        public void ReadVideoTitle_EmptyFile_ReturnsErrorMsg()
        {
            var service = new VideoService();
            service.FileReader = new FakeFileReader();

            var result = service.ReadVideoTitle();

            Assert.That(result, Does.Contain("Error").IgnoreCase);
        }

    }
```

**N.B.** some Framework doesn't accept injections via properties.
<br/>
<br/>

##### 3. Constructor
Another alternaive to the firts two methids is the injection of a dependecy via a constuctor parameteter,
as follows:

in `videoService` file

```
public class VideoService
   {
    private IFileReader _fileReader; 

        public VideoService(IFileReader filetreader)
        {
            _fileReader = filetreader;
        }
    }
```
Note, that this field needs to be private and by convention the naming will start by unserscore.
If we are modify an existing code this latter implementation myght break the code in other spots.
A better alernative is to create a default constructor that doesn't take a parameter, like this:

```
 public class VideoService
   {
    private IFileReader _fileReader;

       public VideoService()
        {
            _fileReader = new FileReader();
        }
        public VideoService(IFileReader filetreader)
        {
            _fileReader = filetreader;
        }
    }
``` 

A further improvement to this code can be to combine into one in two steps
1. set the parameter as null by default
2. set the `_filereader` private field using a ternary conditional operator (in this case null-coalescing operator).

```
 public class VideoService
    {

        private IFileReader _fileReader;

        public VideoService(IFileReader filetreader = null)
        {
            _fileReader = filetreader ?? new FileReader();
        }
   }
```
Setting the parameter as null by default will make the method work where no parameter are passed.
Using the ternary conditional operator will set the `_filereader` private field as the parameter received or if null, 
it will be initialized.

We also need to change the `VideoServiceTest` file like this:
```
 [TestFixture]
    public class VideoServicesTests
    {
        [Test]
        public void ReadVideoTitle_EmptyFile_ReturnsErrorMsg()
        {
            var service = new VideoService(new FakeFileReader ());

            var result = service.ReadVideoTitle();

            Assert.That(result, Does.Contain("Error").IgnoreCase);
        }

    }
```
**N.B.** using the null-coalescing operator can be a solution but is not used in the real world, 
because this class might have more dependencies in that case this expression will be repeated over and over.
But above of all in real world the null parameter is not a good practice. 
This latter approach is defined as " poorman's dependency injection".
In real life application, the best solution will be to use a framework injection.

<!-- visible breakline is " __ __" 2underscore1whitespace2underscore--> 
<!-- breakline is simple html <br/> tag-->
<br/>

##### Bonus: Framework injection

A dependency injection framework will take care of creating and initializing object at run time.
There are various framework you can choose from: NInject, StructureMap, Spring.NET, Autofac, Unity.
They all have the same principles:
A container which is a registry of all interfaces and implementations.
When the application starts, it will take care of creating object graphs based on the interfaces and types registered in the container.
Using the same principle with **Mocking Isolation Framework**, we can create dynamically mock objects as part of the tests' suit,
and more important we can program them to behave the way we want. Eg. returning a value, throw an exception, raise an events. 
Here too we have options like: Moq, NSubstitute, FakeItEasy, Rhino Mocks.

The framework (in our case Moq) will replace and implement the `FakeFileReader.cs` file.

In the `VideoTestService.cs` we need to initialize the mock object using the generic argument like so:
`var mockfileReader = new Mock<IFileReader>();` as this is a framework the mock object we are initializing hasn't a behaviour, so we need to program it like our app object.

`VideoService.cs`
```
public class VideoService
    {
        private IFileReader _fileReader;

        public VideoService(IFileReader filetreader)
        {
            _fileReader = filetreader;
        }

        public string ReadVideoTitle()
        {
            var str = _fileReader.Read("video.txt");
            var video = JsonConvert.DeserializeObject<Video>(str);
            if (video == null)
                return "Error parsing the video.";
            return video.Title;
        }
    }
```
the `ReadVideoTitle()` is calling the `.Read()` receiving a string "video.txt" and it returns a string.
To do so we need to use the built-in `setup()` method, like this:
`fileReader.Setup(fr => fr.Read("video.txt")).Returns("");`

**N.B.** Mocks should be reserved only for external dependencies

it should look like this:
```
[TestFixture]
    public class VideoServicesTests
    {
        [Test]
        public void ReadVideoTitle_EmptyFile_ReturnsErrorMsg()
        {
            var fileReader = new Mock<IFileReader>();
            fileReader.Setup(fr => fr.Read("video.txt")).Returns("");

            var service = new VideoService(fileReader.Object); 

            var result = service.ReadVideoTitle();

            Assert.That(result, Does.Contain("Error").IgnoreCase);
        }

    }
```
Certainly if we need to use the framework for more than one test we might want to create a setup function.

```
 [TestFixture]
    public class VideoServicesTests
    {
        private VideoService _videoService;
        private Mock<IFileReader> _fileReader;
        [SetUp]
        public void SetUp()
        {
             _fileReader = new Mock<IFileReader>();
            _videoService = new VideoService(_fileReader.Object);
        }

        [Test]
        public void ReadVideoTitle_EmptyFile_ReturnsErrorMsg()
        {
           _fileReader.Setup(fr => fr.Read("video.txt")).Returns("");

            var result = _videoService.ReadVideoTitle();

            Assert.That(result, Does.Contain("Error").IgnoreCase);
        }

    }
```

