# How to plot data points(marks) using Svelte?

So far I have learned how to import data and define the canvas. 

**The most urging topic is on the way: how to plot data points in Svelte.**

You can use `<HTML>`,`<svg>`, `<canvas>` or `<WebGL>`, but I only know so much about `<svg>` so far so today's notes will be largely based on `<svg>`

## 							Svelte's each block

Svelte has an excellent [tutorial](https://svelte.dev/tutorial/each-block-bindings) so I am not going to dive into {#each} block. Here is a brief introduction for you:

1. each block basically loops through an array to create HTML contents.
2. If you want to specify the variables/columns that you intend to use. Try object destructuring(Providing your dataset is an array of objects)

I highly recommend [LiHau's tweet thread for detailed explanation](https://twitter.com/lihautan/status/1411547328931143680).


## Tip1-Svelte Data Viz Mindset: each and DOM

With such limited amount of knowledge, I believe you are ready to plot the data points in Svelte.

**Before you actually code/plot anything, I must tell you that I found it more intuitive to sketch your DOM beforehand.**

Just for the sake of demostration: Here are two data visualizations from [Axios](https://www.axios.com/). I marked different levels with pink arrows.

![featured](/Users/gordontu/Documents/Learning/Data Viz Designer 养成/Svelte Blog/4EachData/featured.png)

#### The first data visualization is [Change in teacher salary, 2010–16](https://www.axios.com/oklahoma-teachers-brace-themselves-6c21da6f-06ef-474c-a2e8-ca84e7d53828.html). Its DOM looks like this:

```html
<svg>		<!-- 1st level -->
	<g class="chart-g">		<!-- 2nd level -->
			<g class="state">		<!-- 3rd level -->
    		<rect></rect>		<!-- 4th level -->
        <text></text>
        <line></line>
    	</g>
    	<g class="state">
    		<rect></rect>
        <text></text>
        <line></line>
    	</g>
    	.
    	.
    	.
    	.etc.
  </g>

</svg>
```
If you are turning it into a Svelte project, you can try:

```html
This is just a demo
<svg>		<!-- 1st level -->
	<g class="chart-g">		<!-- 2nd level -->
  	{#each teacher_data as d,i}
			<g class="state">		<!-- 3rd level -->
    		<rect></rect>		<!-- 4th level -->
        <text></text>
        <line></line>
    	</g>
  	{/each} 
  </g>

</svg>

```

#### The second data visualization is [Thirty years of Atlantic hurricanes](https://www.axios.com/a-history-of-atlantic-hurricanes-2482247577.html)

**DOM**:

```html
<svg>		<!-- 1st level -->
	<g class="chart-g">		<!-- 2nd level -->
			<g>		<!-- 3rd level -->
        <g class="storm">		<!-- 4th level -->
          <path></path>		<!-- 5th level -->
        </g>
        <g class="storm">		<!-- 4th level -->
          <path></path>		<!-- 5th level -->
        </g>
        .
        .
        .
    	</g>		
  </g>
</svg>
```

In analogy, **Svelte demo**:

```html
<svg>		<!-- 1st level -->
	<g class="chart-g">		<!-- 2nd level -->
			<g>		<!-- 3rd level -->
        {#each hurricane_data as d,i}
          <g class="storm">		<!-- 4th level -->
            <path></path>		<!-- 5th level -->
          </g>
    	</g>		
  </g>
</svg>
```

Svelte writes so much like creating DOM elements with your own hands.

Consequently:

1. It is really helpful to know the DOM structure of your desired plot before you actually start coding.
2. You may learn from other people's data viz project by inspecting their DOMs. In fact, this trick actually has been teaching me how to correctly arrange my data and DOM elements correspondly.

## Tip2-Seperate Actual Visual Encodings Component from each block Component

You probably find the demo code I wrote above verbose. **The hierarchy gets longer in your actual projects, and often you even need to write nested each block!**

To keep the project manageable and readable, I often put each block in one component, and the actual SVG elements/visual encodings in another component. In the project below, there is **Rects.svelte** and its child component, **Rect.svelte**.

**Rects.svelte** basically passes `year`, `month` and `value` to **Rect.svelte** and loops through the whole dataset, and then repeats the same numbers of **Rect.svelte** component. 

**Rect.svelte** then receives data and pass the scaled value as properties of `<rect>`.

Therefore, there appears multiple `<rect>` in the DOM. 

<iframe src="https://svelte.dev/repl/de12831b31f64004bc5dc8bfcd2f89d0?version=3.38.3" width="100%" height='600' title="Svelte temperature each demo"></iframe>

### Bonus: SVG dimensions

I often forgot to use CSS to set up the dimensions of `<svg>`, which results in a very limited space for the chart.

Remember: `<style>` in Svelte is scoped, which means it normally only styles the element written/created in the current component.

Here I used width and height property to equal `<svg>`'s width and height to 100% of its parent component.  More customization can be added.

```css
svg {
  width: 100%;
  height: 100%
}
```

