
# When and how to write reusable components in Vue 
Designing reusable components is an important part of writing modern web apps. 
Many unrelated parts of the app may share features and styles. If you write reusable components it can save time when adding and updating features.
It can also make your app easier to test. 

In this article I want to explain the benefits of writing reusable components and one way of writing them. 
The examples are written in Vue 2, but the same principles can be applied to React components

## An example
Let's say you want to build a graph to show how many apples you have compared to how many oranges. 

![graph](https://raw.githubusercontent.com/liamsain/liamsain.github.io/master/img/graph.PNG)

*Note: I will not be dealing with how to implement a graph like this, just how to write it in a reusable way from a props perspective*

For 2 apples and 8 oranges, 20% of your fruit are apples, 80% oranges.
The curve fills the circle based on these percentages

The graph also has a key to the right, an icon in the center, coloured curves, and percentages at the point in the circle where the curve stops

## A naive implementation
You could write an 'OrangeApplesComparison' component, put the SVG markup and CSS code in there, slap some props on it like 'orangesCount' and 'applesCount' and be done with it.

```html
<OrangeApplesComparison 
  :orangesCount="8"
  :applesCount="2"
/>
```

However, if you write it in this way, you will run into trouble later if you want to compare different data using the same graph style.

One approach I've seen some developers take is to copy paste the internals of the component and change the prop names to suit a different set of data. Another use of the graph might be to compare the amount of different staff members:


```html
<DoctorsNursesComparison
  :nurses="3"
  :doctors="2"
/>
```
This approach is rarely a good idea, as it leads to duplicate component code and unit tests. Also if you wanted a change in the component to be reflected in all instances, you would need to update several components, which is tedious and error prone.

## A reusable implementation
Instead, you could create a reusable component that allows users of that component to customise everything, including:
- The colour of the curves
- The data being compared
- The icon in the center of the graph 

First things first: naming your reusable component. 'OrangeApplesComparison' is not a great name for this component if we want to reuse it elsewhere.

When coming up with names for elements and functionality you need to think about what they are or what they do on a more fundamental level.
It's a graph so we probably want to use the word 'graph' somewhere in the component's name. It's also round, or circular, so maybe 'CircularGraph'? Good enough 
```html
<CircularGraph />
```

The next thing to sort out is the component's props. This is a little tricky as we want to be able to customise multiple things per item. i.e. for apples, we want to be able to specify the number, the main curve colour, the background curve colour and the label (for the graph key). 
We can use an object to group this data together:
```javascript
{
  label: 'Oranges',
  count: 2,
  activeColour: '#ffaa00',
  backgroundColour: '#ffe8ba'
}
```

For multiple graph items, group them together into an array:
```javascript
const fruitGraphItems = [
  {
    label: 'Oranges',
    count: 2,
    activeColour: '#ffaa00',
    backgroundColour: '#ffe8ba'
  },
  {
    label: 'Apples',
    count: 4,
    activeColour: '#03fc35',
    backgroundColour: '#d1ffda'
  }
];
```
If you make your circular graph expect this kind of data, it will be much easier to reuse
```html
<CircularGraph :items="fruitGraphItems" />
```

Now the component can use the count prop on each object to work out percentages, and use the colours provided via props, rather than hard-coded values.

Next, the icon in the center. This can be done a few ways. If your application uses an icon library like Font Awesome then it could take a unicode string prop:

```html
<CircularGraph :items="items" iconUnicode="f080" />
```

It could also make use of named slots:
```html
<CircularGraph :items="items">
  <div slot="icon">
    <!-- place icon here -->
  </div>
</CircularGraph>
```

Lastly, in the graph's key labels it mentions 'fruit': '8 fruit'. A good prop name for this might be 'subject', since fruit is the 'subject' of the graph:

```html
<CircularGraph :items="items" :subject="subjectOfGraph">
  <div slot="icon">
    <!-- place icon here -->
  </div>
</CircularGraph>
```

Now when we wish to reuse the graph for different data, you can write a wrapper component around the circular graph
```html
<OrangeApplesComparison 
  :orangesCount="8"
  :applesCount="2"
/>
```

We end up with the same component we started with. Except now there won't be much inside this component apart from creating the data needed for our new reusable CircularGraph:
```html
<template>
  <CircularGraph :items="fruitItems" subject="fruit">
    <div slot="icon">
      <!-- place icon here -->
    </div>
  </CircularGraph>
</template>

```
```javascript
computed: {
  fruitItems() {
    return [
      {
        label: 'Oranges',
        count: this.orangesCount,
        activeColour: '#ffaa00',
        backgroundColour: '#ffe8ba'

      },
      {
        label: 'Apples',
        count: this.applesCount,
        activeColour: '#03fc35',
        backgroundColour: '#d1ffda'
      }
    ];
  }
}
```

OrangeApplesComparison has become a simple wrapper for the CircularGraph component

## Benefits of reusable components
- Because I wrote the component's props to be generic it led to additional functionality: the graph can compare more than two things. It would display curves for however many items were given to it via props. It probably doesn't make sense to give it more than four or five in this case, but it's better than being restricted to just two, which is perhaps what designing the component naively would have led to
- It can make it easier to add functionality to an app
- It can reduce duplicate component and unit test code
- Once features are added to a reusable component, all components that consume it get that new feature at the same time


## Difficulties when writing reusable components
### Naming things
Writing components in a reusable way can make it more difficult to name your props, data and components.
My advice here is to try to reduce each element of your component to its most simplest and generic idea.

### Data structures
Writing reusable components sometimes means greater complexity in terms of the structure of the data you pass into the component via props

## When to write reusable components
It's often a matter of taste, judgement and practicality. Some things naturally lend themselves better to being written in a reusable way. 
Sometimes it is better to wait til you've implemented the same layout or functionality a few times before trying create a reusable component.

Some rules of thumb as to when you might want to write a component in a reusable way:
- When it seems likely that you will need to reuse the same style or functionality elsewhere in the app
- When writing complex functionality that will be more understandable, maintainable and testable if broken down into small generic components
- When the thing you are creating seems inherently non-specific, like a graph

## When not to write reusable components
- When the functionality and layout are simple and are not going to be used again
- When the component is mainly a collection of other reusable components


## Risks
It is sometimes easy to get carried away and write a reusable component that is used in many places that each expect the component to behave in slightly different ways. 

I've found that flags that change the look (and css classes) of elements in the component are OK, but as the 'if' statements start to rack up it might be best to create new, smaller reusable components (which may, in turn, use parent component containing common styles and logic)

One of the benefits of reusable components can also cause trouble. If you update one because one of its consumers requires specific functionality, you have to be careful not to break all other components that may be consuming the reusable component. 
Try to keep this in mind when deciding whether to write a component in a reusable way, and when choosing which other components are going to consume it

[test](/vue-memory-leak)