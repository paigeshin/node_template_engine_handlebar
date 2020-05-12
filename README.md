# node_template_engine_handlebar

- Package.json

```json
"dependencies": {
    "express": "^4.16.3",
    "express-handlebars": "^3.0.0",
    "hbs": "^4.1.1",
  }
```

- installation

```jsx
const expressHbs = require('express-handlebars');

const app = express();

app.engine('handlebars', expressHbs()); //default로 node에 있지 않은 engine을 쓰면 `app.engine` 을 써서 명시해준다.
app.set('view engine', 'hbs');
app.set('views', 'views');
```

- key-value 값 가져오기

```jsx
<title>{{ pageTitle }}</title>
```

- conditional statement + foreach

```jsx
{{#if hasProducts }} <!-- Logic을 node에 강제한다. 즉, prods.length > 0 같은 것을 사용할 수 없다. -->
    <div class="grid">
        {{#each prods}}
        <article class="card product-item">
            <header class="card__header">
                <h1 class="product__title">{{ this.title }}</h1>
            </header>
            <div class="card__image">
                <img src="https://image.winudf.com/v2/image/bW9iaS5hbmRyb2FwcC5teXN0ZXJ5aGF0LmM4MDQxX3NjcmVlbl8zXzE1MjM1MjMyMjZfMDQy/screen-3.jpg?fakeurl=1&type=.jpg" alt="A Book">
            </div>
            <div class="card__content">
                <h2 class="product__price">$19.99</h2>
                <p class="product__description">A very interesting book about so many even more interesting things!</p>
            </div>
            <div class="card__actions">
                <button class="btn">Add to Cart</button>
            </div>
        </article>
        {{/each}}
    </div>
{{ else }}
    <h1>No product Found</h1>
{{/if}}
```

- placeholder

```jsx
<!-- Placeholder -->
{{{ body }}}
```

- layout 추가 app engine 설정

```jsx
app.engine('hbs', expressHbs({
    layoutsDir: 'views/layouts/',
    defaultLayout: 'main-layout',
    extname: 'hbs'
})); //default로 node에 있지 않은 engine을 쓰면 `app.engine` 을 써서 명시해준다.
app.set('view engine', 'hbs');
app.set('views', 'views');
```

- layout (header)

```jsx
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>{{ pageTitle }}</title>
    <link rel="stylesheet" href="/css/main.css">
    <!-- CSS를 dynamic 하게 rendering -->
    {{#if formsCSS}}
        <link rel="stylesheet" href="/css/forms.css">
    {{/if}}
    {{#if formsCSS}}
        <link rel="stylesheet" href="/css/product.css">
    {{/if}}
</head>

<body>
<header class="main-header">
    <nav class="main-header__nav">
        <ul class="main-header__item-list">
            <li class="main-header__item"><a class="{{#if activeShop}} active {{/if}}" href="/">Shop</a></li>
            <li class="main-header__item"><a href="{{#if activeAddProduct}} active {{/if}}">Add Product</a></li>
        </ul>
    </nav>
</header>

<!-- Placeholder -->
{{{ body }}}

</body>

</html>
```

- router for dynamic rendering, `shop.js`

```jsx
const path = require('path');

const express = require('express');

const adminData = require('./admin');

const router = express.Router();

router.get('/', (req, res, next) => {
  const products = adminData.products;
  res.render('shop', {
    prods: products,
    pageTitle: 'Shop',
    path: '/',
    hasProducts: products.length > 0,
    activeShop: true,
    productCSS: true
  })
});

module.exports = router;
```

- router for dynamic rendering, `admin.js`

```jsx
const path = require('path');

const express = require('express');

const router = express.Router();

const products = [];

// /admin/add-product => GET
router.get('/add-product', (req, res, next) => {
    res.render('add-product', {
        pageTitle: 'Add Product',
        path: '/admin/add-product',
        formsCSS: true,
        productCSS: true,
        activeAddProduct: true
    });
});

// /admin/add-product => POST
router.post('/add-product', (req, res, next) => {
    products.push({title: req.body.title});
    res.redirect('/');
});

exports.routes = router;
exports.products = products;
```

- layout에 {{{ body }}} 를 추가했다면, 사용하는 곳에는 그냥 코드만 작성하면 된다.

404.hbs

```jsx
<header class="main-header">
    <nav class="main-header__nav">
        <ul class="main-header__item-list">
            <li class="main-header__item"><a class="active" href="/">Shop</a></li>
            <li class="main-header__item"><a href="/admin/add-product">Add Product</a></li>
        </ul>
    </nav>
</header>
<h1>Page Not Found!</h1>
```

add-product.hbs

```jsx
<main>
    <form class="product-form" action="/admin/add-product" method="POST">
        <div class="form-control">
            <label for="title">Title</label>
            <input type="text" name="title" id="title">
        </div>

        <button class="btn" type="submit">Add Product</button>
    </form>
</main>
```

shop.hbs

```jsx
<main>
    {{#if hasProducts }} <!-- Logic을 node에 강제한다. 즉, prods.length > 0 같은 것을 사용할 수 없다. -->
        <div class="grid">
            {{#each prods}}
            <article class="card product-item">
                <header class="card__header">
                    <h1 class="product__title">{{ this.title }}</h1>
                </header>
                <div class="card__image">
                    <img src="data:image/jpeg;base64,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" alt="A Book">
                </div>
                <div class="card__content">
                    <h2 class="product__price">$19.99</h2>
                    <p class="product__description">A very interesting book about so many even more interesting things!</p>
                </div>
                <div class="card__actions">
                    <button class="btn">Add to Cart</button>
                </div>
            </article>
            {{/each}}
        </div>
    {{ else }}
        <h1>No product Found</h1>
    {{/if}}
</main>
```

### Basics

- handlebars는 데이터를 보여주는 것만 가능하다. 즉 모든 business logic은 node.js 서버로부터 와야한다.
- 데이터를 가져올 때는 {{ }}
- placeholder {{{ }}}
