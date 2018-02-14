React Styleguide
===

An opinionated stylegiude for building large projects with react/redux & server side rendering

## Directory Structure
* all shared components & containers are kept in `frontend/components`
* if as shared component has actions or reducer then it becomes a director, eg: `frontend/components/quick-search`
* components that are used within a single page are kept under `frontend/pages/[page-name]/components`
* all files and folders use `kebab-case` except for react components & style files which use `PascalCase`
```
├── __coverage__
├── __tests__
├── app.js // server entry point
├── common // utility functions common between backend and frontend
├── config
├── controllers
├── frontend
│   ├── action-types.js // enum of all redux action types
│   ├── build // destination for built js, css
│   ├── components // reusable UI components
│   ├── css-utils // shared css files
│   ├── layouts // layout components
│   ├── middleware // redux middleware
│   ├── pages // components for different pages
│   ├── root-reducer.js
│   ├── static // images, fonts,...
│   ├── utils // common frontend functions
│   └── views // html view templates
├── middleware // express middleware
├── react-ssr.js // entry point for react server side rendering
├── webpack.config.js
└── yarn.lock // commit & track yarn.lock
```
---

## React/Redux Conventions
* all components should extend `React.PureComponent` by default, which does a shallow compare of props and prevents rerendering if they haven't changes. Prefer this over statelss functions
* all components should use the translation helper `withTranslation` directly instead of passing it down from the container
* if a shared component will always call the same action, make it a connected component instead of having to pass the action everytime it's used
* instead of nesting complex conditionals & ternary checks in JSX, you can create variables with the partial component within the `render` method. These partial component variables should have names ending with `Partial`, eg:
```
// use this
render() {
  let customerPartial;
  if (customer) {
    customerPartial = (<Customer />)
  }
  return (<div> {customerPartial} </div>)
}

// instead of this, which can become hard to read with large components
render() {
  return (<div> {customer && <Customer />} </div>)
}
```
* all logic should be isolated in action creators while keeping components and reducers simple
* unit tests are prioritized for action creators, then reducers, then UI components
* all dispatched actions should have only 2 properties: `{ type, payload }`, where payload object contains all the action details
* prefer connected components over passing props several levels down the tree
* all analytics calls are handled by the redux analytics middleware `frontend/middleware/analytics`, which is split into different files each handling certain actions, eg: `product`, `checkout`, `cart`,...
* SEO changes to head tags like title, meta,... are implemented in the page component by using `HelmetWrapper` component which wraps the `react-helmet` library and adds some defaults. This will render the correct tags on server & client side. This approach is also used to add custom js/css files on the page level that cannot be imported by webpack.
---

## Styles
* css files should follow [suit css conventions](https://suitcss.github.io/)
* each `.jsx` component should import its own style file, so that requiring a react component will automatically add its styles.
* RTL support is achieved by using [rtlcss](http://rtlcss.com/), which automatically reverses all left/right rules in your css. To override this default behavior you can use [control directives](http://rtlcss.com/learn/usage-guide/control-directives/) or [value directives](http://rtlcss.com/learn/usage-guide/value-directives/). To keep these directives during webpack build, we must disable minification in all style loaders, which will be done by `WebpackRTLPlugin` eventually
* instead of requiring global css (variables, mixins, normalize) in each file, use a webpack loader that preprends this import to every `.less` file
* avoid using element type selectors like `a`, `span`,...
* icons are used from custom generated font file. Avoid using letter characters as content in font icons as it will show a random letter until the font loads
---

## Editor
* prefer vscode
* run this script to include the recommended extensions

```
code --install-extension adamvoss.yaml # yaml support
code --install-extension dbaeumer.vscode-eslint # eslint support
code --install-extension eamodio.gitlens # shows info from git about the last changes to a file/class
code --install-extension ms-vscode.sublime-keybindings # uses similar shortcuts to sublime text
code --install-extension shinnn.stylelint # css linting
code --install-extension wix.vscode-import-cost # shows the size of modules while importing
code --install-extension wmaurer.change-case # utility to transform case of text
code --install-extension CoenraadS.bracket-pair-colorizer # colors matching brackets
code --install-extension oderwat.indent-rainbow # adds colors for indentation levels
```
---

## Branching & Merge Requests
* branch names should start with `feature/` or `hotfix/`, followed by `JIRA-ID`, followed by an optional title. eg: `feature/NIS-453-homepage`
* each MR should be reviewed by 2 people and approved before merging. There can be some exceptions for urgent changes.
---

## Linting
* eslint template `.eslintrc`
```
"extends": "airbnb"
"parser": "babel-eslint"
"globals":
  "document": true
  "window": true
"rules":
  // propTypes are optional as they can slow down the development if components are changing rapidly,
  // prefer unit tests over propTypes
  "react/prop-types": 0
"env":
  "jest": true
```

* stylelintrc template `.stylelintrc`
```
"extends": "stylelint-config-standard"
```
---

## Bundle Size
* be careful while using lodash in frontend code as it can increase the bundle size. Always use `import` and destructure only the functions you need to avoid importing the whole library. Using `import` means that the file needs to have a `.jsx` file so it'll go through babel complier as node does not support `import` yet. You can't use `_.chain` on the frontend with this approach.
* running `yarn build` generates file `frontend/build/size-report.txt` which explains the size of all modules in the bundle
---

## Scripts
`package.json` should include the following scripts
* install `yarn`

* to start the server & webpack with watching changes: `yarn dev`

* to debug, use vscode configuration named "Attach to nodemon"

* to run linter for js & css `yarn lint`

* to run eslint `yarn js:lint`, to auto fix: `yarn js:lint --fix`

* to run stylelint `yarn css:lint`, to auto fix: `yarn css:lint --fix`

* to create a merge request from current local branch to a remote branch called `target` run `yarn mr target`. This should open the web page to create a merge request with the branches already selected.
---

## CI Pipeline
The CI server should perform the following tasks
  * build docker image
  * run js & css linter
  * run tests
  * check the size of the `vendors.js` to avoid accidentally bloating the bundle size
---

## Commit Message Format
Each commit message consists of a **header**, an optional **body** and an optional **footer**.
The header has a special format that includes a **type**, an optional **jira ticket id** or **scope** and a **subject**:

```
<type>(<jira ticket id>|<scope>)?: <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

Any line of the commit message cannot be longer 100 characters! This allows the message to be easier
to read on github as well as in various git tools.

### Type
Must be one of the following:

* **feat**: A new feature
* **fix**: A bug fix
* **docs**: Documentation only changes
* **style**: Changes that do not affect the meaning of the code (white-space, formatting, missing
  semi-colons, etc)
* **refactor**: A code change that neither fixes a bug or adds a feature
* **perf**: A code change that improves performance
* **test**: Adding missing tests
* **chore**: Changes to the build process or auxiliary tools and libraries such as documentation
  generation

### Scope
The scope could be anything specifying place of the commit change. For example `cart-controller`,
`auth-middleware`, etc...

### Subject
The subject contains succinct description of the change:

* use the imperative, present tense: "change" not "changed" nor "changes"
* don't capitalize first letter
* no dot (.) at the end

### Body
Just as in the **subject**, use the imperative, present tense: "change" not "changed" nor "changes"
The body should include the motivation for the change and contrast this with previous behavior.
