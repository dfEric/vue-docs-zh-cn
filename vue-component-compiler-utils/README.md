# @vue/component-compiler-utils

[英文原版](https://github.com/vuejs/component-compiler-utils)

> Lower level utilities for compiling Vue single file components

This package contains lower level utilities that you can use if you are writing a plugin / transform for a bundler or module system that compiles Vue single file components into JavaScript. It is used in [vue-loader](https://github.com/vuejs/vue-loader) version 15 and above.

The API surface is intentionally minimal - the goal is to reuse as much as possible while being as flexible as possible.

## API

### parse(ParseOptions): SFCDescriptor

Parse raw single file component source into a descriptor with source maps.

``` ts
interface ParseOptions {
  source: string
  filename?: string
  sourceRoot?: string
  needMap?: boolean
}

interface SFCDescriptor {
  template?: SFCBlock
  script?: SFCBlock
  styles: SFCBlock[]
  customBlocks: SFCCustomBlock[]
}

interface SFCCustomBlock {
  type: string
  content: string
  attrs: { [key: string]: string }
  start: number
  end: number
  map: RawSourceMap
}

interface SFCBlock extends SFCCustomBlock {
  lang?: string
  src?: string
  scoped?: boolean
  module?: string | boolean
}
```

### compileTemplate(TemplateCompileOptions): TemplateCompileResults

Takes raw template source and compile it into JavaScript code. The actual compiler (`vue-template-compiler`) must be passed so that the specific version used can be determined by the end user.

It can also optionally perform pre-processing for any templating engine supported by [consolidate](https://github.com/tj/consolidate.js/).

``` ts
interface TemplateCompileOptions {
  source: string
  filename: string

  // See https://github.com/vuejs/vue/tree/dev/packages/vue-template-compiler
  compiler: VueTemplateCompiler
  compilerOptions?: VueTemplateCompilerOptions

  // Template preprocessor
  preprocessLang?: string
  preprocessOptions?: any

  // Transform asset urls found in the template into `require()` calls
  // This is off by default. If set to true, the default value is
  // {
  //   video: ['src', 'poster'],
  //   source: 'src',
  //   img: 'src',
  //   image: 'xlink:href'
  // }
  transformAssetUrls?: AssetURLOptions | boolean

  // For vue-template-es2015-compiler, which is a fork of Buble
  transpileOptions?: any

  isProduction?: boolean  // default: false
  isFunctional?: boolean  // default: false
  optimizeSSR?: boolean   // default: false
}

interface TemplateCompileResult {
  code: string
  source: string
  tips: string[]
  errors: string[]
}

interface AssetURLOptions {
  [name: string]: string | string[]
}
```

#### Handling the Output

The resulting JavaScript code will look like this:

``` js
var render = function (h) { /* ... */}
var staticRenderFns = [function (h) { /* ... */}, function (h) { /* ... */}]
```

It **does NOT** assume any module system. It is your responsibility to handle the exports, if needed.

### compileStyle(StyleCompileOptions)

Take input raw CSS and applies scoped CSS transform. It does NOT handle pre-processors. If the component doesn't use scoped CSS then this step can be skipped.

``` ts
interface StyleCompileOptions {
  source: string
  filename: string
  id: string
  map?: any
  scoped?: boolean
  trim?: boolean
}

interface StyleCompileResults {
  code: string
  map: any | void
  rawResult: LazyResult | void // raw lazy result from PostCSS
  errors: string[]
}
```
