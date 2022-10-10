# GTMS-3
## Template code formatting

### Problem that solved by this standard

GTM doesn't have a built-in code formatter. 
This leads to the situation when the code in the template is not formatted in the same way. 
This makes it difficult to read and understand the code.

### Standard description

The code in the template should be formatted using the [Prettier](https://prettier.io/) tool.

### Example of use

```shell
npx prettier --single-quote --write template.js
```
