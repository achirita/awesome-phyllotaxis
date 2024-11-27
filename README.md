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

### Adding depth to the pattern

To give our phyllotaxis pattern a more realistic, three-dimensional appearance, we can adjust the z position of each organ. One way to do this is to use a function that approximates the curvature of a flower head.

In this case, we'll add a new parameter `height` which defines the maximum height of the curvature and use the function `Math.pow(index, 2)` to determine the z height of each organ. This will create a gentle, curved shape that resembles the natural growth of a flower.

```javascript
/**
 * Planar phyllotaxis algorithm.
 * 
 * @param {object} options
 * @param {number} options.organs - The number of organs in the arrangement.
 * @param {number} [options.divergenceAngle=Math.PI * (3 - Math.sqrt(5))] - The divergence angle between organs (in radians). Defaults to the golden angle.
 * @param {number} [options.innerRadius=0] - Distance from the center at which the first organ is placed. Defaults to 0.
 * @param {number} options.outerRadius - The total radius of the generated pattern. outerRadius must be grater than innerRadius.
 * @param {number} [options.height=0] - The max height of the organs in the arrangement.
 * @param {number} [options.distribution=0.5] - The organ distribution.
 * @return {Object[]} - An array of 2D points representing the phyllotaxis arrangement.
 */
const planarPhyllotaxis = ({organs, divergenceAngle = Math.PI * (3 - Math.sqrt(5)), innerRadius = 0, outerRadius, height = 0, distribution = 0.5}) => {
  const points = [];
  const radiusConstant = (outerRadius - innerRadius) / Math.pow(organs - 1, distribution);
  for (let index = 0; index < organs; index++) {
    const angle = index * divergenceAngle;
    const radius = innerRadius + radiusConstant * Math.pow(index, distribution);
    points.push({
      x: radius * Math.cos(angle),
      y: radius * Math.sin(angle),
      z: - (height / Math.pow(organs - 1, 2) * Math.pow(index, 2)),
    });
  }
  return points;
};

planarPhyllotaxis({organs: 300, outerRadius: 25, height: 7})
  .forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/456f9b15-6ffa-483a-9758-9606bfb02497" />
</p>

Of course there are many other mathematical functions which can be used instead of $index^2$. Feel free to play around until you find something that suits your use case.

## The cylindrical phyllotaxis algorithm

The cylindrical phyllotaxis model is a mathematical representation of the growth patterns found in plants with cylindrical or columnar shapes, such as cacti and succulents. By introducing a linear function to increase the Z coordinate value, the cylindrical phyllotaxis model simulates the growth of plants with a vertical axis.

The cylindrical model uses the following formula to determine the x, y and z coordinates of each point: $\phi = n ∗ 137.5, r = const, H = h * n$.

```javascript
/**
 * Cylindrical phyllotaxis algorithm.
 * 
 * @param {object} options
 * @param {number} options.organs - The number of organs in the arrangement.
 * @param {number} [options.divergenceAngle=Math.PI * (3 - Math.sqrt(5))] - The divergence angle between organs (in radians). Defaults to the golden angle.
 * @param {number} radius - The cylinder radius.
 * @param {number} height - The cylinder height.
 * @return {Object[]} - An array of 3D points representing the phyllotaxis arrangement.
 */
const cylindricalPhyllotaxis = ({organs, divergenceAngle = Math.PI * (3 - Math.sqrt(5)), radius, height}) => {
  const points = [];
  for (let index = 0; index < organs; index++) {
    const angle = index * divergenceAngle;
    points.push({
      x: radius * Math.cos(angle),
      y: radius * Math.sin(angle),
      z: height * index / organs,
    });
  }
  return points;
};

cylindricalPhyllotaxis({organs: 100, radius: 7, height: 15})
  .forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/d1cc1baa-b7b0-42c9-8872-343193be514e" />
</p>

### Wavy patterns on the z axis

As described in the previous example, a constant radius value generates points distributed on the surface of a cylinder. Let's see what happens when we use various mathematical functions to change the radius value based on the organ index. 

We will start by defining the `radiusConstant` as `radius + amplitude * Math.sin(index)`. This will introduce a repeating, onlulating pattern along the z axis. If the cylinder radius value is large enough the variation introduced by the `Math.sin` function will not be noticeable and that's the reason why the `amplitude` parameter was added.

```javascript
/**
 * Cylindrical phyllotaxis algorithm.
 * 
 * @param {object} options
 * @param {number} options.organs - The number of organs in the arrangement.
 * @param {number} [options.divergenceAngle=Math.PI * (3 - Math.sqrt(5))] - The divergence angle between organs (in radians). Defaults to the golden angle.
 * @param {number} radius - The cylinder radius.
 * @param {number} height - The cylinder height.
 * @param {number} [amplitude=1] - Wave pattern amplitude. Defaults to 1.
 * @return {Object[]} - An array of 3D points representing the phyllotaxis arrangement.
 */
const cylindricalPhyllotaxis = ({organs, divergenceAngle = Math.PI * (3 - Math.sqrt(5)), radius, height, amplitude = 1}) => {
  const points = [];
  for (let index = 0; index < organs; index++) {
    const angle = index * divergenceAngle;
    const radiusConstant = radius + amplitude * Math.sin(index);
    points.push({
      x: radiusConstant * Math.cos(angle),
      y: radiusConstant * Math.sin(angle),
      z: height * index / organs,
    });
  }
  return points;
};

cylindricalPhyllotaxis({organs: 200, radius: 5, height: 20})
  .forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/7a4f647d-afd0-428e-a9d3-bc9c04791dc9" />
</p>

# Resources
- [Algorithmic Botany - Chapter 4](https://algorithmicbotany.org/papers/abop/abop-ch4.pdf)
- [Algorithmic Botany - The use of positional information in the modeling of plants](https://algorithmicbotany.org/papers/sigcourse.2003/2-27-positional.pdf)
