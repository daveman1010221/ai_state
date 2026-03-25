# Rust Actor Testing Pattern

To test a ractor Actor, always use this pattern:

```rust
#[tokio::test]
async fn test_actor_receives_message() {
    let (actor_ref, handle) = Actor::spawn(
        None,
        MyActor,
        MyActorArgs { ... }
    ).await.unwrap();

    actor_ref.send_message(MyActorMsg::SomeVariant {
        field: value
    }).unwrap();

    tokio::time::sleep(std::time::Duration::from_millis(50)).await;
    actor_ref.stop(None);
    handle.await.unwrap();
}
```

To observe outputs, subscribe an OutputPort before spawning:
```rust
let port = OutputPort::default();
let subscription = port.subscribe(|event| { ... });
let (actor_ref, handle) = Actor::spawn(None, MyActor, MyActorArgs {
    output: Arc::new(port),
    ...
}).await.unwrap();
```

NEVER do this — it won't compile because State is private:
```rust
let state = MyActorState { ... }; // WRONG
```

ALWAYS find the Args struct first:
```bash
rg -n 'pub struct.*Args\|pub enum.*Msg' src/myactor.rs
```

## Mock Actor Pattern

When an Actor's Arguments require TLS certs, env vars, or other complex
dependencies, use a no-op MockActor instead:

```rust
struct MockActor;

#[ractor::async_trait]
impl Actor for MockActor {
    type Msg = BrokerMessage;  // use whatever Msg type is needed
    type State = ();
    type Arguments = ();

    async fn pre_start(
        &self,
        _myself: ActorRef<Self::Msg>,
        _args: (),
    ) -> Result<(), ActorProcessingErr> {
        Ok(())
    }
}
```

Always spawn mock actors with `None` as the name to avoid `ActorAlreadyRegistered`
errors when tests run in parallel.

## Before constructing ANY struct in tests

Read the struct definition before constructing it:
```bash
rg -n "pub struct StructName" src/ --type rust
sed -n '{start},{end}p' src/path/to/file.rs
```

Check for private fields. If ANY field is private, you CANNOT use struct
literal syntax. Use a constructor method instead:
```bash
rg -n "pub fn new\|pub fn from\|pub fn with" src/path/to/file.rs
```

Never assume all fields are public. Always check first.
