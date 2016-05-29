Below are described common features which provide abstractions part of this library.

## Repleace service
There is possible to override implementaion of service (decorate) which was registered. You can do that by `RepleaceServiceAttribute` (see exapmle below)

```csharp
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

## Create proxy over service
(todo)
