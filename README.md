# 众筹Dapp项目

教程链接：![https://learnblockchain.cn/2019/12/20/vue-dapp](https://learnblockchain.cn/2019/12/20/vue-dapp)

## 需求

假设我准备出版一本区块练技术书籍，但是不确定有多少人愿意购买这本书，于是我发起了一个众筹， 如果在一个月内，能筹集到10个ETH，我就进行写作，并给参与的读者每人赠送一本书，如果未能筹到足够的资金，参与的读者赎回之前投入的资金。

同时，为了让读者积极参与，初始时，参与众筹的价格非常低（0.02 ETH），每筹集满1个ETH时，价格上涨0.002ETH。

归纳出合约三个对外动作（函数）：

1. 汇款进合约，可通过实现合约的回退函数来实现。
2. 读者赎回汇款，这个函数仅仅在众筹未达标之后，由读者本人调用生效。
3. 创作者提取资金，这个函数需要在众筹达标之后，由创作者调用。

除此之外，还需要一个状态变量来保存以下状态：

1. 用户众筹的金额，使用一个mapping 来保存。
2. 当前众筹的价格，以及一个相应的内部函数逐步上涨价格。价格可以使用一个uint来保存。
3. 合约众筹的截止时间，用uint来保存， 在合约创建时往后追加30天，在构造函数中完成。
4. 记录合约众筹的收益者，即创作者，在合约创建时在构造函数中进行赋值，合约创建者就是创作者。
5. 再加入众筹状态，如果众筹停止阻止用户在参与。创作者提取资金时及时关闭众筹。

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
