# GTMS-4

## Template code formatting

### Problem that solved by this standard

GTM doesn't have a built-in code formatter.
This leads to the situation when the code in the template is not formatted in the same way.
This makes it difficult to read and understand the code.

### Standard description

The code in the template should be formatted using the [Prettier](https://prettier.io/) tool.

The standard Prettier configuration for template repositories is

```
{
    singleQuote: 'true',
    traillingComma: 'none',
    tabWidth: 2,
    printWidth: 100
}
```

and should be in a `.prettierrc` file in root-level.

You can also run an npx command to format the file with these configurations, although **it is recommended using the configuration file instead**.

### Example of use

```shell
npx prettier --single-quote --trailing-comma none --tab-width 2 --print-width 100 --write template.js
```
