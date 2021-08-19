安装依赖

webpack webpack-nano mini-html-webpack-plugin webpack-plugin-serve

webpack-nano ——— webpack-cli

webpack-plugin-serve ——— webpack-dev-server



```javascript
module.exports = {
  performance: {
    maxAssetSize: 50000,
    maxEntrypointSize: 50000,
    hints: 'error',
    assetFilter: function(assetFilename) {
      return !assetFilename.endsWith('.jpg');
    },
  }
};
```



