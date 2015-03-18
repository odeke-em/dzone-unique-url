## dzone-multiple

This module attempts to fix a usecase for [Dropzone](https://github.com/enyo/dropzone) which by default allows for multiple file uploads, but one common url.

## Motivation:

If for example you wanted the file to go to different domains that would fail.
    
A simple example is trying to  upload a file to AWS S3 using your browser after pre-signing its data.

If the original Dropzone is used, all files will be corrupted and only the last file will be uploaded.

Given the fact that you have shared state, xhr and url decoupling each is a problem.


```javascript
Dropzone.prototype.uploadFiles = function(files) {
      var file, formData, handleError, headerName, headerValue, headers, i, input, inputName, inputType, key, method, option, progressObj, response, updateProgress, url, value, xhr, _i, _j, _k, _l, _len, _len1, _len2, _len3, _m, _ref, _ref1, _ref2, _ref3, _ref4, _ref5;
      xhr = new XMLHttpRequest();
      for (_i = 0, _len = files.length; _i < _len; _i++) {
        file = files[_i];
        file.xhr = xhr;
      }
      method = resolveOption(this.options.method, files);
      url = resolveOption(this.options.url, files);
      xhr.open(method, url, true);
      xhr.withCredentials = !!this.options.withCredentials;
      response = null;
      handleError = (function(_this) {
        return function() {
          var _j, _len1, _results;
          _results = [];
          for (_j = 0, _len1 = files.length; _j < _len1; _j++) {
            file = files[_j];
            _results.push(_this._errorProcessing(files, response || _this.options.dictResponseError.replace("{{statusCode}}", xhr.status), xhr));
          }
          return _results;
        };
      })(this);
```

## Real life case:

You have a web app that accepts file uploads. To cut out bandwidth costs as well as memory

issues on your server, you probably would like the user's browser to handle this instead of first

uploading to your server, then your server moving the file over, then deleting it. Using for example

AWS S3 allows you to pre-sign a URL with an HMAC signature that AWS verifies on receiving the data.

With the original Dropzone, as previously explained the shared state and url means that uploading multiple

files fail for this case. Queueing files and calling processQueue() individually does not help.


This repo attempts to solve this problem.
