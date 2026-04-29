# GTMS-5

## Template repository file structure

### Problem that solved by this standard

There are no standards for the structure of the repository.
Because of this, the repository's structure is different for each project.

There is no requirement to store the code of a template in a separate file, however, it is hard to read the code of the `template.tpl` if it is not extracted to a separate file. This standard requires the template code to be stored in a separate file (`template.js`), which makes it easier to read and maintain.

It also requires the repository to contain a `README.md` file, which is used to describe the template and provide instructions on how to use it. And a `.prettierrc` file, which is used to maintain a consistent code style across the repository.

And finally, it requires the repository to contain the workflow files (in `.github/workflows`) referencing the [shared workflows repository](https://github.com/stape-io/template-changes-shared-workflows/tree/main), which is used to notify template and README changes to Stape Community and perform GTM Gallery Status checks.

Optionally, the repository can contain a `tests.yaml` file (if any tests exist), which is used to store the tests of the template in a separate file.

### Standard description

#### Required by GTM Template Gallery
- `LICENSE`
- `metadata.yaml`
- `template.tpl`

#### Required by this standard
- `template.js` - template code formatted with [GTMS-4 standard](standards/GTMS-4.md)
- `README.md` - template description
- `.prettierrc` - Prettier repo-level configuration
  ```json
  {
    "singleQuote": true,
    "trailingComma": "none",
    "tabWidth": 2,
    "printWidth": 100
  }
  ```
- `tests.yaml` - tests used in the template
- `.github/workflows/discourse-notify-and-readme-sync.yml` - workflow for template changes notifications to Stape Community and README updates. [Workflow reference](https://github.com/stape-io/template-changes-shared-workflows/tree/main/gtm-templates-specific-workflows/.github/workflows).
- `.github/workflows/gallery-status.yml` - workflow for GTM Gallery Status checks twice a week. [Workflow reference](https://github.com/stape-io/template-changes-shared-workflows/tree/main/gtm-templates-specific-workflows/.github/workflows).

##### `tests.yaml` File

The tests inside `template.tpl` are written in .yaml syntax. Given that, the file `tests.yaml` must be a copy of what is in the `template.tpl` under `__TESTS__`.

Observation: when exporting a template, the tests codes are formatted by GTM into an inline compact hard way to read. Before pushing the `tests.yaml` file, make sure the tests codes are formatted properly using yaml code block sintax (`|-`). See illustration below:

**Correct Format**

<img src="../images/gtms-5-correct.png" width="50%" alt="Correct Format"/>

**Wrong Format**

<img src="../images/gtms-5-wrong.png" width="40%" alt="Wrong Format"/>