# 代码调试

## debugger

直接在代码中debugger, 查看此处各变量的值

```
  onChangePerformanceDate = (formValues) => {
    const { performanceId } = this.state
    const { transferAccountPeriod } = this.props
    debugger
    transferAccountPeriod(Object.assign({}, formValues, { performanceId }))
      .then(() => this.setState({ isShowAdjustPerformanceDateModal: false }))
  }
```

## react-developer-tools

react的调试插件，需要先在chrome中的扩展程序(chrome://extensions/)中添加;

在调试的时候可以在控制台利用Elements选中需要调试的组件，然后在React选项卡中，查看该组件的state和props，也可以选中某属性并双击修改该组件的属性。

![]('img/react-develop-tools-1.jpg')
