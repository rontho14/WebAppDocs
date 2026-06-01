# UIExplorer / RNTester — Verified Code Snippets

Every excerpt below was fetched from `raw.githubusercontent.com` (verbatim source).
Each snippet lists its exact source path and raw URL. Snippets are trimmed for
length; ellipses (`...`) mark omissions.

---

## Classic UIExplorer — `rnplay/uiexplorer`, branch `master`

### 1. The example registry (two arrays of modules)

**Path:** `UIExplorerList.js`
**Raw:** https://raw.githubusercontent.com/rnplay/uiexplorer/master/UIExplorerList.js

```js
var COMPONENTS = [
  require('./ActivityIndicatorIOSExample'),
  require('./ImageExample'),
  require('./ListViewExample'),
  require('./Navigator/NavigatorExample'),
  require('./ScrollViewExample'),
  require('./TextExample.ios'),
  require('./ViewExample'),
  require('./WebViewExample'),
  // ...
];

var APIS = [
  require('./ActionSheetIOSExample'),
  require('./AlertIOSExample'),
  require('./GeolocationExample'),
  require('./PanResponderExample'),
  require('./TimerExample'),
  // ...
];
```

This *is* the gallery's catalog. Demonstrates the core registry pattern: the menu
is a flat data array, decoupled from file layout (note the nested
`./Navigator/NavigatorExample`). **Kiosk:** add one line to expose a new feature
panel; list/search/navigation pick it up with no other wiring.

### 2. Turning a module into a renderable page

**Path:** `UIExplorerList.js`
**Raw:** https://raw.githubusercontent.com/rnplay/uiexplorer/master/UIExplorerList.js

```js
function makeRenderable(example: any): ReactClass<any, any, any> {
  return example.examples ?
    createExamplePage(null, example) :
    example;
}
```

A module is "self-describing": if it has an `examples` array it is wrapped by the
generic `createExamplePage`; otherwise it renders itself. **Kiosk:** the host
renders panels generically from data instead of importing each screen explicitly.

### 3. List rows + list → detail navigation

**Path:** `UIExplorerList.js`
**Raw:** https://raw.githubusercontent.com/rnplay/uiexplorer/master/UIExplorerList.js

```js
_renderRow(example: ExampleModule, i: number) {
  return (
    <View key={i}>
      <TouchableHighlight onPress={() => this._onPressRow(example)}>
        <View style={styles.row}>
          <Text style={styles.rowTitleText}>{example.title}</Text>
          <Text style={styles.rowDetailText}>{example.description}</Text>
        </View>
      </TouchableHighlight>
      <View style={styles.separator} />
    </View>
  );
}

_onPressRow(example: ExampleModule) {
  if (example.external) {
    this.props.onExternalExampleRequested(example);
    return;
  }
  var Component = makeRenderable(example);
  this.props.navigator.push({ title: Component.title, component: Component });
}
```

Rows are rendered straight from registry metadata (`title`/`description`); a tap
pushes the rendered module onto the navigator. **Kiosk:** a single generic index
screen drives navigation to every feature.

### 4. Live search/filter over the registry

**Path:** `UIExplorerList.js`
**Raw:** https://raw.githubusercontent.com/rnplay/uiexplorer/master/UIExplorerList.js

```js
_search(text: mixed) {
  var regex = new RegExp(text, 'i');
  var filter = (component) => regex.test(component.title);

  this.setState({
    dataSource: ds.cloneWithRowsAndSections({
      components: COMPONENTS.filter(filter),
      apis: APIS.filter(filter),
    }),
    searchText: text,
  });
  Settings.set({searchText: text});
}
```

Filtering is just a regex over registry `title`s — no per-screen search code.
**Kiosk:** free "jump to function" search over a large menu.

### 5. The generic example block (shared scaffolding)

**Path:** `UIExplorerBlock.js`
**Raw:** https://raw.githubusercontent.com/rnplay/uiexplorer/master/UIExplorerBlock.js

```js
propTypes: {
  title: React.PropTypes.string,
  description: React.PropTypes.string,
},

render: function() {
  var description;
  if (this.props.description) {
    description = <Text style={styles.descriptionText}>{this.props.description}</Text>;
  }
  return (
    <View style={styles.container}>
      <View style={styles.titleContainer}>
        <Text style={styles.titleText}>{this.props.title}</Text>
        {description}
      </View>
      <View style={styles.children}>{this.props.children}</View>
    </View>
  );
}
```

One reusable "card" that renders title + optional description + children. **Kiosk:**
consistent panel chrome across every feature, defined once.

### 6. A self-describing example module

**Path:** `ViewExample.js`
**Raw:** https://raw.githubusercontent.com/rnplay/uiexplorer/master/ViewExample.js
(metadata reproduced; render bodies summarized for length)

```js
exports.title = '<View>';
exports.description = 'Basic building block of all UI.';
exports.displayName = 'ViewExample';
exports.examples = [
  { title: 'Background Color', render() { /* blue bg + Text */ } },
  { title: 'Border',           render() { /* 5px blue border */ } },
  { title: 'Border Radius',    render() { /* rounded box */ } },
  // ... Padding/Margin, Circle, Overflow, Opacity
];
```

The contract every registry entry follows: `title` + `description` + `examples[]`
of `{title, render}`. **Kiosk:** describe a feature screen as data; the framework
renders it.

### 7. Root registration delegating to the list

**Path:** `index.ios.js` (module `UIExplorerApp`)
**Raw:** https://raw.githubusercontent.com/rnplay/uiexplorer/master/index.ios.js

```js
var UIExplorerApp = React.createClass({
  render: function() {
    return <UIExplorerList/>;
  }
});

AppRegistry.registerComponent('UIExplorerApp', () => UIExplorerApp);
```

The app shell is trivial — it just hands off to the registry-driven list.

---

## Modern RNTester — `facebook/react-native`, branch `main`

### 8. The typed registry entry

**Path:** `packages/rn-tester/js/utils/RNTesterList.ios.js`
**Raw:** https://raw.githubusercontent.com/facebook/react-native/main/packages/rn-tester/js/utils/RNTesterList.ios.js

```js
// Components entries (type RNTesterModuleInfo: {key, module, category?})
{
  key: 'ImageExample',
  module: require('../examples/Image/ImageExample'),
  category: 'Basic',
},
// APIs entries
{
  key: 'AlertExample',
  module: require('../examples/Alert/AlertExample').default,
  category: 'UI',
},

const RNTesterList = { APIs, Components, Modules };
module.exports = RNTesterList;
```

Same two-bucket registry as classic, now with a stable `key` and a `category` for
grouping, plus a `Modules` key→module map for lookup/deep-linking. **Kiosk:**
`category` groups menu items into sections automatically.

### 9. The formal example-module contract (types)

**Path:** `packages/rn-tester/js/types/RNTesterTypes.js`
**Raw:** https://raw.githubusercontent.com/facebook/react-native/main/packages/rn-tester/js/types/RNTesterTypes.js

```js
export type RNTesterModuleExample = Readonly<{
  name?: string,
  title: string,
  platform?: 'ios' | 'android',
  description?: string,
  hidden?: boolean,
  scrollable?: boolean,
  render: component(),
}>;

export type RNTesterModule = Readonly<{
  title: string,
  description: string,
  displayName?: ?string,
  documentationURL?: ?string,
  category?: ?string,
  examples: Array<RNTesterModuleExample>,
  showIndividualExamples?: boolean,
}>;
```

The informal classic contract, now a checked Flow type. **Kiosk:** a typed panel
contract makes a registry-driven gallery safe to extend by many contributors.

### 10. A modern example module conforming to the contract

**Path:** `packages/rn-tester/js/examples/View/ViewExample.js`
**Raw:** https://raw.githubusercontent.com/facebook/react-native/main/packages/rn-tester/js/examples/View/ViewExample.js

```js
export default {
  title: 'View',
  documentationURL: 'https://reactnative.dev/docs/view',
  category: 'Basic',
  description: 'Basic building block of all UI, ...',
  displayName: 'ViewExample',
  examples: [
    {
      title: 'Background Color',
      name: 'background-color',
      render(): React.Node {
        return (
          <View testID="view-test-background-color"
                style={{backgroundColor: '#527FE4', padding: 5}}>
            <RNTesterText style={{fontSize: 11}}>Blue background</RNTesterText>
          </View>
        );
      },
    },
    // { title: 'Border', name: 'border', render() {...} }, ...
  ],
} as RNTesterModule;
```

The evolved `ViewExample`: same `title`/`description`/`examples[]` shape, now with
`documentationURL`, per-example `name`, and `testID`s for automated testing.
**Kiosk:** self-contained feature panels that are also test-addressable for
unattended regression checks.
