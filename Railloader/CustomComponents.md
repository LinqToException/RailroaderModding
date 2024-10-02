# Custom Components

Railloader 1.9 added the ability to more easily add custom components for rolling stock, scenery, and whatever else is supported by the components system.
By registering a component with RL, it takes care of the whole behind-the-scenes requirements. This document outlines the necessary steps to get it going.

It will feature a simple dummy component.

## Component Model

Each component features a POCO-ish structure that serves as a model, which is the deserialized JSON. It's both the entry point to make it available in e.g.
the editor, as well as providing the necessary data from the JSON after its deserialization.

```cs
using Model.Definition;

[Component(ComponentDefinitionMask.Car, ComponentLifetime.Model)]
internal class MyComponent : Component
{
    // Used for registering too and must match; therefore outsourced into a constant
    internal const string _Kind = "MyMod.MyComponent";
    public override string Kind => _Kind;

    // Optionally: Define parameters (that are JSON serializable).
    // Order can be used to arrange them in the editor, but is otherwise functionally useless.
    [DefinitionProperty(Order = 500)]
    public string AdditionalProperty { get; set; }
}
```

The `ComponentDefinitionMask` defines which entity (types) it is displayed for in the editor. It has no filtering function itself; you could theoretically add
a car component to a piece of scenery (with likely negative effects).

THe `ComponentLifetime` actually defines when the `ComponentBuilder` is invoked. The game may unload models (e.g. for objects that are very far away), and then
re-load them later. When creating components that interact with the model (e.g. manipulate it in some way), using the model lifetime is probably the better way
to go, as it will assure your code is (re-)executed whenever the model appears.

## Component Builder

```cs
using Model;
using Railloader.Extensions;

internal class MyComponentBuilder : ComponentBuilder<MyComponent>
{
    protected override void Build(ComponentBuilderContext ctx, MyComponent component)
    {
        // Setup the component here, using `ctx` as reference for the object being created.
        // Note that not all fields in `ctx` are populated for all entity types;
        // as example, scenery does not have most (or any) properties besides `GameObject` available.
        // GameObject will usually be a child of the GO you're attached to, as each component is instantiated with its own child GO.
        // To get e.g. the Car component, you could use `ctx.GameObject.GetComponentInParent<Car>()`.
    }
}
```

## Registration

In order for the component to be recognized by the game, it needs to be registered. This is preferably done in your plugin constructor, or `OnEnable`, depending
on your mod's architecture. This example will assume that it's necessary to have your component loaded all the time, and if it's disabled, it will do some kind
of self-disabling at the component/builder level.

The registration is done on the `IModdingContext`, which can be injected into your plugin's constructor by Railloader.

```cs
using Railloader;

publiy class MyPlugin : PluginBase
{
    public MyPlugin(IModdingContext moddingContext)
    {
        moddingContext.RegisterComponent<MyComponent, MyComponentBuilder>(MyComponent._Kind);
    }
}
```
