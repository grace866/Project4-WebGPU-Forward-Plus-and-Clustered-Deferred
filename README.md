WebGL Forward+ and Clustered Deferred Shading
======================

**University of Pennsylvania, CIS 565: GPU Programming and Architecture, Project 4**

* Grace Tan
* Tested on: (TODO) **Google Chrome 222.2** on
 Windows 11 Home 25H2 (Build 26200), AMD Ryzen 9 8945HX @ 2.50GHz, 16GB RAM, NVIDIA GeForce RTX 5060 Laptop GPU 8GB

## Live Demo

[![](img/thumb.png)](https://grace866.github.io/Project4-WebGPU-Forward-Plus-and-Clustered-Deferred/)

## 
This project implements three different methods for computing lighting within a scene:
1. Naive Renderer
2. Forward+ Renderer
3. Clustered Deferred Renderer

We do this to explore how to improve upon the Naive Renderer, which iterates through every light in the scene to compute the total light contribution for each fragment.

## Video Demos

### Naive
https://github.com/user-attachments/assets/a544b983-bc3c-4a23-a1e7-de248013b8bc

The naive renderer is a standard forward renderer. Every geometry fragment loops over every light in the scene and accumulates its contribution. This is inefficient because not all lights contribute to the color of a given fragment. As pictured above, this works reasonably well for a small number of lights, but performance tends to scale poorly as the light count increases or the size of the scene increases. Fragments that are later overwritten by closer geometry (overdraw) also pay this full cost.

### Forward Plus
https://github.com/user-attachments/assets/70eb99d8-9558-41a0-9c6c-bcc7285d6c79

Forward+ adds a light culling pass before shading. The view frustum is divided into a 3D grid of clusters (x and y space-space tiles, z depth slices), and compute shaders build the axis-aligned bounding box in camera space and assign the corresponding lights for each cluster. 

The cluster grid is configurable. Smaller clusters overlap fewer lights, so shading is cheaper, but the culling pass has more clusters to test; larger clusters have the opposite tradeoff. Forward+ still shades every rasterized fragment, including ones that are later hidden by closer geometry. 

### Clustered Deferred
https://github.com/user-attachments/assets/356d6bb4-7a67-4c7d-8b91-dea4d62c9ef5

Clustered deferred uses the same cluster light culling as Forward+ but separates geometry from lighting. 
1. **G-buffer pass:** The scene geometry is rendered once in order to write per-pixel surface data (albedo, normal, and depth) to textures. Depth is used to calculate position in lighting pass (doing this avoids writing to a fourth position texture).
2. **Lighting pass** A fullscreen quad is drawn. For each pixel, the fragment shader reads the G-buffer, reconstructs the world position from depth, finds the pixel's cluster, and loops over only that cluster's lights.

Because the depth test has already resolved visibility, lighting runs exactly once per visible pixel and no work is wasted on occluded fragments. The cost of lighting no longer depends on scene complexity, only on screen resolution and light count. However, there are still tradeoffs: extra memory and bandwidth is the cost for writing and reading to the G-buffer textures. 

## Debugging Images 
To verify the success of the G-buffer pass, I rendered the data stored in the albedo, normal, and position (computed from depth) textures. 

<table>
  <tr>
    <td align="center">
      <img src="img/albedo.png" width="300"><br>
      <em>Albedo</em>
    </td>
    <td align="center">
      <img src="img/normal.png" width="300"><br>
      <em>Normal</em>
    </td>
    <td align="center">
      <img src="img/position.png" width="300"><br>
      <em>Position</em>
    </td>
  </tr>
</table>

## Performance Analysis

### Credits

- [Vite](https://vitejs.dev/)
- [loaders.gl](https://loaders.gl/)
- [dat.GUI](https://github.com/dataarts/dat.gui)
- [stats.js](https://github.com/mrdoob/stats.js)
- [wgpu-matrix](https://github.com/greggman/wgpu-matrix)
