---
title: Definitions
---


Koin Annotations allow to declare the same kind of definitions as the regular Koin DSL, but with annotations. Just tag your class with the needed annotation, and it will generate everything for you!

For example the equivalent to `single { MyComponent(get()) }` DSL declaration, is just done by tagging with `@Single` like this:

```kotlin
@Single
class MyComponent(val myDependency : MyDependency)
```

Koin Annotations keep the same semantic as the Koin DSL. You can declare your components with the following definitions:

- `@Single` - singleton instance (declared with `single { }` in DSL)
- `@Factory` - factory instance. For instances recreated each time you need an instance. (declared with `factory { }` in DSL)
- `@KoinViewModel` - Android ViewModel instance (declared with `viewModel { }` in DSL)

For Scopes, check the [Declaring Scopes](../koin-core/scopes.md) section.

## Automatic or Specific Binding

When declaring a component, all detected "bindings" (associated supertypes) will be already prepared for you. For example, the following definition:

```kotlin
@Single
class MyComponent(val myDependency : MyDependency) : MyInterface
```

Koin will declare that your `MyComponent` component is also tied to `MyInterface`. The DSL equivalent is `single { MyComponent(get()) } bind MyInterface::class`.


Instead of letting Koin detect things for you, you can also specify what type you really want to bind with the `binds` annotation parameter:

 ```kotlin
@Single(binds = [MyBoundType::class])
```

## Nullable Dependencies

If your component is using nullable dependency, don't worry it will be handled automatically for you. Keep using your definition annotation, and Koin will guess what to do:

```kotlin
@Single
class MyComponent(val myDependency : MyDependency?)
```

The generated DSL equivalent will be `single { MyComponent(getOrNull()) }`


> Note that this also works for injected Parameters and properties

## Qualifier with @Named

You can add a "name" to definition (also called qualifier), to make distinction between several definitions for the same type, with the `@Named` annotation:

```kotlin
@Single
@Named("InMemoryLogger")
class LoggerInMemoryDataSource : LoggerDataSource

@Single
@Named("DatabaseLogger")
class LoggerLocalDataSource(private val logDao: LogDao) : LoggerDataSource
```

When resolving a dependency, just use the qualifier with `named` function:

```kotlin
val logger: LoggerDataSource by inject(named("InMemoryLogger"))
```

## Injected Parameters with @InjectedParam

You can tag a constructor member as "injected parameter", which means that the dependency will be passed in the graph when calling for resolution.

For example:

```kotlin
@Single
class MyComponent(@InjectedParam val myDependency : MyDependency)
```

Then you can call your `MyComponent` and pass a instance of `MyDependency`:

```kotlin
val m = MyDependency
// Resolve MyComponent while passing  MyDependency
koin.get<MyComponent> { parametersOf(m) }
```

The generated DSL equivalent will be `single { params -> MyComponent(params.get()) }`


## Injecting a lazy dependency - `Lazy<T>`

Koin can automatically detect and resolve a lazy dependency. Here for example, we want to resolve lazily the `LoggerDataSource` definition. You just need to use the `Lazy` Kotlin type like follow:

```kotlin
@Single
class LoggerInMemoryDataSource : LoggerDataSource

@Single
class LoggerAggregator(val lazyLogger : Lazy<LoggerDataSource>)
```

Behind it will generate the DSL like with `inject()` instead of `get()`:

```kotlin
single { LoggerAggregator(inject()) }
```

## Injecting a list of dependencies - `List<T>`

Koin can automatically detect and resolve all a list of dependency. Here for example, we want to resolve all `LoggerDataSource` definition. You just need to use the `List` Kotlin type like follow:

```kotlin
@Single
@Named("InMemoryLogger")
class LoggerInMemoryDataSource : LoggerDataSource

@Single
@Named("DatabaseLogger")
class LoggerLocalDataSource(private val logDao: LogDao) : LoggerDataSource

@Single
class LoggerAggregator(val datasource : List<LoggerDataSource>)
```

Behind it will generate the DSL like with `getAll()` function:

```kotlin
single { LoggerAggregator(getAll()) }
```

## Properties with @Property

To resolve a Koin property in your definition, just tag a constructor member with `@Property`. Ths is will resolve the Koin property thanks to the value passed to the annotation:

```kotlin
@Single
class MyComponent(@Property("my_key") val myProperty : String)
```

The generated DSL equivalent will be `single { MyComponent(getProperty("my_key")) }`

## Declaring Scopes with @Scope

You can declare definition inside a scope, by using the `@Scope` annotation. The target scope can be specified as a class, or a name:

```kotlin
// scope by type
@Scope(MyScope::class)
class MyComponent

// scope by name
@Scope(name = "MyScopeName")
class MyComponent
```  

The generated DSL equivalent will be:

```kotlin
scope<MyScope> {
  scoped { MyComponent() }
}
// or
scope(named("MyScopeName")) {
  scoped { MyComponent() }
}
```

> You can cumulate `@Factory` or `@KoinViewModel`, to specify a scoped Factory or a ViewModel. Also you can use the `@Scoped` annotation to let define specific bindings on a `@Scope` tagged components.

---
