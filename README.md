<p>
    <a href="https://npmjs.com/package/lingallery"><img src="https://img.shields.io/npm/v/lingallery.svg?style=flat-square" alt="NPM version"></a>
    <a href="https://www.npmjs.com/package/lingallery"><img src="https://img.shields.io/npm/dm/lingallery.svg?style=flat-square" alt="NPM downloads"></a>
    <a href="https://www.npmjs.com/package/lingallery"><img src="https://img.shields.io/npm/l/lingallery.svg?style=flat-square" alt="License"></a>
</p>

# Lingallery + Video Addons
This is a fork of Lingallery with video addons
## Demo
<img src="https://i.imgur.com/dpsLhVi.png">
<br><br>
## Install

```bash
npm i ta0uf19/lingallery
```

## Usage
 Be sure to enable the addon by setting **enableVideoElement: true** in the addons object
```javascript
<lingallery
      :mobile-height="300"
      :mobile-height-breakpoint="600"
      :addons="{ enableVideoElement: true  }"
      :width="600"
      :height="400"
      :items="[{
           thumbnail: 'https://picsum.photos/64/64/?image=0',
           video: {
              src: 'http://techslides.com/demos/sample-videos/small.mp4',
              type: 'video/mp4',
              autoplay: true, // required
              controls: true, // required
              muted: false, // required
           }
    }]"

/>
```


### Lingallery options

You can pass some props to adapt the behavior and looks of Lingallery.

| Prop        | Description | Default | Type |
|-------------|-------------|---------|------|
| `width` | Specifies the width of the main image area in pixels. | 600 | Number |
| `height` | Specifies the height of the main image area in pixels. | 400 | Number |
| `responsive` | Defines whether the image gallery should take up all available width space. | false | Boolean |
| `startImage` | Sets the index of the image the gallery should start with. | 0 | Number |
| `baseColor` | Defines the base color (at the moment only for the thumbnail border color) | #fff | String |
| `accentColor` | Defines the accent color (at the moment only for the spinner and the active thumbnail border) | #3498db | String |
| `textColor` | Defines the text color of the caption. | #000 | String |
| `showThumbnails` | Defines whether thumbnails should be displayed. | true | Boolean |
| `mobileHeight` | If set to a number larger than 0 the image height will not exceed that value on mobile devices | 0 | Number |
| `mobileHeightBreakpoint` | If mobileHeight is defined this prop sets the breakpoint below which the image height will not exceed the mobileHeight value | 0 | Number |
| `leftControlClass` | If defined adds a class to the left control button to enable custom icons | '' | String |
| `rightControlClass` | If defined adds a class to the right control button to enable custom icons | '' | String |
| `disableImageClick` | If set to true a click on the large image will not show the next image | false | Boolean |

## Another addons

### Large View
By passing the prop `enableLargeView` you can enable the large view feature. When clicking on the large main image a modal will open up with the image displaying in full size.

```
<lingallery :addons="{ enableLargeView: true }" ... />
``` 

If you want to show a different image (maybe a larger version) in the modal you can add the property `largeViewSrc` into the `items` prop.

```
<lingallery :addons="{ enableLargeView: true }" :items="[{ src: 'image1.jpg', largeViewSrc: 'image1_large.jpg' }]" ... />
```

### Picture Element
Sometimes you might want to display different images depending on the user's screen size. Responsive images in Lingallery are possible with this addon. You can specify both a `type` and a `media` attribute. Be sure to enable the addon by setting `enablePictureElement: true` in the `addons` object.

```
<lingallery
  :addons="{ enablePictureElement: true }"
  :items="[
    {
      src: 'image1.jpg',
      pictureElement: [
        {
          srcset: 'image1_large.jpg',
          media: '(min-width: 600px)',
          type: 'image/jpg'
        },
        {
          srcset: 'image1_large.webp',
          media: '(min-width: 600px)',
          type: 'image/webp'
        },
        {
          srcset: 'image1_huge.jpg',
          media: '(min-width: 1200px)'
        }
      ]
    }
  ]"
  ...
/>
```
