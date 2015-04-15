# mongoose-crate-gm

[![Dependency Status](https://david-dm.org/achingbrain/mongoose-crate-gm.svg?theme=shields.io)](https://david-dm.org/achingbrain/mongoose-crate-gm) [![devDependency Status](https://david-dm.org/achingbrain/mongoose-crate-gm/dev-status.svg?theme=shields.io)](https://david-dm.org/achingbrainmongoose-crate-gm#info=devDependencies) [![Build Status](https://img.shields.io/travis/achingbrain/mongoose-crate-gm/master.svg)](https://travis-ci.org/achingbrain/mongoose-crate-gm) [![Coverage Status](http://img.shields.io/coveralls/achingbrain/mongoose-crate-gm/master.svg)](https://coveralls.io/r/achingbrain/mongoose-crate-gm)

A [mongoose-crate](https://github.com/achingbrain/mongoose-crate) FileProcessor that applies image transformations with [GraphicsMagick](http://www.graphicsmagick.org) via the [`gm`](http://npmjs.org/package/gm) module.

## Prequisites

A modern version of GraphicsMagic or ImageMagick installed and available on the path.  You may also need GhostScript, I did on one computer but not on another.  YMMV.

## Usage

You can pass images through one or more filters:

```javascript
var mongoose = require('mongoose'),
  crate = require('mongoose-crate'),
  LocalFS = require('mongoose-crate-localfs'),
  GraphicsMagic = require('mongoose-crate-gm')

var PostSchema = new mongoose.Schema({
  title: String,
  description: String
})

PostSchema.plugin(crate, {
  storage: new LocalFS({
    directory: '/path/to/storage/directory'
  }),
  fields: {
    image: {
      processor: new GraphicsMagic({
        tmpDir: '/tmp', // Where transformed files are placed before storage, defaults to os.tmpdir()
        formats: ['JPEG', 'GIF', 'PNG'], // Supported formats, defaults to ['JPEG', 'GIF', 'PNG', 'TIFF']
        transforms: {
          original: {
            // keep the original file
          },
          small: {
            resize: '150x150',
            format: '.jpg'
          },
          medium: {
            resize: '250x250',
            format: '.jpg'
          }
        }
      })
    }
  }
})

var Post = mongoose.model('Post', PostSchema)
```

.. then later:

```javascript
var post = new Post()
post.attach('image', {path: '/path/to/image'}, function(error) {
  // image is now attached and post.image is populated e.g.:
  // post.image.small.url
  // post.image.medium.url
})
```

## Meta data

`mongoose-crate-gm` extends the basic meta data added by `mongoose-crate` to add some image specific fields.  It provides the following for each transformation:

```javascript
{
  width: 120,
  height: 103,
  depth: 8,
  format: 'PNG',
  name: 'dragon.png',
  size: 26887,
  url: 'http://my_bucket.s3.amazonaws.com/folder/4fbaaa31db8cec0923000019-medium.png'
}
```

## Transformations

Transformations are achieved by invoking the **convert** command and passing all the properties of the transform as arguments.

Example in convert command:

    gm convert source.png -crop 120x120 -blur 5x10 output.png

Example in plugin options:

```javascript
PostSchema.plugin(crate, {
  ...
  fields: {
    image: {
      processor: new GraphicsMagic({
        transforms: {
          small: {
            crop: '120x120',
            blur: '5x10'
```

For more information on available transforms, see the [GraphicsMagick website](http://www.graphicsmagick.org/convert.html).

### Changing the destination format

You can change the destination format by using the special transform property `format` with a known file extension like *png*, *jpg*, *gif*, etc:

```javascript
PostSchema.plugin(crate, {
  ...
  fields: {
    image: {
      processor: new GraphicsMagic({
        transforms: {
          as_jpeg: {
            format: 'jpg'
```

### Supported formats

By default we'll only try to process a few common image types.  The supported list defaults to *jpgs*, *pngs*, *gifs* and *tiffs*.

This list can be overridden by specifying the `formats` argument:

```javascript
PostSchema.plugin(crate, {
  ...
  fields: {
    image: {
      processor: new GraphicsMagic({
        formats: ['JPEG', 'GIF', 'PNG']
```

The values should match up with `convert`'s supported formats.  To see a list of all formats supported by your install, run:

```
gm convert -list format
```

### Using ImageMagick instead of GraphicsMagic

By default the `gm` module uses the GraphicsMagic binary. If you wish to use ImageMagick instead, pass true for the `imageMagick` option:

```javascript
PostSchema.plugin(crate, {
  ...
  fields: {
    image: {
      processor: new GraphicsMagic({
        imageMagick: true
```
