# RFC: Layer Mask

* **Authors**: Ravi Akkenapally
* **Date**: July 2018
* **Status**: For Review.


## Overview

deck.gl provides many layers to visualize the data. But in some cases it is very useful to use one or more layers as a mask to visualize the other layer(s). For example a `HexagonLayer` is used to show the data aggregated to the hexagon bins, and a `ScatterplotLayer` can define regions of interest, and user is interested it visualizing hexagon aggregation only in regions defined by `ScatterplotLayer`. This RFC proposes new API to use one or more layers as a mask when rendering other layers.


## Proposed

A new prop object `layerMask` is added for `Layer` class, that defines the layers to use as mask and the mask operation, which defines either to discard pixels outside of the mask or just highlight the pixels inside the mask.

To use the layer with id `layerid-1` as a mask and discard all pixels that are not in region defined by `layerid-1`.

```
const layerMask = {
  layers: 'layerid-1'
  operation: 'filter'
}
```

To highlight all pixels that fall into the area defined by `layerid-1` or `layerid-2`.

```
const layerMask = {
  layers: ['layerid-1', 'layer-id2']
  operation: 'highlight'
  highlightColor: [255, 0, 0, 255]
}
```


## Implementation

When a layer is being drawn , if it contains `layerMask` prop, a texture is generated by rendering all the layers into a `Framebuffer` object. This texture is then used to identify the masked region during layer's draw cycle.

### Building the mask Texture

In the beginning of `drawLayerInViewport`, we will check for `layerMask` prop and setup `Framebuffer` object (equal to he size of current viewport). All the layers specified in `layerMask` prop are rendered into this `Framebuffer` object. All we need from these layers is final pixel position, we can add fast render mode for all layers where features such as lighting and etc can be disabled, and we can enable such mode during this step.

### Performing Intersection operations

Above texture is sampled in Layers's vertex shader and a boolean varying, `inMaskRegion` is set. This can be used in fragment shader to either discard or change color of the pixel. All this new shader functionality can be enabled by a uniform. We can be group all this vertex and fragment shader functionality into a shader module (`filterModule` ?).
