# mongoose-json-patch
Adds JSON Patch (RFC 6902) support to Mongoose models.

###**[Don't patch like an idiot.](http://williamdurand.fr/2014/02/14/please-do-not-patch-like-an-idiot/)**

## Usage

`npm install --save mongoose-json-patch`

**mongoose-json-patch** is used as a [Mongoose plugin](http://mongoosejs.com/docs/plugins.html).

```javascript
var mongoose = require('mongoose'),
	patcher = require('mongoose-json-patch');

var CarSchema = new mongoose.Schema({ ... });
CarSchema.plugin(patcher);

mongoose.model('Car', CarSchema);
```

Your documents now have a **`patch`** method that accepts an array of JSON patches and a callback method. The patches are applied, the document is saved, and any patch errors are returned to the callback.

```javascript
var myCar = new Car({
	make: 'Honda',
	model: 'CR-V',
	year: 1999
});

var patches = [
	{ op: 'replace', path: '/model', value: 'Civic' }
];

myCar.patch(patches, function callback(err) {
	if(err) return next(err);
	res.send(myCar);

	/* myCar:
		{
			make: 'Honda',
			model: 'Civic',
			year: 1999
		}
	*/
});
```

### Preventing Writes

Defining any schema paths as **`writable: false`** will return an error if a patch tries to modify that path. `_id` and `__v` are blocked by default.

```javascript
var CarSchema = new mongoose.Schema({

	owner: {type: String, writable: false}
	
});
```

----

## Bonus
### Read Protection

There are some properties that you never want to leave the server (like passwords), and you can use Mongoose's `select` [query method](http://mongoosejs.com/docs/api.html#query_Query-select) to filter out certain properties. However, this will not work if you wanted to work with all properties server-side before sending them to the client. For that, this plugin also adds a `filterProtected` method to all documents. It will return a copy of your document, removing any properties that have been defined as `readable: false` in the schema.

```javascript
var CarSchema = new mongoose.Schema({
	make: {type: String},
	key: {type: String, readable: false}
});

var Car = mongoose.model('Car', CarSchema);

var myCar = new Car({make: 'Honda', model: 'CR-V', key: 'secret15829a'});

myCar; // {make: 'Honda', model: 'CR-V', key: 'secret15829a'}
myCar.filterProtected(); // {make: 'Honda', model: 'CR-V'}
```

Passing any arguments will filter additional properties.

```javascript
myCar.filterProtected('make'); // {model: 'CR-V'}
```

If you have an array of documents that you need to filter, there is a static convenience method added to the model.

```javascript
Car.filterProtected(carsArray);
```