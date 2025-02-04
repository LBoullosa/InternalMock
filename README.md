<p align="center">
  <img width="25%" height="25%" src="https://github.com/hassanhabib/InternalMock/blob/master/InternalMock/InternalMock.png">
</p>


# InternalMock
[![.Net](https://github.com/hassanhabib/InternalMock/actions/workflows/dotnet.yml/badge.svg)](https://github.com/hassanhabib/InternalMock/actions/workflows/dotnet.yml)
[![The Standard - COMPLIANT](https://img.shields.io/badge/The_Standard-COMPLIANT-2ea44f)](https://github.com/hassanhabib/The-Standard)

## Introduction
This is a beta release please use carefully.

This library allows you to mock internal private static methods in your service - please watch the following videos for context:
- [Developing InternalMock Library from Scratch](https://www.youtube.com/watch?v=SuyzK2aWdqI)
- [On The Topic Of Inner Exceptions](https://www.youtube.com/watch?v=t7iBywCu35U)
- [TryCatch Reflection In Depth](https://www.youtube.com/watch?v=694A0qj6uC4)
- [Postfixing Anonymous Functions](https://www.youtube.com/watch?v=nGU8OD7CRdY)


## Architecture
Here's the architecture of the library according to [The Standard](https://github.com/hassanhabib/The-Standard):

![image](https://user-images.githubusercontent.com/1453985/164965639-ae86363c-e29b-48ef-b1b0-b39a95a860b0.png)

## Examples
Let's Assume you have a service that has several functions that don't call any dependencies. Your service is what we call self-sufficient or dead-end service, as the flow stops there and might just be returned from the same service. An example of a service like this is a tax calculation service, you pass the total income, along with some other details and it calculates the taxes for a certain year. It doesn't call any dependencies. 

Now, in that very unique scenario we need to find a way to test-drive that our self-sufficient service here can handle a generic `exception` or any other exception of any type. Since there are no dependencies injected, it's impossible to tag an exception the regular way where we do:
```csharp
  this.someDependency.Setup(dependency =>
    dependency.GetStuff())
      .Throws(exception);
```

The solution here is to create inner dependencies. A self-sufficient service can rely on multiple other private static functions to perform certain functions. We can mock these functions without changing the exception handling code as follows:

Let's say our service looks like this:
```csharp
public string RetrieveStudentFullName(string firstName, string lastName)
{
  ValidateStudentName(firstName, lastName);

  return $"{firstName} {lastName}";
}
```

We can write the test as follows to make the `ValidateStudentName` function throw an exception as follows:

```csharp
  [Fact]
  public void ShouldThrowServiceExceptionOnRetrieveStudentFullNameIfServiceErrorOccurrs3()
  {
      // given
      var exception = new Exception();

      this.studentService.Mock(
        methodName: "ValidateStudentName")
          .Throws(exception);

      // when
      Action retrieveStudentFullNameAction = () =>
        this.studentService.RetrieveStudentFullName(
          firstName: "Hassan", 
          lastName: "Habib");

      // then
      Assert.Throws<StudentServiceException>(retrieveStudentFullNameAction);

      this.expressionService.ClearAllOtherCalls();
  }
```

And now we can make that very same test pass by doing the following:

```csharp
  public string RetrieveStudentFullName(string firstName, string lastName)
  {
    try
    {
      ValidateStudentName(firstName, lastName);

      return $"{firstName} {lastName}";
    }
    catch (Exception exception)
    {
      throw new StudentServiceException(exception);
    }
  }
```

## Contact us

For more information contact Hassan Habib: hassanhabib@live.com
Also join our Standard Community on Discord Community here:

[![Discord](https://discord.com/api/guilds/934130100008538142/widget.png?style=banner2)](https://discord.gg/vdPZ7hS52X)
