# An exploration of various phyllotaxis algorithms

## The planar phyllotaxis algorithm

As we explore the fascinating world of phyllotaxis, we'll start with a fundamental concept: the planar phyllotaxis algorithm. This mathematical model helps us understand the intricate patterns found in nature, from the arrangement of leaves on a stem to the branching of trees.

So, what is phyllotaxis, and how does the planar phyllotaxis algorithm work? Simply put, phyllotaxis is the study of the geometric patterns that occur in the growth of plants. The planar phyllotaxis algorithm is a mathematical formula that generates a two-dimensional spiral pattern, mimicking the way leaves or branches grow in a circular arrangement.

Throughout this guide we'll use vanilla javascript and three.js to implement these algorithms. 

Setting up a three.js project is out of scope for this exploration. We'll assume we have a basic scene setup and a function called `makeSphere` which can create a sphere centered around a 3D point in space, with a particular radius.


```javascript
/**
 * Planar phyllotaxis algorithm.
 * 
 * @param {object} options
 * @param {number} options.organs - The number of organs in the arrangement.
 * @param {number} [options.divergenceAngle=Math.PI * (3 - Math.sqrt(5))] - The divergence angle between organs (in radians). Defaults to the golden angle.
 * @param {number} options.radiusConstant - The radius constant for the phyllotaxis arrangement.
 * @return {Object[]} - An array of 2D points (with z = 0) representing the phyllotaxis arrangement.
 */
const planarPhyllotaxis = ({organs, divergenceAngle = Math.PI * (3 - Math.sqrt(5)), radiusConstant}) => {
  const points = [];
  for (let index = 0; index < organs; index++) {
    const angle = index * divergenceAngle;
    const radius = radiusConstant * Math.sqrt(index);
    points.push({
      x: radius * Math.cos(angle),
      y: radius * Math.sin(angle),
      z: 0,
    });
  }
  return points;
};

planarPhyllotaxis({organs: 100, radiusConstant: 1.2})
  .forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/6da802ef-42f6-44ac-a111-36b6eef7ebe9" />
</p>

Let's break down the code snippet provided earlier. The `planarPhyllotaxis` function takes three main parameters: `organs`, `divergenceAngle`, and `radiusConstant`. Think of `organs` as the number of leaves or branches you want to generate. The `divergenceAngle` is the angle between each leaf or branch, and the `radiusConstant` controls how quickly the spiral pattern grows.

The algorithm uses a simple loop to calculate the position of each point based on these parameters. The x and y coordinates of each point are calculated using the polar coordinates formula $\phi = n ∗ 137.5\degree, r = c\sqrt{n}$. The radius of the spiral increases as the index of the point increases, creating a beautiful and intricate pattern.

### Introducing an empty area around the center

In some cases, having an empty area around the center of the phyllotaxis pattern can be beneficial. This can be achieved by introducing a inner radius to our implementation. The inner radius will shift the spiral pattern outward, creating a circular empty space at the center.

```javascript
/**
 * Planar phyllotaxis algorithm.
 * 
 * @param {object} options
 * @param {number} options.organs - The number of organs in the arrangement.
 * @param {number} [options.divergenceAngle=Math.PI * (3 - Math.sqrt(5))] - The divergence angle between organs (in radians). Defaults to the golden angle.
 * @param {number} [options.innerRadius=0] - Distance from the center at which the first organ is placed. Defaults to 0.
 * @param {number} options.radiusConstant - The radius constant for the phyllotaxis arrangement.
 * @return {Object[]} - An array of 2D points (with z = 0) representing the phyllotaxis arrangement.
 */
const planarPhyllotaxis = ({organs, divergenceAngle = Math.PI * (3 - Math.sqrt(5)), innerRadius = 0, radiusConstant}) => {
  const points = [];
  for (let index = 0; index < organs; index++) {
    const angle = index * divergenceAngle;
    const radius = innerRadius + radiusConstant * Math.sqrt(index);
    points.push({
      x: radius * Math.cos(angle),
      y: radius * Math.sin(angle),
      z: 0,
    });
  }
  return points;
};

planarPhyllotaxis({organs: 100, innerRadius: 5, radiusConstant: 1})
  .forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/f0ac53ca-c834-4e09-82f2-b8f14bded1c2" />
</p>

### Constraining the outer radius of the pattern

To make the phyllotaxis pattern more flexible and useful, we need to introduce a way to control its overall size. Currently, the outer radius of the pattern is directly proportional to the number of organs. As we add more organs, the outer radius increases.

To fix this, we'll introduce a new parameter `outerRadius` that specifies the desired maximum radius of the pattern. By doing so, we can calculate the `radiusConstant` in a way that ensures the maximum radius value is constant, regardless of the number of organs.

```javascript
/**
 * Planar phyllotaxis algorithm.
 * 
 * @param {object} options
 * @param {number} options.organs - The number of organs in the arrangement.
 * @param {number} [options.divergenceAngle=Math.PI * (3 - Math.sqrt(5))] - The divergence angle between organs (in radians). Defaults to the golden angle.
 * @param {number} [options.innerRadius=0] - Distance from the center at which the first organ is placed. Defaults to 0.
 * @param {number} options.outerRadius - The outer radius of the generated pattern. outerRadius must be grater than innerRadius.
 * @return {Object[]} - An array of 2D points (with z = 0) representing the phyllotaxis arrangement.
 */
const planarPhyllotaxis = ({organs, divergenceAngle = Math.PI * (3 - Math.sqrt(5)), innerRadius = 0, outerRadius}) => {
  const points = [];
  const radiusConstant = (outerRadius - innerRadius) / Math.sqrt(organs - 1);
  for (let index = 0; index < organs; index++) {
    const angle = index * divergenceAngle;
    const radius = innerRadius + radiusConstant * Math.sqrt(index);
    points.push({
      x: radius * Math.cos(angle),
      y: radius * Math.sin(angle),
      z: 0,
    });
  }
  return points;
};

planarPhyllotaxis({organs: 100, innerRadius: 10, outerRadius: 25})
  .forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/3e35fbe2-6bed-4c97-8d1d-292fe12e3881" />
</p>

The image above includes a circle with a radius equal to the outer radius of the pattern. This illustrates the fact that none of the sphere center points extend outside of the outer radius.

### Changing the point distribution 

The current radius formula, which uses `Math.sqrt(index)`, generates evenly distributed points. However, we can explore other options to create different point distributions.

One approach is to replace `Math.sqrt(index)` with `Math.pow(index, distribution)`, where `distribution` is a new parameter that controls the rate at which the radius increases as the index grows. This change allows us to adjust the density of points in the phyllotaxis pattern.

The `distribution` parameter has a significant impact on the resulting phyllotaxis pattern. Here are some scenarios to consider:

- `distribution` = 0.5: This value produces the same evenly distributed points as before, using the square root of the index.
- `distribution` ∈ (0.5, 1]: In this range, points will be denser in the center of the pattern. As the distribution value approaches 1, the points will become more concentrated near the center.
- `distribution` ∈ [0, 0.5): In this range, points will be denser towards the edge of the pattern. As the distribution value approaches 0, the points will become more sparse near the center and more concentrated near the edge.

```javascript
/**
 * Planar phyllotaxis algorithm.
 * 
 * @param {object} options
 * @param {number} options.organs - The number of organs in the arrangement.
 * @param {number} [options.divergenceAngle=Math.PI * (3 - Math.sqrt(5))] - The divergence angle between organs (in radians). Defaults to the golden angle.
 * @param {number} [options.innerRadius=0] - Distance from the center at which the first organ is placed. Defaults to 0.
 * @param {number} options.outerRadius - The total radius of the generated pattern. outerRadius must be grater than innerRadius.
 * @param {number} [options.distribution=0.5] - The organ distribution. Defaults to 0.5.
 * @return {Object[]} - An array of 2D points (with z = 0) representing the phyllotaxis arrangement.
 */
const planarPhyllotaxis = ({organs, divergenceAngle = Math.PI * (3 - Math.sqrt(5)), innerRadius = 0, outerRadius, distribution = 0.5}) => {
  const points = [];
  const radiusConstant = (outerRadius - innerRadius) / Math.pow(organs - 1, distribution);
  for (let index = 0; index < organs; index++) {
    const angle = index * divergenceAngle;
    const radius = innerRadius + radiusConstant * Math.pow(index, distribution);
    points.push({
      x: radius * Math.cos(angle),
      y: radius * Math.sin(angle),
      z: 0,
    });
  }
  return points;
};

planarPhyllotaxis({organs: 100, outerRadius: 15, distribution: 0.7})
  .forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/47e82999-10dc-4100-9278-5719454a572f" />
</p>


