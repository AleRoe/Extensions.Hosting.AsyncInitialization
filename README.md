# Extensions.Hosting.AsyncInitialization

[![NuGet version](https://img.shields.io/nuget/v/Extensions.Hosting.AsyncInitialization.svg?logo=nuget)](https://www.nuget.org/packages/Extensions.Hosting.AsyncInitialization)
[![AppVeyor build](https://img.shields.io/appveyor/ci/thomaslevesque/extensions-hosting-asyncinitialization.svg?logo=appveyor)](https://ci.appveyor.com/project/thomaslevesque/extensions-hosting-asyncinitialization)
[![AppVeyor tests](https://img.shields.io/appveyor/tests/thomaslevesque/extensions-hosting-asyncinitialization.svg?logo=appveyor)](https://ci.appveyor.com/project/thomaslevesque/extensions-hosting-asyncinitialization/build/tests)

A simple helper to perform async application initialization for the generic host in .NET 6.0 or higher (e.g. in ASP.NET Core apps).

## Usage

1. Install the [Extensions.Hosting.AsyncInitialization](https://www.nuget.org/packages/Extensions.Hosting.AsyncInitialization/) NuGet package:

    Command line:

    ```PowerShell
    dotnet add package Extensions.Hosting.AsyncInitialization
    ```

    Package manager console:
    ```PowerShell
    Install-Package Extensions.Hosting.AsyncInitialization
    ```


2. Create a class (or several) that implements `IAsyncInitializer`. This class can depend on any registered service.

    ```csharp
    public class MyAppInitializer : IAsyncInitializer
    {
        public MyAppInitializer(IFoo foo, IBar bar)
        {
            ...
        }

        public async Task InitializeAsync(CancellationToken cancellationToken)
        {
            // Initialization code here
        }
    }
    ```

3. Register your initializer(s) in the same place as other services:

    ```csharp
        services.AddAsyncInitializer<MyAppInitializer>();
    ```

4. In the `Program` class, make the `Main` method async and change its code to initialize the host before running it:

    ```csharp
    public static async Task Main(string[] args)
    {
        var host = CreateHostBuilder(args).Build();
        await host.InitAsync();
        await host.RunAsync();
    }
    ```

    You can also pass a `CancellationToken` in order to propagate notifications to cancel the initialization if needed.

    In the following example, the initialization will be cancelled when `Ctrl + C` keys are pressed :
    ```csharp
    public static async Task Main(string[] args)
    {
        using CancellationTokenSource cancellationTokenSource = new CancellationTokenSource();

        // The following line will hook `Ctrl` + `C` to the cancellation token. 
        Console.CancelKeyPress += (source, args) => cancellationTokenSource.Cancel();

        var host = CreateHostBuilder(args).Build();

        await host.InitAsync(cancellationTokenSource.Token);
        await host.RunAsync();
    }
    ```

This will run each initializer, in the order in which they were registered.
