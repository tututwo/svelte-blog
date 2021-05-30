# For Data Viz Beginners like Me: Two Ways to Import Data in Svelte 

## Synchronous Way: import data from "path/data.csv"

Let's start with the easiest way.

> import data from "./data.json"   // this could be from any path of the desired data file
>
> console.log(data)

Well, now you have the data. Go do your work.

Yes, this is just that simple. Please mind that at most times,  this method is suitable for static data files only.

## Asynchronous Way: onMount

What if your data is on a server and it takes some time to load it. Well, we can get some help from our old friend, d3.js.

1. **Import d3.js**

   > import * as d3 from "d3"

   If you are on REPL, just run the code above. If you are using your code editor, open your terminal first, and install d3.

   > npm install d3 // only if you are using npm

2. **Import data asynchronously**

   Then, if you are fetching your data via a link, and it will possibly take a while. You will want to write an asynchronous function. 

   > let dataset = [];
   >
   > async () => {
   >
   > ​	dataset = await d3.csv('link_or_path_of_dataset'). // d3.csv returns a promise. await "pauses" the execution untill the promise resolves
   >
   > }

3. **Put the function inside onMount**

   Now, onMount is a svelte special. Whatever is inside of onMount will be run once the rest of your code is run, aka "the component is first rendered to the DOM", aka after elements are "mounted".(Or I think of it as: the code your wrote in your .svlete file completes running for the first time.)

   > import { onMount } from "svelte";

   > let dataset = [];
   >
   > onMount(
   >
   > ​	async () => {
   >
   > ​		dataset = await d3.csv('link_or_path_of_dataset'). // d3.csv returns a promise. await "pauses" the execution untill the promise resolves
   >
   > ​	}
   >
   > )

   So the chunk above is basically saying:
   
   " Ok, everyone outside of me(onMount). Keep going. I will catch up with you once you are all 'mounted'"(you know, script is run, css is parsed, and html added to the DOM for the first time)
   
   When other code finishes running, functions within onMount get called. You will change the dataset variable you defined before, and then Svelte's reactivity will happliy change other variables of interest. 
   
   ***Caution***: Do not define the dataset variable inside onMount. It would be scoped in onMount so that you can't refer to it outside of onMount function.



****

### Bonus: Here is just some common measure people adpot with data in hands: Context and Store

So your App.svelte component will not be a lonely island. It passes data between componenets to create more stuff on the web page. To facilitate this "passing-data" process, Svelte offers you **context** and **store**. Those two are basically objects where you store your data and expect Svelte to perform the miracle.

Here is what you do:

1. **Set a context**

> import { setContext } from "svelte"
>
> setContext("chart", dataset)

Done. It is ready to be passed to the current component's descendent components.

There are array, objects, and string etc. in JavaScript. Now here is context in svelte. (I just think context as some object) Why do you need another object called **context** in Svelte? 

Because it is **convenient**. Context data can only be accessed by its child components. Just give it a key parameter and the data you want to pass around. You are ready to go.

You see the first parameter, *key*, here? It's like a phone we give to the parent(papa) component so that child components, only child components, can call him to get the data.(or money if you resonate with this metaphor a lot...) 

*PS: The key parameter can be a variety of things. We are using string here. Please refer to Svelte tutorial for more details.*

Here is the phone, how do child components call the parent then?

2. **get the Context**

> // This is in some Child.svelte
>
> import { getContext } from "svelte"
>
> const dataset = getContext("chart")

Just this easy. Invoke getContext function and pass the "key" parameter we define before. Now you have the data.

If you have followed me here, thank you for your patience and I am sorry that your dataset value is still empty.

[Check Parent1.svelte and Child.svelte](https://svelte.dev/repl/4eb3f32119e044c796919d8bd64d2fa0?version=3.38.2)

**Why do I get no data in the child component using Svelte's Context?**

Ok, on the bright side, we successfully get the data from parent.component, but it is just the very initial 

> let dataset = []

we defined earlier.

**Context is not reactive.** Consequently, when we change the dataset variable, it will not be updated in context. Your context still remembers the good old empty array earlier.

Now, the good thing is that other data visualization experts have created way to make use of context's conveninet data-passing feature, while making it reactive.

**Store**

Store is basically another object. Unlike context being solely available to its descendents, store is accessible to all components. Being "globally accessible", stores in Svelte are really like stores in reality--everyone can come and get the goods(data in our context, no pun intended...)

Stores are reactive, so we can create some stores which you want to save in context.

> const data = writable({})
> // const dimensions = writable({})
> // const scales = writable({})

We use writable() to create a store here because it allows us to modify the store later. And...Here we are going to set the values of each store reactively.

> $: data.set(dataset)   // set the value of store
>
> $: chartContext = {data}   // save store inside an object

> $: setContext('chart', chartContext)

Just like how to defined context above, two more steps are added here. $: tells Svelte to re-run the line when variables inside change, and we save the store inside another object.( If there are multiple stores, it would be convenient to set context later this way)

**Caution**: we are not updating the data inside **context**. We just told Svelte to re-run the line when chartContext, an object of stores, changes.

> // This is Child.svelte
>
> const {data} = getContext('chart').  // we use object destructuring here because we save stores in an object in parent component.

One more thing, when you want to use data in Child component. Please add a dollar sign. For example

> $: console.log($data)

$ before store is just how Svelte recognizes a store. If you want to know more about store and auto-subscription($), please visit the website.

**One more question: if store is generally accessible to every component, why do we even need context?**

As you see above, if there are a lot of reactive data, like dimensions, scales etc., we will define multiple stores. It could be easier to manage all the stores by unique key/id, when they all stored in the same context.

Context is scoped. Things can be **reusable** inside context.

For example, you determine the x coordinates of data points with a scale function, xScale. Then you want to plot the x axis in your Axes component.

> // in Chart.svelte
>
> const {scale} =  getContext('chart')
>
> // in Axes.svelte
>
> const {scale} = getContext('chart')

You import the scale function same way!
