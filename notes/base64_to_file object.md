# base64 to file object



####Way 1: only works for dataURL, not for other types of url.

````js
function dataURLtoFile(dataurl, filename) {
    var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
    while(n--){
        u8arr[n] = bstr.charCodeAt(n);
    }
    return new File([u8arr], filename, {type:mime});
}

//Usage example:
var file = dataURLtoFile('data:image/png;base64,......', 'a.png');
console.log(file);
````

####Way 2: works for any type of url, (http url, dataURL, blobURL, etc...)

````js
//return a promise that resolves with a File instance
function urltoFile(url, filename, mimeType){
    return (fetch(url)
        .then(function(res){return res.arrayBuffer();})
        .then(function(buf){return new File([buf], filename, {type:mimeType});})
    );
}

//Usage example:
urltoFile('data:image/png;base64,......', 'a.png', 'image/png')
.then(function(file){
    console.log(file);
})
````
Both works in Chrome and Firefox.



-------

-------




> The function described below is available on NPM: var b64toBlob = require('b64-to-blob')


The atob function will decode a base64-encoded string into a new string with a character for each byte of the binary data.

````js
var byteCharacters = atob(b64Data);
````

Each character's code point (charCode) will be the value of the byte. We can create an array of byte values by applying this using the .charCodeAt method for each character in the string.

````js
var byteNumbers = new Array(byteCharacters.length);
for (var i = 0; i < byteCharacters.length; i++) {
    byteNumbers[i] = byteCharacters.charCodeAt(i);
}
````

You can convert this array of byte values into a real typed byte array by passing it to the Uint8Array constructor.

````js
var byteArray = new Uint8Array(byteNumbers);
````

This in turn can be converted to a Blob by wrapping it in an array passing it to the Blobconstructor.

````js
var blob = new Blob([byteArray], {type: contentType});
````
The code above works. However the performance can be improved a little by processing the byteCharacters in smaller slices, rather than all at once. In my rough testing 512 bytes seems to be a good slice size. This gives us the following function.

````js
function b64toBlob(b64Data, contentType, sliceSize) {
  contentType = contentType || '';
  sliceSize = sliceSize || 512;

  var byteCharacters = atob(b64Data);
  var byteArrays = [];

  for (var offset = 0; offset < byteCharacters.length; offset += sliceSize) {
    var slice = byteCharacters.slice(offset, offset + sliceSize);

    var byteNumbers = new Array(slice.length);
    for (var i = 0; i < slice.length; i++) {
      byteNumbers[i] = slice.charCodeAt(i);
    }

    var byteArray = new Uint8Array(byteNumbers);

    byteArrays.push(byteArray);
  }

  var blob = new Blob(byteArrays, {type: contentType});
  return blob;
}

var blob = b64toBlob(b64Data, contentType);
var blobUrl = URL.createObjectURL(blob);

window.location = blobUrl;
````


### es6 code

````js
'use strict';

const b64toBlob = (b64Data, contentType='', sliceSize=512) => {
  const byteCharacters = atob(b64Data);
  const byteArrays = [];
  
  for (let offset = 0; offset < byteCharacters.length; offset += sliceSize) {
    const slice = byteCharacters.slice(offset, offset + sliceSize);
    
    const byteNumbers = new Array(slice.length);
    for (let i = 0; i < slice.length; i++) {
      byteNumbers[i] = slice.charCodeAt(i);
    }
    
    const byteArray = new Uint8Array(byteNumbers);
    
    byteArrays.push(byteArray);
  }
  
  const blob = new Blob(byteArrays, {type: contentType});
  return blob;
}


const contentType = 'image/png';
const b64Data = 'iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg==';

const blob = b64toBlob(b64Data, contentType);
const blobUrl = URL.createObjectURL(blob);

const img = document.createElement('img');
img.src = blobUrl;
document.body.appendChild(img);
````

