<script lang="ts">
    import PTGraph from '$lib/components/Figures/fundamentals/calculus/p_t_graph.svelte';
    import PTRotGraph from '$lib/components/Figures/fundamentals/calculus/p_t_rot_graph.svelte';
    import DpDtGraph from '$lib/components/Figures/fundamentals/calculus/dp_dt_graph.svelte';
    import Katex from '$lib/components/Math/katex.svelte';
</script>

<p class="article-p">
    You might already be familiar with calculus, but even if you've completed high school, there's a bit more involved with the calculus used in neural networks and the like.
    This article mainly focuses on <em>differential</em> calculus, which is basically always required in what is called <em>gradient descent</em>. Integral calculus, although it has its uses, is much less common in machine learning. 
</p>
<p class="article-p">
    Differential calculus is the field of study concerned with rates of change. Essentially, differential calculus tells us how quickly one variable changes with respect to another variable. For example, consider the relationship between <Katex math={'p'} /> and <Katex math={'t'} /> shown by the graph below (Hover over the graph or click on it on mobile!):
</p>
<PTGraph />
<p class="article-p">
    You can define the rate of change between two points to be the difference in one variable divided by the difference in another variable. For example, if you travelled 10 meters in 2 seconds, then the rate of change of your position, also known as your speed, would be 5 m/s. The slider below changes the value of the distance between the two values of <Katex math={'t'} />. Click on the graph to lock and unlock the position of t!
</p>
<PTRotGraph />
<p class="article-p">
    Here, we have a delta term to denote the difference between the two points that we use to calculate the rate of change. <Katex math={'\\Delta'} /> cannot be zero, because that would result in division by zero, but the ratio <Katex math={'\\frac{p_2 - p_1}{\\Delta}'} /> seems to be stable for small values of <Katex math={'\\Delta'} />. In some cases, it is well defined enough that we can define the rate of change at a point to be the value of this ratio as <Katex math={'\\Delta'} /> gets smaller. The derivative of a variable is percisely this: the rate of change of a variable over a given domain at each point.
</p>
<DpDtGraph />
<p class="article-p">
    The straight yellow line is known as the <em>tangent</em> line to the graph, and the slope of the line represents the rate of change of the variable at that point. The value of the slope is given by the yellow curve, the derivative: when the tangent is steepest, the derivative is at its peak or trough, and when the tangent is flat, the derivative is at zero.
</p>
<p class="article-p">
    The derivative of a variable <Katex math={'f'} /> with respect to a variable <Katex math={'x'} /> is denoted <Katex math={'\\frac{df}{dx}'} />. To denote the value of the derivative at a specific point <Katex math={'a'} />, you would use <Katex math={'\\frac{df}{dx}|_{x=a}'} />. Alternatively, if <Katex math={'f'} /> is a function of  <Katex math={'x'} />, notated as <Katex math={'f\\left(x\\right)'} />, then the derivative as a function of <Katex math={'x'} /> can be written as <Katex math={'f\'\\left(x\\right)'} />.
</p>
<p class="note article-p">
    Although a lot of functions have well-defined derivatives, some functions do not! Consider the function <Katex math={'\\left|x\\right|'} /> at <Katex math={'x = 0'} />. If <Katex math={'\\Delta'} /> is positive, then the slope will be equal to 1, but if <Katex math={'\\Delta'} /> is negative, then the slope will be equal to -1! Also, a function can have a derivative at a point even if it doesn't have a defined value at the point!
</p>
<p class="article-p">
    There are to extra considerations for derivatives in machine learning: the <em>partial derivative</em> and the <em>gradient</em> operator. A partial derivative is similar to a normal derivative, in that it is the rate of change of one variable with respect to another. However, the difference between the two is that a partial derivative is the rate of change when the variable depends on multiple variables. For example, taking the derivative of the function <Katex math={'u = \\sin\\left(x\\right)\\sin\\left(y\\right) + x^2y'} /> with respect to x yields a partial derivative. The reason this is significant is because the derivatve now depends on more variables than just the variables the rate of change is being taken to. Using the previous example, the derivative of <Katex math={'u'} /> w.r.t. <Katex math={'x'} /> is <Katex math={'\\frac{\\partial u}{\\partial x} = \\cos\\left(x\\right)\\sin\\left(y\\right) + 2xy'}/>. The notation used for the partial derivative of a variable <Katex math={'f'} /> with respect to a variable <Katex math={'x'} /> is <Katex math={'\\frac{\\partial f}{\\partial x}'} />. 
</p>
<p class="article-p">
    One final important concept is that of the gradient. If you understand partial derivatives, it's straightforward enough: the gradient represnts all of the partial derivatives of a variable with respect to each of its inputs. Once again, using our function <Katex math={'u = \\sin\\left(x\\right)\\sin\\left(y\\right) + x^2y'} />, the gradient of <Katex math={'u'} /> would be <Katex math={'u = \\left[\\begin{smallmatrix}\\cos\\left(x\\right)\\sin\\left(y\\right) + 2xy\\\\\\sin\\left(x\\right)\\cos\\left(y\\right) + x^2\\end{smallmatrix}\\right]'} />. The result is a vector, as opposed to a single number as with the derivative and partial derivative. The first component of the vector here is <Katex math={'\\frac{\\partial u}{\\partial x}'} />, and the second component is <Katex math={'\\frac{\\partial u}{\\partial y}'} />. This can be extended for an arbitrary number of input variables, and the resulting vector will have a component for each input. The order of the components is somewhat arbitrary, and varies depending on how the inputs are being used.
</p>