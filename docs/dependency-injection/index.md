# AE.Extensions.DependencyInjection
[![Build Status](https://travis-ci.org/app-enhance/ae-di.svg?branch=master)](https://travis-ci.org/app-enhance/ae-di)
[![Build status](https://ci.appveyor.com/api/projects/status/s5ej8f3uechsx3gs/branch/master?svg=true)](https://ci.appveyor.com/project/Ermesx/ae-di/branch/master)
[![app-enhance-dev MyGet Build Status](https://www.myget.org/BuildSource/Badge/app-enhance-dev?identifier=891bb83e-b009-4793-b622-495a6eab6afc)](https://www.myget.org/gallery/app-enhance-dev)

This is an extension library for `Microsoft.Extensions.DependencyInjection` which provides you with a declarative way to pick out lifetime of services and register them in just a few lines.

## Motivation
When you want to register a service (create dependency) in `IServiceCollection` you have to describe at least 3 things.

* Service interface
* Service implementation
* Lifetime scope

Ususally it should look like this (see below)
```c#
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
```c#
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

I'm inspired by how Orchard team resolves that issue. 
They created interfaces which correspond to lifetime scopes. So every service interface inherits appropriate "scope interface" and describes dependencies to register in global container.
In my case it looks like below - interfaces correspond exactly to `ServiceLifetime` enum.

```c#
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
## Goals/Features

- [x] [Declarative way of defining lifetime scope](https://github.com/app-enhance/ae-di#how-does-it-work)
- [x] [Retrieval servies from assemblies](https://github.com/app-enhance/ae-di#how-does-it-work)
- [ ] [Repleace dependencies (Decorate)](https://github.com/app-enhance/ae-di#repleace-dependency)
  - [x] Repleace already registered services
  - [x] Repleace dependencies from declarative way
  - [ ] Add wrapper over IServiceCollection to lazy registration
- [ ] [Proxing whole interfaces](https://github.com/app-enhance/ae-di#create-proxy-over-service)
- [ ] [Integrations with popular containers](https://github.com/app-enhance/ae-di#integrations)
  - [ ] [Autofac](https://github.com/app-enhance/ae-di#autofac)
  - [ ] [Ninject](https://github.com/app-enhance/ae-di#ninject)

## How does it work?

In order to use this approach you have to do two things:

* Select dependency interface and inherit
```c#
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
```c#
public void ConfigureServices(IServiceCollection services)
{
    var assemblies = ... Get all assemblies ex. by ILibraryLoader or Assembly.GetExecutingAssembly(...) etc.
    services.AddFromAssemblies(assemblies);
 
    // Add MVC services to the services container.
    services.AddMvc();
}
```

There are many custom ways to use service description builder. Most cases described [here](https://github.com/app-enhance/ae-di/wiki/Custom-usage-of-service-descriptions-builder)

### Repleace service
There is possible to override implementaion of service (decorate) which was registered. You can do that by `RepleaceServiceAttribute` (see exapmle below)
```c#
[RepleaceService(typeof(BankManager))]
public class AuditBankManager : BankManager
{
    // Suppose that OpenAccount is virtual
    public override int OpenAccount(string clientName)
    {
        // Do audit...
        
        var accountNumber = base.OpenAccount(clientName);
        
        // Do more audit...
        
        return accountNumber;
    }
}
```

Repleace dependency works also with services alredy added to `IServiceCollection`. TODO: possibility to repleace services added after using this extension.

There is another attibute `DecorateServiceAttribute` which works the same (inherit of `RepleaceServiceAttribute`). It is introduced due to semantics and in order to improve code cleanliness.

### Create proxy over service
(todo)

## Integrations

### Autofac 
(todo)

### Ninject 
(todo)

## Contribute
Before pushing new feature or improvement please read [CONTRIBUTING.md](https://github.com/app-enhance/ae-core/blob/master/CONTRIBUTING.md)
