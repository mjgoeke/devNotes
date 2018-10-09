---
layout: post
title: WPF Attached Properties
---

I worked munging around in wpf today.  There were several behaviors I wanted to attach to some templated controls.
I ended up using attached properties for their ease of consumption.
Behaviors are another option, but require some weird syntax to hook them up to controls, not to mention another one-off dependency (I don't know that there's a good official nuget dependency for `interactivity`.)  

e.g. Attached Behaviors usage - <mark>(cumbersome, let's not do it this way)</mark>
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
```c#
public static class TextBoxProperties  // generally grouped by control type of set of behaviors 
  {
    public static readonly DependencyProperty TabOnEnterProperty = DependencyProperty.RegisterAttached("TabOnEnter", typeof(bool),
        typeof(TextBoxProperties), new UIPropertyMetadata(TabOnEnterPropertyChanged));
    
    public static bool GetTabOnEnter(DependencyObject obj) => (bool)obj.GetValue(TabOnEnterProperty);
    public static void SetTabOnEnter(DependencyObject obj, bool value) => obj.SetValue(TabOnEnterProperty, value);

    static void TabOnEnterPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
      if (d is UIElement element)
      {
        if ((bool)e.NewValue) element.KeyDown += Keydown;
        else element.KeyDown -= Keydown;
      }
    }

    static void Keydown(object sender, KeyEventArgs e)
    {
      if (e.Key.Equals(Key.Enter))
        (sender as UIElement)?.MoveFocus(new TraversalRequest(FocusNavigationDirection.Next));
    }
```
