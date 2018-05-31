Easily allow to convert an HTML page into structured JSON data

# Installation

`npm i html-to-json-data`

# Usage

This module only provides convenient methods to transform an HTML page from string to JSON.
You'll have to fetch your pages through whatever mean you prefer

```js
const convert = require('html-to-json-data');
const { group, text, number, href, src, uniq } = require('html-to-json-data/definitions');

const html = '<html>...</html>'; // in this example https://github.com/piuccio?tab=repositories
const json = convert(html, {
  page: 'GitHub',
  name: text('.vcard-fullname'),
  nickname: text('.vcard-username'),
  avatar: src('img.avatar', 'https://github.com'),
  languages: uniq('span[itemprop="programmingLanguage"]'),
  repos: group('#user-repositories-list li', {
    name: text('h3'),
    link: href('h3 a', 'https://github.com'),
    stars: number('a[href$="stargazers"]'),
  }),
});
```

The resulting object looks like the following

```js
{
  page: 'GitHub',
  name: 'Fabio Crisci',
  nickname: 'piuccio',
  avatar: 'https://avatars1.githubusercontent.com/u/680284?s=460&v=4',
  languages: ['JavaScript', 'HTML', 'Python'],
  repos: [{
    name: 'cowsay',
    link: 'https://github.com/piuccio/cowsay',
    stars: 314,
  }, {
    name: 'flat-earth',
    link: 'https://github.com/piuccio/flat-earth',
    stars: 1,
  }], // the list goes on
}
```

Have a look at the tests for more detailed examples.


## Definitions

The functions exported by `html-to-json-data/definitions` allow to select data from the HTML page and convert it to your desired type.

They all take a selector as first parameter. Any selector that is valid for [cheerio](https://github.com/cheeriojs/cheerio#-selector-context-root-) will work.


### text

`text(selector)` return the text content (trimmed) of the selected node.

If the selector finds multiple nodes, it'll return an array with all selected values.


### uniq

`uniq(selector)` similar to `text` but always return an array of unique values.


### number

`number(selector)` convert the text content to a number, return 0 if the selector doesn't match any element


### attr

`attr(selector, name)` returns the value of the attribute `name` of the node selected by `selector`.

For instance if selector returns `<a title="Link" />`, the definition `attr('a', 'title')` will return `Link`.


### href

`href(selector, baseURI)` convenience method to return the value of the `href` attribute.

Similar to `attr(selector, 'href')` but it resolves relative paths from `baseURI`.


### src

`src(selector, baseURI)` convenience method to return the value of the `src` attribute.

Similar to `attr(selector, 'src')` but it resolves relative paths from `baseURI`.


### prop

`prop(selector, name)` similar to `attr` but returns a property of the node.

For instance in `<input type="checbox" />`, the definition `prop('input', 'checked')` will return `false`.


### data

`data(selector, name)` similar to `attr` but returns the data attribute.

For instance in `<div data-apple-color="pink" />`, the definition `data('div', 'apple-color')` will return `pink`.


### group

`group(selector, definitions)` creates a list of objects described by `definitions`.

The selectors inside `definitions` are scoped inside `selector`.

For instance `group('li', { title: text('h3') })` returns an array of objects with `title` extracted from `li h3`.