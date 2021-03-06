# Ember-cli-visual-acceptance

[ci-img]: https://img.shields.io/travis/ciena-frost/ember-cli-visual-acceptance.svg "Travis CI Build Status"
[ci-url]: https://travis-ci.org/ciena-frost/ember-cli-visual-acceptance
[cov-img]: https://img.shields.io/coveralls/ciena-frost/ember-cli-visual-acceptance.svg "Coveralls Code Coverage"
[cov-url]: https://coveralls.io/github/ciena-frost/ember-cli-visual-acceptance
[npm-img]: https://img.shields.io/npm/v/ember-cli-visual-acceptance.svg "NPM Version"
[npm-url]: https://www.npmjs.com/package/ember-cli-visual-acceptance

Create baseline images and test for CSS regression during standard Ember tests using html2Canvas and ResembleJS

## Installation

`ember install ember-cli-visual-acceptance`
### Configuration
You can modify the save directory in `ember-cli-build.js` by including
```
visualAcceptanceOptions: {
      imageDirectory: 'visual-acceptance'
    }
``` 

## Usage

  * Configure the systems and browsers that will capture images
    * Different systems and browsers produce different images
    * To prevent false positives images are only captured against specific targets
    * The results of each target are stored in separate directories and are only compared against the same target
  ```javascript
    Config example
  ```
  * Add labeled captures into your tests (what are the params here?)
  ```javascript
    return capture(label, height, width, misMatchPercentage)
  ```
  * The first capture will automatically become the baseline image
  * When executing asynchronous tests with an explicit done() you must use a `.catch` to handle image assertion failures
  ```javascript
    capture('label', null, null, 0.00).then(function (data) {
      console.log(arguments)  
      /* ResembleJs output
      {
        misMatchPercentage : 100, // %
        isSameDimensions: true, // or false
        dimensionDifference: { width: 0, height: -1 }, // defined if dimensions are not the same
        getImageDataUrl: function(){}
      }
    */
      done()
    }).catch(function (err) {
      done(err)
    })
```
  * Otherwise just return the promise
```javascript
return capture('placeholder', null, null, 0.00)
```


### Parameters
|           Name           | Type   | Default             | Description                                                                                                                                                                         |
|:------------------------:|--------|---------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| imageName                | string | required            | Name of the image you wish to save                                                                                                                                                  |
| height                   | number | null                | Define the height of the canvas in pixels. If null, renders with full height of the window.                                                                                         |
| width                    | number | null                | Define the width of the canvas in pixels. If null, renders with full width of the window.                                                                                           |
| misMatchPercentageMargin | float  | 0.00                | The maximum percentage ResembleJs is allowed to misMatch.                                                                                                                           |
| imageDirectory           | string | 'visual-acceptance' | The location where the `-passed.png` and `-failed.png` images will be saved. *(Note: Cannot be within the `tests` folder as this will restart the test every time an image is save) |

### Establishing a new baseline
To establish a new baseline simply located the `-passed.png` of the image you wish to establish a new baseline for and delete it. The next run of `ember test` will create the new baseline.

### What a failure looks like
From ember test:
```
Integration: FrostSelectComponent selects the hovered item when enter is pressed
    ✘ Image is above mismatch threshold.: expected false to be true
        AssertionError: Image is above mismatch threshold.: expected false to be true
```

Then a new `<nameOfImage>-fail.png` will show up in your `visual-acceptance` directory. 
Visual differences are shown in pink. 
More info about visual diffs can be found here https://github.com/Huddle/Resemble.js. 
ember-cli-visual-acceptance only uses the `.scaleToSameSize()` option for ResembleJS

### Example Usage

#### Without done() callback
```javascript
it('supports placeholder', function () {
  const $input = this.$('.frost-select input')
  expect($input.attr('placeholder')).to.eql('Select something already')
  return visualAcceptance('placeholder', null, null, 0.00)
})
```

#### With done() callback
```javascript
it('selects the hovered item when enter is pressed', function (done) {
  keyUp(dropDown, 40)
  keyUp(dropDown, 13)

  Ember.run.later(() => {
    let dropDownInput = this.$('.frost-select input')
    let value = dropDownInput.val()
    expect(value).to.eql(props.data[0].label)
    capture('Boston', null, null, 0.00).then(function (data) {
      done()
    }).catch(function (err) {
      done(err)
    })
  })
})
```
  return capture('placeholder', null, null, 0.00)
