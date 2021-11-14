# Sharing ui components among tests

As the application grows, sooner or later the question of using a design system and common components arises.
<br> The design system forces us to think not with ready-made components provided by the `Android SDK`, but with our
own, which are reused in different parts of the application and significantly speed up the development of new
functionalities.

![alt text](../images/practices/design_system.svg "Sad")

## Problem

Design system introduction raises a lot of custom views usage. Let's review a typical `PageObject` implementation, if
you have a lot of custom views _(Including our own toolbar
`implementation` from the design system)_:

```kotlin

object MainScreen : KaspressoScreen<MainScreen> {


    private val tvToolbarTitle =  KTextView {
        withParent {
           withId(R.id.toolbar_root)  
        }     
        withId(R.id.toolbar_title)
    }
    
    private val ivToolbarImage =  KTextView {
        withParent {
           withId(R.id.toolbar_root)  
        }     
        withId(R.id.toolbar_title)
    }
    
    private val recycler = KRecyclerView( { withId(R.id.recycler_view) }, 
    itemTypeBuilder = {
        itemType(::HeaderItem)
        itemType(::ContactItem)
    }
    
    fun clickContact(name: String) {
       recycler.childWith<ContactItem> {
            withText(name)       
       } perform {
          isVisible()
          click()
       }  
    }
    
    fun assertTitleVisible() {
       tvToolbarTitle.isVisible()
    }
    
    fun assertImageVisible() {
        ivToolbarImage.isVisible()
    }
          
    private class ContactItem(parent: Matcher<View>) : KRecyclerItem<MainItem>(parent) {
        val title: KTextView = KTextView(parent) { withId(R.id.tv_header) }        
    }
    
    private class TitleItem(parent: Matcher<View>) : KRecyclerItem<CheckBoxItem>(parent) {
        val tvHeader: KTextView = KTextView { withId(R.id.tv_header) }
    }    
}

```

We may find the next issues:

- **Readability** <br>
  _In that case really easy to have a mistake with proper matchers_

- **Hard to support** <br>
  _Changing a component can break all use cases in tests. Imagine, `ToolbarView` uses in a hundred tests and described
  in each `PageObject` differently, all of them may be broken_

- **Time consuming** <br>
  _To write such matchers, you need to spend the required amount of time. Developers don't really like writing tests.
  Ideally, this process should be simplified to a minimum._

## Solution

The solution to this problem is exactly the same as for real components.<br>
Each component of the design system must have its own component for UI testing, which encapsulates all matchers inside.

Let's see what our `PageObject` looks like using these components:

```kotlin
object MainScreen : KaspressoScreen<MainScreen> {

    // located now in the test design system module
    private val toolbar =  TToolbarView {  
        withId(R.id.toolbar_component)
    }

    private val recycler = KRecyclerView( { withId(R.id.recycler_view) }, 
    itemTypeBuilder = {
        itemType(::TRowItem) // located now in the test design system module
        itemType(::THeaderItem) // located now in the test design system module
    }

    fun assertTitleVisible() {
       toolbar.title.isVisible()
    }

    fun assertImageVisible() {
       toolbar.image.isVisible()
    }
    
    fun clickContact(name: String) {
       recycler.childWith<TRowItem> {
            withText(name)       
       } perform {
          isVisible()
          click()
       }  
    }

}
```

Such components require a minimum of developer effort to write a `PageObject` and solve all the above issues

## Where to locate such components?

If there is a system in the product, it is most likely located in one or more separated `gradle` modules.

Tests components are recommended to be located in a separate module, which will be used only in the instrumented testing
and will be connected using `androidTestImplementation` in `gradle`. <br>
For instance, if the product design system module is called `design_system`, then in tests you can use the
prefix `ui_tests_design_system`

![alt text](../images/practices/design_system_modules.svg "Test components module scheme")

However, you need also make sure that when adding/modifying a new/old component, the test component is also
created/modified.

This can be guaranteed with code analysis tools such as `Danger` or `Detect`.

## R files problem

When you move test view components to a separate module, if you still use `transitive R files` in your project, you will
have a problem with a wrong `ids` generation.

`Transitive R files` are still used by default. Internally, each module generates own `R file` and re-generate each
dependant resources from other modules.

Unfortunately, using `ids` from production module in the test module, which connected by `androidTestImplementation`
will raise an issue: dependant resources will be generated wrongly and view won't be found in the test. _(Most likely
this is a bug in the Android SDK)_

This problem can be easily solved by migration to `Non-transitive R files`, where we can use already generated `R files`
by other modules.

You can read the details about R files and that problem
here: [The past and the future of Android R class](https://www.mobileit.cz/Blog/Pages/r-class.aspx)