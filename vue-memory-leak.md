# That time I had a memory leak in a Vue app
I noticed that when I navigated to a certain page in an app I was working on, it would lag for a few moments. 
I opened up Windows task manager out of curiosity and soon noticed that every time I navigated to that page, the memory that Chrome was using went up by several hundred megabytes!

After navigating back and fourth a few times, Chrome soon started using gigabytes of memory.
There was a leak!
I had never had to deal with this sort of problem before so I didn't really know where to start.

I had a vague idea of it being caused by the garbage collector not collecting something because it believed the app still needed it.
A quick search brought up some useful links like [Fix memory problems](https://developers.google.com/web/tools/chrome-devtools/memory-problems) and [Identify js heap memory leaks with allocation timelines](https://developers.google.com/web/tools/chrome-devtools/memory-problems#identify_js_heap_memory_leaks_with_allocation_timelines)
They explain the basics of memory leaks, common causes and Chrome developer tools that can help identify the problem.

But I found that it didn't really help in my case.

Using the chrome dev tools you can do things like take snapshots of all the things in memory at a certain point.

I got into a loop: 
- Navigate to the page with the memory leak problem
- Navigate away to a page without the problem
- Click the button in Chrome dev tools to collect garbage
- Take a memory snapshot and look at the things in memory that are related to the page with the problem (and that should have been collected)

My problem was that there was way too much left in memory after triggering the garbage collection to make much sense of the snapshot

As far as I know there was nothing in Chrome dev tools at the time that let you drill down to the single cause of a leak as massive as this

I had several theories as to what could be causing the problem as there was a few things in the app that were not being handled that well

One thought was that it might be down to circular references in data that was held in app state (in our case, VueX Store) that probably had no business being in global app state anyway

After some time I tried something that led me to the cause of the leak in seconds. It seems blindingly obvious now and will be the first thing I try when faced again with the same problem in a JavaScript web application:

- Go to the parent node for the page with the problem e.g.

```html
<NodeWithMemoryLeak>
  <!-- stuff inside -->
</NodeWithMemoryLeak>
```

- Comment out all markup inside the node
- Navigate to the page in the browser and check if the memory leak still happens
- If it does not, then uncomment the next child node(s). 
- Repeat until you reach the problem node 

If you have one or more sibling nodes below the parent, then comment out each sibling one by one
```html
<NodeWithMemoryLeak>
  <Child1>
    <!-- Stuff -->
  </Child1>
  <!--<Child2>
  </Child2> -->
</NodeWithMemoryLeak>
```


In my case it turned out that once I commented out a custom ```<Select /> ``` component there was no longer a memory leak problem.
When I opened the file an issue that jumped out at me straight away was that the component was adding a click event listener on mount, and not removing that listener on unmount.

Once I added the code to remove the event listener in the component's destroy method and uncommented all the code, the problem went away.


One thing I still don't understand is how one small component prevented everything around it from being collected by the garbage collector



