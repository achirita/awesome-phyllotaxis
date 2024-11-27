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

planarPhyllotaxis({organs: 100, radiusConstant: 1.2}).forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/6da802ef-42f6-44ac-a111-36b6eef7ebe9" />
</p>

Let's break down the code snippet provided earlier. The planarPhyllotaxis function takes three main parameters: organs, divergenceAngle, and radiusConstant. Think of organs as the number of leaves or branches you want to generate. The divergenceAngle is the angle between each leaf or branch, and the radiusConstant controls how quickly the spiral pattern grows.

The algorithm uses a simple loop to calculate the position of each point based on these parameters. The x and y coordinates of each point are calculated using the polar coordinates formula $\phi = n ∗ 137.5\degree, r = c\sqrt{n}$. The radius of the spiral increases as the index of the point increases, creating a beautiful and intricate pattern.

### Adding radius offset

In some cases, having an empty area around the center of the phyllotaxis pattern can be beneficial. This can be achieved by introducing a radius offset to our implementation. The radius offset will shift the spiral pattern outward, creating a circular empty space at the center.

```javascript
/**
 * Planar phyllotaxis algorithm.
 * 
 * @param {object} options
 * @param {number} options.organs - The number of organs in the arrangement.
 * @param {number} [options.divergenceAngle=Math.PI * (3 - Math.sqrt(5))] - The divergence angle between organs (in radians). Defaults to the golden angle.
 * @param {number} [options.radiusOffset=0] - Distance from the center at which the first organ is placed. Defaults to 0.
 * @param {number} options.radiusConstant - The radius constant for the phyllotaxis arrangement.
 * @return {Object[]} - An array of 2D points (with z = 0) representing the phyllotaxis arrangement.
 */
const planarPhyllotaxis = ({organs, divergenceAngle = Math.PI * (3 - Math.sqrt(5)), radiusOffset = 0, radiusConstant}) => {
  const points = [];
  for (let index = 0; index < organs; index++) {
    const angle = index * divergenceAngle;
    const radius = radiusOffset + radiusConstant * Math.sqrt(index);
    points.push({
      x: radius * Math.cos(angle),
      y: radius * Math.sin(angle),
      z: 0,
    });
  }
  return points;
};

planarPhyllotaxis({organs: 100, radiusOffset: 5, radiusConstant: 1}).forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/f0ac53ca-c834-4e09-82f2-b8f14bded1c2" />
</p>


