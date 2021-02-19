# label-match

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

### Current state of this project

This project is only a proof of concept and is not suitable for production use.

The general concept is the user takes a photo of a label, uploads an image of what the label should look like, then both are sent to google vision cloud service to read all words/characters on both labels. The code then compares the returned list for each label and forms a list for both the taken picture and the uploaded image of words and characters that are present in it but not the other. This is then used to highlight over both the picture and the image to show the differences.

The original concept was to make an api call to get the image of the label from the WineWorks system, but this was no achievable as there is no such api endpoint at the current time. If an api endpoint is created to provide this, then this project could be easily modified to add an input for a stock code (could also be grabbed from url parameters) and code for handeling the api call. Note: google vision likes to take images in base64 encoding, but may be able to take other formats.

As the code is it will not run as there is no API key in the project. To fix this, create a file in assets called secure.json the add the privateKey and the new API key. secure.json is not tracked by git, but this storage of an api key is not recommended for anything beyond development testing. Remember that if your api key leaks it may be utilized by bad actors who could use it at your costs.
To get an api key go to google cloud console and enable the vision API then generate a key for it. Note: it is recommended to restrict where and how that key can be used. This will require a google account and billing information is required to be able to generate and use a key, but the first 1000 calls are free each month with a cost of $1.50 USD per 1000 calls after that (keep in mind that each comparison is 2 calls)

Limitations:

Due to the incomplete implementation, the highlight may not line up with the image. This is because the highlights are sized and placed relative to their parent element, and this is not always the same size as the image that it also contains.

Low resolutions of either the uploaded image or the taken picture can lead to an increased rate of false positives. This may lead to large portions of the label being highlighted incorrectly and therefore make finding any actual differences difficult.
Along with this there should be reasonable lighting when taking the picture of the label. There is no need for studio lighting, but if the image looks hard to read from a human point of view because of bad lighting, it is likely to cause issues for google vision too.
