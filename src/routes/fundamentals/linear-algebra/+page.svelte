<script lang="ts">
    import VectorsMatching from '$lib/components/Figures/fundamentals/linear-algebra/vectors_matching.svelte';
    import VectorAdding from '$lib/components/Figures/fundamentals/linear-algebra/vector_adding.svelte';
    import VectorScaling from '$lib/components/Figures/fundamentals/linear-algebra/vector_scaling.svelte';
    import MatrixMultiplication from '$lib/components/Figures/fundamentals/linear-algebra/matrix_multiplication.svelte';
    import Katex from '$lib/components/Math/katex.svelte';
</script>

<p class="article-p">
    Linear algebra is the field of math concerned with <em>vectors</em> and <em>linear transformations</em>. Knowledge of linear algebra is very generally useful, and finds a large variety of applications, including in physics, computer science as well as in digital art, modelling and design.
</p>
<p class="article-p">
    Firstly, what is a vector? One way to think of a vector is as a list of numbers. For example, <Katex math={'\\left[\\begin{smallmatrix}2\\\\4\\end{smallmatrix}\\right]'} />, <Katex math={'\\left[\\begin{smallmatrix}.36\\\\.48\\\\.8\\end{smallmatrix}\\right]'} /> and <Katex math={'\\left[\\begin{smallmatrix}0\\end{smallmatrix}\\right]'} /> are all vectors. Vectors can be summed or subtracted by adding or subtracting each components together. For example, <Katex math={'\\left[\\begin{smallmatrix}2\\\\4\\end{smallmatrix}\\right] + \\left[\\begin{smallmatrix}3\\\\6\\end{smallmatrix}\\right] = \\left[\\begin{smallmatrix}5\\\\10\\end{smallmatrix}\\right]'} />, and <Katex math={'\\left[\\begin{smallmatrix}2\\\\4\\end{smallmatrix}\\right] - \\left[\\begin{smallmatrix}3\\\\6\\end{smallmatrix}\\right] = \\left[\\begin{smallmatrix}-1\\\\-2\\end{smallmatrix}\\right]'} />. Vectors can also be multipled by single numbers by multiplying each component by the number. For example, <Katex math={'3 \\times \\left[\\begin{smallmatrix}2\\\\4\\end{smallmatrix}\\right] = \\left[\\begin{smallmatrix}6\\\\12\\end{smallmatrix}\\right]'} />. When singular numbers are used this way, they are referred to as <em>scalars</em>. Scalars also allow us to write the vector <Katex math={'\\left[\\begin{smallmatrix}-1\\\\-2\\end{smallmatrix}\\right]'} /> as <Katex math={'-1 \\times \\left[\\begin{smallmatrix}1\\\\2\\end{smallmatrix}\\right]'} />, or just <Katex math={'-\\left[\\begin{smallmatrix}1\\\\2\\end{smallmatrix}\\right]'} />.
</p>
<p class="article-p">
    Vectors can also be interpreted as coordinates in some n-dimensional space. Under this interpretation, the first component of the vector represents the x-coordinate of the point, the second component the y-coordinate, the third, the z-coordinate, etc. As opposed to points, vectors are typically drawn as an arrow from the origin to the point in space they represent. In the demo below, the red axis represents the x-axis, the green axis represents the y-axis, and the blue axis represents the z-axis. Try writing the coordinate of each coloured vector:
</p>
<VectorsMatching />
<p class="article-p">
    Unlike in the demo, the z-axis is usually the axis pointing upwards in math contexts. However, computational implementations vary. The library used to build these demonstrations, Three.js, happens to use the y-axis as the vertical one. In machine learning applications, the geometric interpretation of vectors isn't important, but a geometric intuition of the operations used can help intuit logic behind algorithms.
</p>
<p class="article-p">
    Under the geometric interpretation of vectors, a more intuitive idea of addition and scalar multiplication can be formed. The result of adding two vectors is equivalent to the vector gotten by putting the base of one vector at the tip of another.
</p>
<VectorAdding />
<p class="article-p">
    Similarly, scalar multiplication can be interpreted as "scaling" the vector along its current direction. A negative scale would make the vector go the other direction.
</p>
<VectorScaling />
<p class="article-p">
    We have addition and scaling of vectors, but what about products between vectors? Sadly, there is no good analog for vector-vector multiplication whose result is a vector. There are still two products that find common use in machine learning: The dot product and the hadamard product. The hadamard product is straightforward: the hadamard product of two vectors <Katex math={'\\mathbf{\\vec{a}} = \\left[\\begin{smallmatrix} x_1 \\\\ y_1 \\\\ z_1 \\end{smallmatrix}\\right]'} /> and <Katex math={'\\mathbf{\\vec{b}} = \\left[\\begin{smallmatrix} x_2 \\\\ y_2 \\\\ z_2 \\end{smallmatrix}\\right]'} />, denoted as <Katex math={'\\mathbf{\\vec{a}} \\odot \\mathbf{\\vec{b}}'} />, is equal to <Katex math={'\\left[\\begin{smallmatrix} x_1 x_2 \\\\ y_1 y_2 \\\\ z_1 z_2 \\end{smallmatrix}\\right]'} />. The dot product is equal to the sum of each component of the hadamard product and is denoted <Katex math={'\\mathbf{\\vec{a}} \\cdot \\mathbf{\\vec{b}} = x_1 x_2 + y_1 y_2 + z_1 z_2'} />. The dot product's result is a scalar, as opposed to a vector. The dot product is also equivalent to <Katex math={'\\| \\mathbf{\\vec{a}} \\| \\| \\mathbf{\\vec{b}} \\| \\cos \\theta'} />, where <Katex math={'\\theta'} /> is the angle between the two vectors.
</p>
<p class="article-p">
    Another mathematical object just as important as the vector is the matrix. In short, a matrix is a list of vectors. A matrix containing vectors <Katex math={'\\left[\\begin{smallmatrix} x_1 \\\\ y_1 \\\\ z_1 \\end{smallmatrix}\\right]'} /> and <Katex math={'\\left[\\begin{smallmatrix} x_2 \\\\ y_2 \\\\ z_2 \\end{smallmatrix}\\right]'} /> would look like <Katex math={'\\left[\\begin{smallmatrix} x_1 & x_2 \\\\ y_1 & y_2 \\\\ z_1 & z_2 \\end{smallmatrix}\\right]'} />. You may also notation referring to specific elements of a matrix as <Katex math={'M_{i,j}'} />. The first number refers to which row the element, and the second refers to the column, so a matrix would look like <Katex math={'\\left[\\begin{smallmatrix} M_{1,1} & M_{1,2} \\\\ M_{2,1} & M_{2,2} \\\\ M_{3,1} & M_{3,2} \\end{smallmatrix}\\right]'} />.
</p>
<MatrixMultiplication />