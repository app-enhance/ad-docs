# Dependency injection

`AE.Extensions.DependencyInjection` packages are an extension library for `Microsoft.Extensions.DependencyInjection` which provides you with a declarative way to pick out lifetime of services and register them in just a few lines.

## Motivation
When you want to register a service (create dependency) in `IServiceCollection` you have to describe at least 3 things:

* Service interface
* Service implementation
* Lifetime scope

Usually it should look like this (see below)

```csharp
// Method ex. in Startup.cs
// doc: http://docs.asp.net/en/latest/fundamentals/dependency-injection.html

public void ConfigureServices(IServiceCollection services)
{
    var serviceDescriptor = new ServiceDescriptor(typeof(IBankManager), typeof(BankManager), ServiceLifetime.Transient);
    services.Add(serviceDescriptor);
    // or
    services.AddTransient<IBankManager, BankManager>();

    // Add MVC services to the services container.
    services.AddMvc();
}
```

```csharp
/// And service definition somewhere, deeper in the code
public interface IBankManager
{
    int OpenAccount(string clientName);
}

public class BankManager : IBankManager
{
    public int OpenAccount(string clientName)
    {
        // Account creating process...

        return Random.Next(1000000, 9999999);
    }
}
```

Every service must be described in `ConfigureServices` method manualy which means that something can be easily missed.
Many people who create libraries like MVC they add static extension methods to simplify registration.
It means all descriptions of their services are hidden.
They make our life easier but what about our own custom services ? What if you have dozens of them ?

## Overview

I'm inspired by how Orchard team resolves that issue.
They created interfaces which correspond to lifetime scopes. So every service interface inherits appropriate "scope interface" and describes dependencies to register in global container.
In my case it looks like below - interfaces correspond exactly to `ServiceLifetime` enum.

```csharp
// You can use it to get all dependencies
public interface IDependency
{
}

public interface ISingletonDependency : IDependency
{
}

public interface IScopedDependency : IDependency
{
}

public interface ITransientDependency : IDependency
{
}

// Omit registration for special cases
public interface INotRegisterDependency
{
}
```

## How does it work?

In order to use this approach you have to do two things:

* Select dependency interface and inherit

```csharp
// It will be registered as a
// new ServiceDescriptor(typeof(IBankManager), typeof(BankManager), ServiceLifetime.Transient);
public interface IBankManager : ITransientDependency
{
    int OpenAccount(string clientName);
}

public class BankManager : IBankManager
{
    public int OpenAccount(string clientName)
    {
        // Account creating process...

        return Random.Next(1000000, 9999999);
    }
}
```
* Use extensions methods for `IServiceCollection` or `ServiceDescriptorsBuilder` to retrieve all dependencies from assemblies (see below)

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var assemblies = ... Get all assemblies ex. by ILibraryLoader or Assembly.GetExecutingAssembly(...) etc.
    services.AddFromAssemblies(assemblies);

    // Add MVC services to the services container.
    services.AddMvc();
}
```

There are many custom ways to use service description builder. Most cases described in [customizations](customizations) section.
Also you can find list of additional features [here](features)
