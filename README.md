# Unity 3d Style Guidelines

## Unity project setup
* To setup a new Unity 3d project for git, select `Edit -> Project Settings -> Editor` from the menu, and then:
	1. Under *Version Control*, set _Mode_ to 'Visible Meta Files'
	2. Under *Asset Serialization*, set _Mode_ to 'Force Text'

This will ensure that generated .meta files are visible to git, and will be checked into source control. It also makes sure that all assets are serialised as text, rather than binary, which makes merging easier and dealing with merge conflicts feasible rather than impossible (as it is when pure binary serialisation is used).

## Unity C# Code Templates

The location for the default script templates is:
* Windows: C:\Program Files\Unity\Editor\Data\Resources\ScriptTemplates on Windows
* Mac: /Applications/Unity/Unity.app/Contents/Resources/ScriptTemplates/
* Linux: /opt/Unity/Editor/Data/Resources/ScriptTemplates

To match the coding guidelines, it's worth changing the default MonoBehaviour template to:
```
using UnityEngine;

public class #SCRIPTNAME# : MonoBehaviour
{
    void Start()
    {
        
    }

    void Update()
    {
        
    }
}
```

## Code formatting and structure

* Write all scripts in C# exclusively. UnityScript is considered Legacy at this point.

* Follow [Unity's own C# style guidelines](http://wiki.unity3d.com/index.php/Csharp_Coding_Guidelines), with the following exceptions:
    * Indent with 4 spaces, not tabs.
    * `if`/`while`/`for`/ etc should always use braces (Unity's own advice is contradictory in this regard)
    * Always include documentation for public interfaces/classes/structs (type `///`` in VS to generate the boilerplate automatically).
    * Class members don't need to be alphabetized. Use whichever order makes things clearest.
 
## Unity Specific considerations
* Put everything inside a top-level namespace. This avoids clashes with your own project code and 3rd-party libraries.
* Favour pure .NET C# for model objects and business logic wherever possible (over `MonoBehviour` subclasses or other things that dependy on `UnityEngine`).
* For dependency management, serialize references in the inspector (or interfaces via the Full Inspector asset).
* Enable full Unity inspector serilisation by inheriting from the pure C# model objects. (e.g. `Weapon` becomes `UnityWeapon` with Unity's serialisation attiritubes added to the class' variables.
* `GameObject.Find()` is a code smell. If you think you need it, you can probably improve your design to remove it.
* Use `[RequireComponent(typeof(<Type>))]` to enforce component dependencies in scripts.
* Use `[SerializeField]` on private member fields to expose them in the inspector, rather than making fields public.
* Use `Action<>` and `Func<>` based callbacks and lambdas or the event/delegate pattern rather than Coroutines for async methods. This provides proper asynchronous operation, rather than the main-thread-based coroutine pattern.
	```
    (todo: include examples for each approach).
    ```
* Use the [humble object pattern](http://blogs.unity3d.com/2014/06/03/unit-testing-part-2-unit-testing-monobehaviours/) to enable testing of Monobehaviours.
* Use Assertions available in `Unity.Assertions.Assert` to verify expected conditions at runtime. Log an error to the console on assertion failures.
* Avoid stringly-typed code. Don't refer to objects or methods using strings, as this is brittle and incompatible with IDE refactoring tools.
* For the exceptions to the above, (the few things that can only be accessed by name in Unity), define the required strings as constants in static classes, e.g. `LayerNames` or `AnimationNames`. Use nested classes here to provide organisation if needed, e.g. `AnimationNames.Monster.Die`
* Instantiated objects should be placed under an empty parent object to avoid cluttering the scene hierarchy, call it somethign sensible like `DynamicObjects`.

* Use the defensive GetComponent alternative:
```
public static T GetRequiredComponent(this GameObject obj) where T : MonoBehaviour
{
   T component = obj.GetComponent();
 
   if(component == null)
   {
      Debug.LogError("Expected to find component of type " 
         + typeof(T) + " but found none", obj);
   }
 
   return component;
}
```

This avoids null reference issues where you can't enforce components with `RequiresComponent` (e.g. in 3rd party scripts).

* Use ScriptableObjects for scene specific data; to configure components in the inspector; and to specialise generic prefabs.

## Prefabs

* Use generic prefabs for everything. Two similar objects should be two prefabs with different configuration.
* Link prefabs to prefabs; do not link instances to instances. Links to prefabs are maintained when dropping a prefab into a scene; links to instances are not. Linking to prefabs whenever possible reduces scene setup, and reduce the need to change scenes. This is also one solution to Unity's lack of nested prefabs.

## Documentation

Only certain things need to be documented outside of your project code:

* Tags (what they are and what they are used for).
* Layers (i.e. which things go in what layer, and what they are used for).
* GUI depths for layers (what should display over what).
* Scene setup (if creating a new scene involves several steps and is non-obvious).
* Prefab structure of complex prefabs.

#### Inspector design for components
Use property drawers to make fields more user-friendly. Property drawers can be used to customize controls in the inspector. This allows you to make controls that better fit the nature of the data, and put certain safe-guards in place (such as limiting the range of the variables). Use the `Header` attribute to organise fields, and use the `Tooltip` attribute to provide extra documentation to designers. 

## Third-Party Assets

Create a clean project and import the asset into that, then re-export a clean version of the asset. This avoids potential problems with assets overwriting files in your own project (e.g. Standard Assets, etc). It also helps you to organise the 3rd-party files how you want, and allows you to remove any superfluous code and assets (e.g. examples).

Workflow for this:

  1. Make new project, import asset.
  2. Verify examples are working.
  3. Move files to organise suitably:
    * Put all files under a single folder if they aren't alreadt, to keep them isolated when you import later.
    * Remove anything that will clash with your existing project.
  4. Re-run the examples to make sure the asset still functions as expected.
  5. Delete all example assets, scenes, scripts.
  6. Verify that everything still compiles and that prefabs all still have their serialised references.
  7. Select all assets and export a package.
  8. Import your new package into existing project.

## External Resources - required reading

[http://www.gamasutra.com/blogs/HermanTulleken/20160812/279100/50_Tips_and_Best_Practices_for_Unity_2016_Edition.php](http://www.gamasutra.com/blogs/HermanTulleken/20160812/279100/50_Tips_and_Best_Practices_for_Unity_2016_Edition.php)

[http://unity3d.com/learn/tutorials/topics/best-practices](Unity Best Practices articles)

