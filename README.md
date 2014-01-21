# Pigeon

*Alpha stage, do not use in production.*

###High performance Actor Model framework.
Pigeon is an unofficial port of the Scala/Java Akka actor model framework.

Why not Akka# or dotAkka? 
I simply assume that TypeSafe that makes Akka don’t want to be associated with spare time projects like this, so I try not to stain their brand name.
Pigeon tries to stay as close to the Akka implementation as possible while still beeing .NET idiomatic.

#####Message Throughput

    Worker threads: 1023
    OSVersion: Microsoft Windows NT 6.2.9200.0
    ProcessorCount: 8
    ClockSpeed: 3392 MHZ
    Actor count, Messages/sec
    2, 7073000 messages/s
    4, 11760000 messages/s
    6, 14534000 messages/s
    8, 18039000 messages/s
    10, 20161000 messages/s
    12, 18785000 messages/s
    14, 17523000 messages/s
    16, 17482000 messages/s
    18, 17931000 messages/s
    20, 18575000 messages/s
    22, 18975000 messages/s
    24, 20920000 messages/s
    ....


## Getting started
Write your first actor:
```csharp
public class Greet
{
    public string Who { get; set; }
}

public class GreetingActor : UntypedActor
{
    protected override void OnReceive(object message)
    {
        Pattern.Match(message)
            .With<Greet>(m => Console.WriteLine("Hello {0}", m.Who));
    }
}
```
Usage:
```csharp
var system = new ActorSystem();
var greeter = system.ActorOf<GreetingActor>("greeter");
greeter.Tell(new Greet { Who = "Roger" });
```
##Remoting
Server:
```csharp
var system = ActorSystemSignalR.Create("myserver", "http://localhost:8080);
var greeter = system.ActorOf<GreetingActor>("greeter");
Console.ReadLine();
```
Client:
```csharp
var system = new ActorSystem();
var greeter = system.ActorSelection("http://localhost:8080/greeter");    
//pass a message to the remote actor
greeter.Tell(new Greet { Who = "Roger" });
```
    
##Code Hotswap
```csharp
public class GreetingActor : UntypedActor
{
    protected override void OnReceive(object message)
    {
        Pattern.Match(message)
            .With<Greet>(m => {
                Console.WriteLine("Hello {0}", m.Who);
                //this could also be a lambda
                Become(OtherReceive);
            });
    }
    
    void OtherReceive(object message)
    {
        Pattern.Match(message)
            .With<Greet>(m => {
                Console.WriteLine("You already said hello!");
                //Unbecome() to revert to old behavior
            });
    }
}
```

##Supervision
```csharp
public class MyActor : UntypedActor
{
    private ActorRef logger = Context.ActorOf<LogActor>();

    // if any child, e.g. the logger above. throws an exception
    // apply the rules below
    // e.g. Restart the child if 10 exceptions occur in 30 seconds or less
    protected override SupervisorStrategy SupervisorStrategy()
    {
        return new OneForOneStrategy(
            maxNumberOfRetries: 10, 
            duration: TimeSpan.FromSeconds(30), 
            decider: x =>
            {
                if (x is ArithmeticException)
                    return Directive.Resume;
                if (x is NotSupportedException)
                    return Directive.Stop;

                return Directive.Restart;
            });
    }
    
    ...
}
```

#####Contribute
If you are interested in helping porting the actor part of Akka to .NET please let me know.

#####Contact
Twitter: http://twitter.com/rogeralsing  (@rogeralsing)

Mail: rogeralsing@gmail.com
