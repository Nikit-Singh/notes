# Node.js

- **http module** -> launch a server, send http requests
- **http.createServer()** -> creates a server that takes an callback function (c/d request listener) that keeps running till the program is closed manually.
- **request listener** -> (req, res) => { console.log(req) } -> this will perform the action based on the type of request it gets.
- **listen()** -> this assigns a path for the server where we can send the request and server can listen it.

```js
import http from 'http'

const serv = http.createServer((req, res) => {
	console.log(req.url, req.method, req.headers)
	res.setHeader('Content-Type', 'text/html')
	res.write('<html>')
	res.write('<head><title>My Node App</title></head>')
	res.write('<body><h1>My heading</h1></body>')
	res.write('</html>')
	res.end()
})

serv.listen(3000)
```

- **req.url** -> shows the url where the request is coming form => localhost:3000/test => /test
- **req.method** -> request method type => GET
- **req.headers** -> shows the header object => { host: 'localhost:3000', connections: 'bla bla' } 
- **res.setHeader** -> set the and type of data type to be sent.
- **res.write()** -> data to be sent is written to the respone object in multiple lines.
- **res.end()** -> to tell response that we are done writing data to it.
- **event loop** in js keeps running as long as there's an event listener running likr the callback inside the createServer in above example.
- **process.exit()** -> to exit from a process/event loop. eg: to exit from the event loop in js/nodejs.
  
```js
const http = require('http')
const fs = require('fs')

const server = http.createServer((req, res) => {
	const url = req.url
	const method = req.method
	if (url === '/') {
		res.write('<html>')
		res.write('<head><title>My Node App</title></head>')
		res.write(
			'<body><form action="/message" method="POST"><input type="text" name="test"/><button type="submit">Submit</button></form></body>'
		)
		res.write('</html>')
		return res.end()
	}

	// Here we are checking if the url is '/' then we will send html to show the particular data and we are returning res.end() so that further execution of program doesn't take place there because we cant do anything once res.end() is called.

	if (url === '/message' && method === 'POST') {
		fs.writeFileSync('message.txt', 'Baka')
		res.statusCode = 302
		res.setHeader('Location', '/')
		return res.end()
	}

	res.setHeader('Content-Type', 'text/html')
	res.write('<html>')
	res.write('<head><title>My Node App</title></head>')
	res.write('<body><h1>Welcome to My App</h1></body>')
	res.write('</html>')
	res.end()
})
// This will show for any other route 


server.listen(3000)
```

- **fs module** -> for file manipulation
- **res.statusCode** -> for setting up the response code after we created the file 
- **fs.writeFileSync** -> to create or append a file with some data -> fs.writeFileSync('filepath', 'data goes here')

```js
if (url === '/message' && method === 'POST') {
		const body = []
		req.on('data', chunk => {
			body.push(chunk)
		})
		return req.on('end', () => {
			const parsedData = Buffer.concat(body).toString()
			const par = parsedData.split('=')[1]
			const sp = par.replace(/\+/g, ' ')
			fs.writeFileSync('message.txt', sp)
			res.statusCode = 302
			res.setHeader('Location', '/')
			return res.end()
		})
}
```

- The **data** that comes **from request** is always in the **form of chunks**. We first **store** them and **move** them **to buffer** to do **operations** on them. Buffer is kind of bus stop for these chunks of data
- **req.on('req type', callback)** -> this is an event listener that listens to the type of request. req types => on, end, error, etc
- we **returned** req.on because other wise nodejs will **execute next lines** of code after the if statements as the **execution is asynchronous**.

```js
module.exports = functionName
```

- To **export function** in Nodejs.


# Express

- It's a **framework** for nodejs.
  
```js
const express = require('express')
const app = express()
//It exports a function as we can see in its definition.
```

- **Middleware functions** are the functions through which the request is funneled and finally the response is sent
  
```
REQUEST ----> MIDDLEWARE [ (res, req, next) => {...} ] --next()--> MIDDLEWARE [ (res, req, next) => {...} ] --res.send()--> RESPONSE ---->
```

- **app.use()** => It allows us to **add new middleware** function.

```js
const express = require('express')

const app = express()

app.use((req, res, next) => {
	console.log('In the first middleware')
	next();
})

app.use((req, res, next) => {
	console.log('In the next middleware')
	res.send('<h1>Using the ExpressJS</h1>')
	//Now we dont need to set the header as it automatically done by express and header of the above file will be automatically set to text/html
})

//If we dont use the next() function then the second app.use wont be executed as the first one keeps running.

app.listen(3000)
//this automatically creates the server and listens on the given port number by us and removes the necessity to use the createServer http function

```

- To **parse** the incoming request body (**req.body**) we use the **body-parser middleware**.

```js
const bodyParser = require('body-parser')

app.use(bodyParser.urlencoded())
// add {extended: false} inside the urlencoded() we it expects as defaults and shows warning if not passed
```

- **res.redirect('/path')** -> a function by express to **redirect the path.**

```js
const http = require('http')
const express = require('express')
const bodyParser = require('body-parser')

const app = express()

app.use(bodyParser.urlencoded({ extended: false }))

app.use('/add-product', (req, res) => {
	res.send(
		'<form action="/product" method="POST"><input type="text" name="data"/><button type="submit">Submit</button></form>'
	)
})

app.use('/product', (req, res) => {
	console.log(req.body)
	// this will show what's sent through the form 
	res.redirect('/')
})

app.use('/', (req, res) => {
	res.send('<h1>Hello Mate!</h1>')
})

app.listen(3000)
```

- app.post(), app.get(), app.delete(), app.put(), app.patch() should be used instead of app.use() to specify what type of request can use the particular path.
  
## Routing
> We create a routes folder in the main directory where we specify the routes based on our needs.

### ./routes/admin.js
```js
const express = require('express')

const router = express.Router()
// This registers the router function for this module

router.get('/add-product', (req, res) => {
	// now we use router instead of app here in the file
	res.send(
		'<form action="/admin/add-product" method="POST"><input type="text" name="data"/><button type="submit">Submit</button></form>'
	)
	// we are using action as /admin/add-product because the original path starts with /admin only and we are just filtering it in the index.js file
})

router.post('/add-product', (req, res) => {
	console.log(req.body)
	res.redirect('/')
})

module.exports = router
// we just need to export the router now instead of all functions one by one
```

### ./routes/shop.js
```js 
const express = require('express')

const router = express.Router()

router.get('/', (req, res) => {
	res.send('<h1>Hello Mate!</h1>')
})

module.exports = router
```

### ./index.js
```js
const express = require('express')
const bodyParser = require('body-parser')

const adminRoutes = require('./routes/admin')
const shopRoutes = require('./routes/shop')
// we just import the routes like we import normal modules

const app = express()

app.use(bodyParser.urlencoded({ extended: false }))

// the imported routes are middleware functions so we can directly use them inside the app.use function without () brackets as they are already imported as functions
app.use('/admin', adminRoutes)
// this '/admin' is called filtering. Here we are filtering the routes that starts with /admin and checking them.
app.use(shopRoutes)

// to set a 404 page we use this at the end of the file to force any non found links to come here
app.use((req, res) => {
	res.status(404).send('<h1>Page Not Found</h1>')
})

app.listen(3000)
```

- **res.status(404).send()** -> status **sets** the **response status number** and we can **chain** it with **other functions** like send() here.

- **path module** -> Inbuilt nodejs module that **helps** in **working with** the **file paths** of a system.

```js
const path = require('path')
const express = require('express')

const router = express.Router()

router.get('/', (req, res) => {
	res.sendFile(path.join(__dirname, '..', 'views', 'shop.html'))
})
// we are using path.join instead of normally concatenating the path like /views/shop.html because the path is different for both windows and unix type systems. To go back up one directory we use '..' to do so. 

module.exports = router
```

- **res.sendFile()** -> Used when we want to **send a file instead** of just **one line**.
- **path.join()** -> To **join** the **paths based** on the type of **operating system**.
- **__dirname** -> It gives the current **location of our file** in the **machine** we are **working in**.

```js
app.use(express.static(path.join(__dirname, 'public')))

//This will make nodejs to treat the public folder as a static folder where it can look for css or js files and we can just import the css normally in the html files by linking it.
```

