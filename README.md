### About this project

This is the new electron GUI for Guus forked from Ryo, Oxen. It is open source and completely free to use without restrictions, anyone may create an alternative implementation of the guus Electron GUI that uses the protocol and network in a compatible manner.

Please submit any changes as pull requests to the development branch, all changes are assessed in the development branch before being merged to master, release tags are considered stable builds for the GUI.

#### Pre-requisites
- Download latest [guusd](https://github.com/framepix/guus-core/releases/tag/v12.0.0)

#### Commands
```
nvm use 11.9.0
npm install -g quasar-cli
git clone https://github.com/framepix/Guus-GUI
cd guus-electron-wallet
cp path_to_guus_binaries/guusd bin/
cp path_to_guus_binaries/guus-wallet-rpc bin/
npm install
quasar build -m electron -t mat
```

### Adding language support

Adding a new language is fairly simple.

1. Duplicate the language file `src/i18n/en-us.js` and rename it to the relevant language code.
2. Translate all the strings in that duplicated file. Take note that capitalization matters.
    - The translated string must go in-between the quotes (`""`)
      - E.G `all: "ALL"` -> `all: "ВСЕ"`
    - If possible try and stick to the general string formatting already present.
      - E.G if there is a new line then try and keep that in your translation.
      - The same goes for the pipe character `|`. **DO NOT REMOVE IT**.
    - Please don't translate strings inside `{}` brackets. They are meant as placeholders for other values.
      - Some examples include `{type}` and `{count}`.
      - E.G if you have a string `A {index}` then you may translate it as `B {index}` or `{index} B` depending on how the string makes sense in your language. You are allowed to reposition the placeholders for the string to make sense **BUT DO NOT DELETE OR REPLACE THE PLACEHOLDERS WITH OTHER VALUES**
3. Add the language to the `languages` array in `src/i18n/index.js`. The `flag` property is the [ISO 3166-1-alpha-2 code](https://www.iso.org/obp/ui/#search/code/) of a country.
   - **NOTE: DO NOT ADD THE LANGUAGE TO `export default`**. Dynamic language loading is handled by the application.
4. Add your language locale to Vue Timeago. Add it in `src/plugins/timeago.js` under `locales`.
   - Ref: https://github.com/egoist/vue-timeago#update-locale-globally
5. Submit a PR with your changes.
