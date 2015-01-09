#Swagger to JS Codegen (for ServiceStack's Swagger API)

This package generates a nodejs or angularjs class from a [swagger specification file](https://github.com/wordnik/swagger-spec). The code is generated using [mustache templates](https://github.com/wcandillon/swagger-js-codegen/tree/master/lib/templates) and is quality checked by [jshint](https://github.com/jshint/jshint/) and beautified by [js-beautify](https://github.com/beautify-web/js-beautify).

##Installation
```bash
npm install swagger-js-codegen
```

##Example - using NodeJs to generate client js file
```javascript
var http = require('http');
var fs = require('fs');
var util = require('util');
var CodeGen = require('swagger-js-codegen').CodeGen;

var BASE_URL = 'http://127.0.0.1:10080';


function downloadJson(url, callback){
	http.get( BASE_URL + url, function(response) {
		var body = '';
		response.on('data', function(chunk) {
			body += chunk;
		});
		response.on('end', function() {
			console.log(body);
			if( typeof(callback) === 'function' ){
				callback(JSON.parse(body));
			}
		});
	});
}


var count = 0;
downloadJson('/resources', function(manifest){
	if( typeof(manifest) === 'object' && util.isArray(manifest.apis) ){
		count = manifest.apis.length;
		var swagger = 
		{
			swaggerVersion : '1.1',
			apiVersion : '1.0',
			basePath : BASE_URL,
			apis : []
		};

		for( var i = 0; i < manifest.apis.length; i++ ){
			var path = manifest.apis[i].path;
			
			downloadJson( path, function(spec){

				spec.apis.forEach( function(el, idx){
					swagger.apis.push(el);
				});
				


				if( --count <= 0 ){
					//getNodeCode 
					var sourceCode = CodeGen.getAngularCode({ moduleName: 'ApiModule', className: 'CmsApi', swagger: swagger }); 

					fs.writeFile("output.js", sourceCode, function(){
						process.exit();
					});
					
				}
			});
		}
	}
});
```

## Grunt task
[There is a grunt task](https://github.com/wcandillon/grunt-swagger-js-codegen) that enables you to integrate the code generation in your development pipeline. This is extremely convenient if your application is using APIs which are documented/specified in the swagger format.

##Who is using it?
The [CellStore](https://github.com/28msec/cellstore) project.

[28.io](http://28.io) is using this project to generate their [nodejs](https://github.com/28msec/28.io-nodejs) and [angularjs language bindings](https://github.com/28msec/28.io-angularjs).
