---
layout: post
title: WPF Attached Properties
---

###### 10/18/2018 updated to include ownerType bits and how to make attached property only available for a select set of control types
--------------------------------------------------------------------------------------------------------------------------------------

I worked munging around in wpf today.  There were several behaviors I wanted to attach to some templated controls.
I ended up using attached properties for their ease of consumption.
Behaviors are another option, but require some weird syntax to hook them up to controls, not to mention another one-off dependency (I don't know that there's a good official nuget dependency for `interactivity`.)  

e.g. Attached Behaviors usage - (cumbersome, let's not do it this way)
```xml
    <!-- snip -->
    xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity"
    xmlns:behaviors="clr-namespace:some.namespace.behaviors"
    <!-- snip -->
  <TextBox>
    <i:Interaction.Behaviors>
      <behaviors:TabOnEnter Value="true" />
    </i:Interaction.Behaviors>
  </TextBox>
```


Attached Property usage
```xml
    <!-- snip -->
    xmlns:attachedProperties="clr-namespace:some.namespace.attachedProperties"
    <!-- snip -->
  <TextBox attachedProperties:TextBoxProperties.TabOnEnter="true"/>
```

Attached Property implementation
###### notes:  
###### <mark>ownerType</mark> needs to be where the attached property lives. If something like 'UIObject' is used, and you try to set the attached property value using a binding, you'll get an uncaught xaml parse exception at runtime that kills the application  
###### <mark>argument type</mark> e.g. UIElement, of get/set property filters what control types this attached property applies to  
```c#
public static class AttachedProperties 
  {
    public static readonly DependencyProperty TabOnEnterProperty = DependencyProperty.RegisterAttached(
        name: "TabOnEnter", 
        propertyType: typeof(bool), 
        ownerType: typeof(AttachedProperties),
        defaultMetadata: new UIPropertyMetadata(TabOnEnterPropertyChanged));
    
    public static bool GetTabOnEnter(UIElement obj) => (bool)obj.GetValue(TabOnEnterProperty);
    public static void SetTabOnEnter(UIElement obj, bool value) => obj.SetValue(TabOnEnterProperty, value);

	    static void TabOnEnterPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
	    {
	        var element = d as UIElement;
	        if (element == null) return;
            if ((bool) e.NewValue) element.KeyUp += tabOnEnterKeyUp;
            else element.KeyUp -= tabOnEnterKeyUp;
	    }

	    static void tabOnEnterKeyUp(object sender, KeyEventArgs e)
		{
		    if (e.Key.Equals(Key.Enter))
		    {
		        e.Handled = true;
		        (sender as UIElement)?.MoveFocus(new TraversalRequest(FocusNavigationDirection.Next));
		    }
		}
```
