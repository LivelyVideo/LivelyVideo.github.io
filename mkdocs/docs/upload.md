# Lively Upload

## Installation
```bash
npm install @lively-video/upload --save
```

## Usage
```javascript
// With UI
import Upload from '@lively-video/upload';
const upload = new Upload(element, options);

// Without UI
import UploadCore from '@lively-video/upload/core';
const uploadCore = new UploadCore(element, options);
```

## Configuration
You can customize your Lively Upload by passing an options object when instantiating the library.

Example config:
```javascript
import Upload from '@lively-video/upload';

// new Upload(targetElement, options);
const myUpload = new Upload('#upload', {
	url: '/my-upload-route'
});
```

## Configuration Options

> You can supply Upload and UploadForm with either a DOM Element or a selector tag.

#### Upload
| Name 			| Type		| Description																							|
| ---			| ---		| ---																									|
| url 			| (string)	| (Required) Route to your upload api.	 																|
| authUrl 		| (string)	| (Required) Route to your auth api.	 																|
| token			| (string)	| Defaults to null. Use custom token.											|
| redirect		| (string)	| Defaults to null. The redirect route to your results template for cross-origin form compatibility.	|
| supportLegacy	| (bool)	| Defaults to true. Turn legacy support on or off.														|
| forceLegacy	| (bool)	| Defaults to false. Helpful option to force use of legacy form for debugging.							|
| autoSubmit	| (bool)	| Defaults to true. Automatically upload file when added to the uploader.								|
| https			| (bool)	| Defaults to false. Set protocol.								|
| MESSAGES 		| (object) 	| (Localization) Object containing list of message names and message strings.							|

#### UploadForm

> The UploadForm library includes Upload options in addition to these.

| Name 			| Type		| Description														|
| ---			| ---		| ---																|
| clickable 	| (bool)	| Defaults to true. UploadForm element opens file select on click.	|
| title 		| (string)	| Title text														|
| subtitle 		| (string)	| Subtitle text														|

## Events
The Upload library triggers events when processing files, which you can register to by listening to your instance.

Example listener:
```javascript
import Upload from '@livelyvideo/upload';

const myUpload = new Upload(element, {
	url: '/my-upload-route'
});

// myUpload.on(eventName, callbackFunction);
myUpload.on('start', () => {
	alert('My upload has started!');
});
```

## Event List
#### Upload
| Name 		| Params						| Description																|
| ---		| ---							| ---																		|
| start		| (file)						| Called at beginning of upload request.									|
| success	| (file) 						| Called after successful upload. 											|
| failure	| (file, reason)				| Called after upload failure. 												|
| progress 	| (currentBytes, totalBytes)	| Called after chunk has been uploaded.	(Not supported in legacy mode)		|
| thumbnail | (file, dataUrl)				| Called after thumbnail has been generated. (Not supported in legacy mode)	|

#### UploadForm

> The UploadForm library includes drag and drop events in addition to Upload events.

| Name 		| Params	| Description											|
| ---		| ---		| ---													|
| drop		| (event)	| The user has dropped something into the UploadForm.	|
| drag		| (event)	| The user is dragging.									|
| dragstart	| (event)	| The user started to drag.								|
| dragend	| (event)	| The user has stopped dragging.						|
| dragenter	| (event)	| The user dragged a file into the UploadForm.			|
| dragover	| (event)	| The user is dragging a file over the UploadForm.		|
| dragleave	| (event)	| The user dragged a file out of the UploadForm.		|

## Compatibility
* Chrome 7+
* Firefox 4+
* IE 10+
* Opera 12+
* Safari 6+

## Legacy Support

For browser that do not have access to HTML5 Drag and Drop and FileReader the uploader can falls back to an old form submission.

#### Cross-site iframe transport uploads
Legacy support async upload technique, by targeting an iframe on form submission.
Unfortunately, it is not possible to access the response body of iframes on a different domain.

Therefore Cross-site iframe transport uploads require a redirect back to a callback page on the origin server.  All this page needs to do is render json for the client to parse out the results of the upload.  The callback page should look like this:

```html
<!DOCTYPE HTML>
<html lang="en">
	<head>
		<meta charset="utf-8">
		<title>Result Page</title>
	</head>
	<body>
		<%= response %>
	</body>
</html>
```

The api will send back a response which will need to be decoded by your server. The result.html page adds the decoded result content as body content. This allows the plugin to access the response without cross-domain access issues.

Example redirect URL sent back to the client:
```
http://mydomain.com/result?%7Bstatus:200,reason:%7BSUCCESS_KEY%7D%7D
```
Example body content decoded by the result.html page:
```json
{"status":200,"reason":{'SUCCESS_KEY'}}
```



