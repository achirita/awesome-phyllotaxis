# An exploration of various phyllotaxis algorithms

## The planar phyllotaxis algorithm

As we explore the fascinating world of phyllotaxis, we'll start with a fundamental concept: the planar phyllotaxis algorithm. This mathematical model helps us understand the intricate patterns found in nature, from the arrangement of leaves on a stem to the branching of trees.

So, what is phyllotaxis, and how does the planar phyllotaxis algorithm work? Simply put, phyllotaxis is the study of the geometric patterns that occur in the growth of plants. The planar phyllotaxis algorithm is a mathematical formula that generates a two-dimensional spiral pattern, mimicking the way leaves or branches grow in a circular arrangement.

### Polar coordinates formula

$\phi = n ∗ 137.5\degree, r = c\sqrt{n}$

where:
- $n$ is the ordering number of a floret, counting outward from the center.
- $\phi$ is the angle between a reference direction and the position vector of the $n^{th}$ floret in a polar coordinate system originating at the center of the capitulum.
- $r$ is the distance between the center of the capitulum and the center of the $n^{th}$ floret, given a constant scaling parameter $c$.

**Note:** Whilst the 137.5 degrees divergence angle most accurately describes the arrangement of organs in flowers, one could use any value for artistic purposes.

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

const points = planarPhyllotaxis({organs: 100, radiusConstant: 1.2});
points.forEach(point => scene.add(makeSphere({radius: 1, center: point})));
```


