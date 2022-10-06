# Enable Compodoc for Storybook on Nx

This is a sample repository showing how you can enable `compodoc` for Storybook on Nx.

## Main concepts

- `compodoc` is a documentation generator for Angular applications. You can read more on the [compodoc](https://compodoc.app/) website.
- `storybook` is a tool for developing UI components in isolation for React, Vue, and Angular. You can read more on the [storybook](https://storybook.js.org/) website.

In your Angular applications, you may add comment blocks above your components, directives, and other Angular constructs. These comments are used by `compodoc` to generate documentation for your application.

For example:

```
  /**
   * What background color to use
   */
  @Input()
  backgroundColor?: string;
```

This comment block will be used by `compodoc`. The text `What background color to use` will be used as the description for the `backgroundColor` input property.

Storybook for Angular uses `compodoc` to generate documentation for your components. This is a great way to document your components, without needed to make any special customizations, or write too much code.

The main things that you need to do are:

1. Tell Storybook to include the component files in the typescript compilation.
2. Use `compodoc` to generate a `documentation.json` file
3. Tell Storybook to use the `documentation.json` file to display the documentation.

Let's see how you can do that.

Note: This repository consists of an Angular application with Storybook configured and an Angular library with Storybook configured. If you do not know how to set these up, please read about [setting up Storybook for Angular](https://nx.dev/storybook/overview-angular) on the Nx documentation website.

## Steps to set up compodoc

### 1. Install the necessary packages

```bash
yarn add -D @compodoc/compodoc @storybook/addon-docs
```

or

```bash
npm install --save-dev @compodoc/compodoc @storybook/addon-docs
```

### 2. Include the component files in the typescript compilation

In your project's `.storybook/tsconfig.json` file, in the `include` array, add the path to the component files (eg. `"../src/**/*.component.ts"`). For example, if you have an application called `web`:

For the file `apps/web/.storybook/tsconfig.json`:

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "emitDecoratorMetadata": true
  },
  "files": ["../src/polyfills.ts"],
  "exclude": ["../**/*.spec.ts"],
  "include": ["../src/**/*.stories.ts", "../src/**/*.component.ts", "*.js"]
}
```

Important! If you are importing stories from other places, and you want the docs for these stories to be generated, too, you need to add the paths to the other components in the `include` array, as well!

For example, if your stories import paths look like this:

```
"../../../**/**/src/lib/**/*.stories.ts"
```

make sure to also include

```
"../../../**/**/src/lib/**/*.component.ts"
```

for the components to be included in the TypeScript compilation as well.

### 3. Enable `compodoc` and configure it

#### a. Set `compodoc: true`

In your project's `project.json` file (eg. `apps/web/project.json`), find the `storybook` and the `build-storybook` targets.

In the `options` you will see `"compodoc": false`. Change that to `true`.

#### b. Set the directory

By default, `compodoc` will generate a `documentation.json` file at the root of your workspace. We want to change that, and keep the documentation file project-specific. Of course you can change that later, or as you see fit for your use case. But let's keep it project-specific for now.

In your project's `project.json` file (eg. `apps/web/project.json`), find the `storybook` and the `build-storybook` targets. Below the `"compodoc"` option, create a new option called `"compodocArgs`" which contains the following: `["-e", "json", "-d", "apps/web"]`. This means that the `exportFormat` (`-e`) will be `json` and the `output` directory (`-d`) will be `apps/web` (change that, of course, to the directory of your project).

Let's see the result for our `web` app, for example:

```json
    "storybook": {
      "executor": "@storybook/angular:start-storybook",
      "options": {
        "port": 4400,
        "configDir": "apps/web/.storybook",
        "browserTarget": "web:build",
        "compodoc": true,
        "compodocArgs": ["-e", "json", "-d", "apps/web"]
      },
    },
```

### 4. Let Storybook know of the `documentation.json` file (of the compodoc path)

In your project's `.storybook/preview.js` file (for example for your `web` app the path would be `apps/web/.storybook/preview.js`), add the following:

```js
import { setCompodocJson } from '@storybook/addon-docs/angular';
import docJson from '../documentation.json';
setCompodocJson(docJson);
```

## Now run Storybook and see the results!

Now you can run Storybook or build Storybook, and documentation will be included!

```bash
nx storybook web
```

and

```bash
nx build-storybook web
```
