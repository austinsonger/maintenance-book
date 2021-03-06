# Customizing ESLint

Even though you can get far with vanilla ESLint, there are certain techniques you should know. For instance, sometimes you want to skip specific rules per file. You can even implement rules of your own.

## Speeding up ESLint Execution

One of the most convenient ways to speed up ESLint execution on big projects is to run it on only files that have been changed while you are working. It’s possible to achieve this by using _lint-staged_. The exact technique is covered in the _Automation_ chapter.

Node comes with startup overhead and it takes a while for the processing to begin. [eslint_d](https://www.npmjs.com/package/eslint_d) is a daemon process designed to overcome this problem. It runs ESLint as a process in the background. [esprint](https://www.npmjs.com/package/esprint) is a similar approach. It runs ESLint across multiple threads parallel.

T> You can find [more technical details about esprint in its introduction post](https://medium.com/@Pinterest_Engineering/introducing-esprint-a-fast-open-source-eslint-cli-19a470cd1c7d).

{pagebreak}

## Skipping ESLint Rules

ESLint allows you to skip rules on several levels. Consider the following examples:

<!-- textlint-disable -->

```javascript
// everything
/* eslint-disable */
// ...
/* eslint-enable */
```

```javascript
// specific rule
/* eslint-disable no-unused-vars */
// ...
/* eslint-enable no-unused-vars */
```

```javascript
// tweaking a rule
/* eslint no-comma-dangle:1 */
```

```javascript
// disable rule per line
alert('foo'); // eslint-disable-line no-alert
```

<!-- textlint-enable -->

The rule specific examples assume you have the rules in your configuration in the first place! You cannot specify new rules here. Instead, you can modify the behavior of existing rules.

## Setting Environment

Sometimes, you want to run ESLint in a specific environment, such as Node or Mocha. These environments have certain conventions of their own. For instance, Mocha relies on custom keywords (e.g., `describe`, `it`) and it’s good if the linter doesn’t choke on those.

ESLint provides two ways to deal with this: local and global. To set it per file, you can use a declaration at the beginning of a file:

<!-- textlint-disable -->

```javascript
/*eslint-env node, mocha */
```

Global configuration is possible as well. In this case, you can use `env` key:

**.eslintrc.json**

```json
{
  "env": {
    "browser": true,
    "commonjs": true,
    "es6": true,
    "node": true
  }
  // ...
}
```

<!-- textlint-enable -->

## Writing ESLint Plugins

ESLint plugins rely on Abstract Syntax Tree (AST) definition of JavaScript. It’s a data structure that describes JavaScript code after it has been lexically analyzed. There are tools, such as [recast](https://github.com/benjamn/recast), that allow you to perform transformations on JavaScript code by using AST transformations. The idea is that you match a structure, then transform it somehow and convert AST back to JavaScript.

### Understanding AST

To get a better idea of how AST works and what it looks like, you can check [Esprima online JavaScript AST visualization](http://esprima.org/demo/parse.html) or [AST Explorer by Felix Kling](http://astexplorer.net/). Alternately, you can install `recast` and examine the output it gives. You work with the structure for ESLint rules.

T> [Codemod](https://github.com/facebook/codemod) allows you to perform large-scale changes to your codebase through AST based transformations.

### Writing a Plugin

In ESLint’s case, the AST structure can be checked. If something is wrong, it should let you know. Follow the steps below to set up a plugin:

1. Set up a new project named `eslint-plugin-custom`. You can replace `custom` with something else. ESLint follows this naming convention.
1. Execute `npm init -y` to create a dummy _package.json_
1. Set up `index.js` in the project root with content.

You can get started with a skeleton as below:

**eslint-plugin-custom/index.js**

```javascript
module.exports = {
  rules: {
    demo: {
      docs: {
        description: 'Demo rule',
        category: 'Best Practices',
        recommended: true,
      },
      schema: [
        {
          type: 'object',
          // JSON Schema to describe properties
          properties: {},
          additionalProperties: false,
        },
      ],
      create(context) {
        return {
          Identifier(node) {
            context.report(node, 'This is unexpected!');
          },
        };
      },
    },
  },
};
```

In this case, you report for every identifier found. In practice, you likely want to do something more complex than this, but this is a good starting point.

Next, you need to execute `npm link` within `eslint-plugin-custom` to make your plugin visible to your system. `npm link` allows you to consume a development version of a library you are developing. To reverse the link, you can execute `npm unlink` when you feel like it.

You need to alter the project configuration to make it find the plugin and the rule within.

**.eslintrc.json**

<!-- textlint-disable -->

<!-- skip-example -->

```json
{
  // ...
  "plugins": [
    // ...
leanpub-start-insert
    "custom"
leanpub-end-insert
  ],
  "rules": {
    // ...
leanpub-start-insert
    "custom/demo": 1,
leanpub-end-insert
  }
}
```

<!-- textlint-enable -->

If you invoke ESLint now, you should see a bunch of warnings. Of course, the rule doesn’t do anything impressive yet. To move forward, check out the [official plugin documentation](http://eslint.org/docs/developer-guide/working-with-plugins.html) and [rules](http://eslint.org/docs/developer-guide/working-with-rules.html).

You can also check out the existing rules and plugins for inspiration to see how they achieve certain things. ESLint allows you to [extend these rulesets](http://eslint.org/docs/user-guide/configuring.html#extending-configuration-files) through `extends` property. It accepts either a path to it (`"extends": "./node_modules/coding-standard/.eslintrc.json"`) or an array of paths. The entries are applied in the given order, and later ones override the former.

## ESLint Resources

Besides the official documentation available at [eslint.org](http://eslint.org/), you should check out the following blog posts:

* [Detect Problems in JavaScript Automatically with ESLint](http://davidwalsh.name/eslint) - A good tutorial on the topic.
* [Understanding the Real Advantages of Using ESLint](http://rangle.io/blog/understanding-the-real-advantages-of-using-eslint/) - Evan Schultz’s post digs into details.
* [eslint-plugin-smells](https://www.npmjs.com/package/eslint-plugin-smells) - This plugin by Elijah Manor allows you to lint against JavaScript smells. Recommended.

If you want a starting point, you can pick one of [eslint-config- packages](https://www.npmjs.com/search?q=eslint-config) or go with the [standard](https://www.npmjs.com/package/standard) style.

## Conclusion

You can customize ESLint to various purposes. Thanks to its vibrant ecosystem, it’s likely you find rules that are close to your purposes. Those make excellent starting points for your own development.

To recap:

* ESLint allows rules to be skipped locally. Use this feature sparingly.
* You can override ESLint environment per file. It’s good to consider other approaches if you notice a lot of overrides in your source, though.
* ESLint can be extended through plugins. They allow you to adjust its behavior to your liking. Given the ecosystem is strong, check it first before going this way.
