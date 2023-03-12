# SafeFrame API

Google Ad Manager's Fast Fetch implementation supports some aspects of the [GPT SafeFrame API](https://support.google.com/dfp_premium/answer/6023110) for non-AMP creatives that are rendered within a SafeFrame. Please refer to the [IAB specification](https://www.iab.com/wp-content/uploads/2014/08/SafeFrames_v1.1_final.pdf) of SafeFrame and the [GPT implementation details](https://support.google.com/dfp_premium/answer/6023110) for a comprehensive explanation of SafeFrame. This document is primarily aimed to explain the AMP-specific differences.

SafeFrame support (including ext.js library injection) can be forced client-side by setting `data-force-safeframe=true` attribute on amp-ad type=doubleclick elements.

# Supported Methods

The following methods all work for creatives rendered in SafeFrames via Google Ad Manager Fast Fetch, but have AMP-specific key differences that should be noted:

## \$sf.ext.resize({t, b, l, r})

This method is only supported on AMP pages, it is unavailble elsewhere. This method allows the safeframe to resize both bigger **and** smaller. Just like for \$sf.ext.expand(), you pass in the values that you wish to change the size of the safeframe by for t, b, l, and r. To change positively, you pass in positive values which is equivalent to an expansion. To change size to a smaller size, you pass in negative values. You may not mix positive and negative values in one call.

### Valid expansion usage of resize

\$sf.ext.resize({t:10, b: 10, l: 20, r:30})

### Valid shrink usage of resize

\$sf.ext.resize({t:0, b:-10, r:-10, l:0})

### Invalid use of resize

\$sf.ext.resize({t:-10, b:20, l:-10, r:-15})

For best results, only modify sizes for the b and r parameters, i.e. instead of resizing via: resize({t:10, b:10, r:10, l:10}), instead resize as resize({t:0, b:20, r:20, l:0}). See _Important Caveats_ section under **\$sf.ext.expand()** heading below.

## \$sf.ext.geom()

This is a synchronous call for the ad slot's geometry. Geometry is continuously updated in the background to keep the data fresh, with updates being sent from AMP at a maximum of once per second. Returns an object formatted as follows:

```js
{
  win: { // The measurement of the application window.
    t: 0,
    b: 800,
    l: 0,
    r: 400
  },
  self: { // The measurement of the SafeFrame, relative to win
    t: 100,
    b: 150,
    l: 10,
    r: 330,
  },
  exp: { // The amount that a SafeFrame can expand
    t: 0,
    l: 0,
    b: 750,
    r: 80
  },
  pos: { // The position of the safeframe, relative to the viewport
    t: 0,
    l: 0,
    b: 50,
    r: 320,
  },
}
```

Note that t=Top, b=Bottom, r=Right, and l=Left. See more details below in \$sf.ext.expand() section.

**Important note about pos:** \$sf.ext.geom().pos is only available on AMP pages. Do not expect to be able to use that API on other pages. However, you should be able to use win, self, and exp anywhere that safeframe runs, AMP or not.

**Important note about exp:** Expansion in AMP only technically works to the right, and bottom. There is no notion of expanding left or up. However, in the majority of cases, creatives are centered in the AMP page. Thus, expanding 100 to the right ends up being equivalent to expanding 50 to the left and right. In AMP, the exp values represent how much the creative would need to expand to consume the entire viewport. See more details about expand below.

## \$sf.ext.expand({t, b, l, r, push})

This API call requests that the SafeFrame be expanded. The parameter passed in is an object that specifies how much to expand in each direction, and whether to expand by push or by overlay (push=true for expand by push, push=false for expand by overlay). However, within AMP, expanding is only supported to the right and bottom.

