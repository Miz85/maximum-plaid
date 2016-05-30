![Maximum Plaid](/logo/maximum-plaid-logo.png)

[![Build Status](https://travis-ci.org/ivanvanderbyl/maximum-plaid.svg?branch=master)](https://travis-ci.org/ivanvanderbyl/maximum-plaid)
[![Ember Observer Score](http://emberobserver.com/badges/maximum-plaid.svg)](http://emberobserver.com/addons/maximum-plaid)

Template driven data visualisation for ambitious applications.

**WIP**

# Design

Traditional charting libraries often times have poor separation of concerns from
data and presentation. This leads to poor maintainability and code reuse.

Another consideration when visualising a dataset is whether you desire _efficiency_
or _expressiveness_, when you typically can't have both.

`maximum-plaid` is designed to change this by utilising Ember's declaritive templating 
and component composition, with D3's leading primitives for producing easy to
compose data visualisations for both data exploration and explanatory presentation.

# Proposed API

On their own, components for even the simplest elements in a visualisation can
quickly require complicated APIs. To solve this, Ember contextual components can
provide lower level primitive components with the necessary inputs for scaling
and positioning. This reduces the API surface for the user to quickly produce
visualisations in very few lines of code.

```hbs
{{#plaid-plot xScale yScale width height as |plot|}}
  {{plot.right-axis}}
  {{plot.line responseTimes}}
{{/plaid-plot}}
```

Or perhaps using [ember-d3-scale](https://github.com/spencer516/ember-d3-scale#linear-scale)

```hbs
{{#plaid-plot (time-scale (extent timestamps) (extent 0 width)) (linear-scale yDomain yRange) width height as |plot|}}
  {{plot.right-axis}}
  {{plot.line responseTimes}}
{{/plaid-plot}}
```

Ideally width and height would be take into account by just supplying the `plotArea`
property.

**What about different scales for each line?**

If you don't supply a scale, it will be set to `null` by default, which can easily
be overridden later. In this case we want to keep the xScale consistent.

```hbs
{{#plaid-plot xScale as |plot|}}
  {{#each metricNames as |metricName|}}
    {{plot.right-axis scale=(get yScalesForMetrics metricName)}}
    {{plot.line (get metrics metricName) yScale=(get yScalesForMetrics metricName)}}
  {{/each}}
{{/plaid-plot}}
```

```js
// my-line-chart-component
export default Component.extend({
  values: {
    responseTimes: [[1450345980000,1914], ...],
    concurrency: [[1450345980000,1000], ...],
    transactionRate: [[1450345980000,80000], ...],
  },

  metricNames: computed('values.@each', {
    get() { 
      const values = this.get('values');
      return Object.keys(values);
    }
  }),

  yScalesForMetrics: computed('values.@each', {
    get() {
      const values = this.get('values');
      const metricNames = this.get('metricNames');
      let scales = {};
      metricNames.forEach((metricName) => {
        scales[metricName] = 
          this.getScaleForMetricName(metricName, values[metricName]);
      });

      return scales;
    }
  }),

  getScaleForMetricName(metricName, values) {
    // We use the same yRange for all lines.
    const yRange = this.get('yRange');
    const yDomain = this.getDomainForMetricName(metricName, values);
    // Compute domain based on values and return a scaling function.
    return scaleLinear().domain(yDomain).range(yRange);
  },

  getDomainForMetricName(metricName, values) {
    // We only need the domain for one dimension, because the other needs to
    // be equal for all lines, in this case; `time`.
    return extent(values, (d) => d[1]);
  },
});
```

# Components

### `plaid-symbol`

Provides an easy to use and straight forward interface to `d3-shape`'s symbol 
generators. `primitive-symbol` can be used in the same way as any Ember component
and will  render as a `<path>` tag containing the path data for the specified
symbol type.

#### Options

- `type`: Symbol to render, can be any of `circle`, `diamond`, `cross`, 
`square`, `star`, `triangle`, `wye`.
- `size`: Specifies the symbol render size. This is the area of the
symbol, which typicall equates to 1/4th of the actual width or height, depending
on the shape.
- `fill`: SVG path `fill`  property.
- `stroke`: SVG path `stroke` property.
- `strokeWidth`: SVG path `stroke-width` property.
- `top`: Top offset (applied using transform).
- `left`: Left offset (applied using transform).


# Mixins

## `PlotArea`

Provides simple calculations for specifying the position of the main graphic in
a visualisation.

### Example:

```js
import { PlotArea } from 'maximum-plaid/mixins/plot-area';

export default Component.extend(PlotArea, {
  margin: '16 24',

  didRender() {
    const { top, left } = this.get('plotArea');
    select(this.element).select('g.plot').attr('transform', `translate(${left},${top})`);
  }
});
```

# Utils

## `computedExtent`

Creates a computed property which calculates the extent of all inputs provided
using `extent` from d3-array.

## Installation

**NOTE: `maximum-plaid` requires Ember v2.3+**

    ember install maximum-plaid

## Running

* `ember server`
* Visit your app at http://localhost:4200.

## Running Tests

* `npm test` (Runs `ember try:testall` to test your addon against multiple Ember versions)
* `ember test`
* `ember test --server`

## Building

* `ember build`

For more information on using ember-cli, visit [http://www.ember-cli.com/](http://www.ember-cli.com/).

# FAQ

### Why `maximum-plaid`?

**Ivan**: Initially I was searching name which represented both a
type of fabric for the visualisation metaphore, but also I've been told that friends
have lost me in crowds walking around Williamsburg because I wear so much plaid. Also, the goal
of this project is to make data visualisation easy and efficient, so in keeping with
the promised performance mode of the next [Tesla Roadster](http://mashable.com/2015/07/17/new-tesla-roadster/#3NCT_4NpL8qU), I found "maximum plaid" to be
fitting.

### You spelt visualization wrong

That's not a question.

As the main author of this project is Australian — a country which speaks a 
variation of British English, words such as `visualisation` are spelt with an `s`
instead of a `z`. Another word you may find incorrectly spelt is `colour`. Please
don't issue Pull Requests to fix this.
