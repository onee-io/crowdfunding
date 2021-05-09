<template>
    <div class="content">
        <h3> 新书众筹</h3>
        <span>以最低的价格获取我的新书 </span>

        <div class="status">
            <div v-if="!closed">已众筹资金：<b>{{ total }} ETH </b></div>
            <div v-if="closed"> 众筹已完成 </div>
            <div>众筹截止时间：{{ endDate }}</div>
        </div>

        <div v-if="joined" class="card-bkg">
            <div class="award-des">
                <span> 参与价格 </span>
                <b> {{ joinPrice }} ETH </b>
            </div>
            <button :disabled="closed" @click="withdraw">赎回</button>
        </div>

        <div v-if="!joined" class="card-bkg">
            <div class="award-des">
                <span> 当前众筹价格 </span>
                <b> {{ price }} ETH </b>
            </div>
            <button :disabled="closed" @click="join">参与众筹</button>
        </div>

        <div v-if="isAuthor">
            <button :disabled="closed" @click="withdrawFund"> 提取资金</button>
        </div>
    </div>
</template>

<script>
import Web3 from "web3";
import contract from "truffle-contract";
import crowd from '../../build/contracts/Crowdfunding.json';

export default {
    name: 'CrowdFund',
    // 定义上面HTML模板中使用的变量
    data() {
        return {
            price: null,
            total: 0,
            closed: true,
            joinPrice: null,
            joined: false,
            endDate: "null",
            isAuthor: true,
        }
    },

    // 当前Vue组件被创建时回调的hook 函数
    async created() {
        // 初始化 web3及账号
        await this.initWeb3Account()
        // 初始化合约实例
        await this.initContract()
        // 获取合约的状态信息
        await this.getCrowdInfo()
    },

    methods: {
        // 初始化web3及账号
        async initWeb3Account() {
            if (window.ethereum) {
                this.provider = window.ethereum;
                try {
                    // 请求用户授权
                    await window.ethereum.request({ method: 'eth_requestAccounts'}).then(function(accounts) {
                        console.log("用户账号", accounts)
                    });
                } catch (error) {
                    if (error.code === 4001) {
                        // 用户不授权时
                        console.error("User denied account access")
                    }
                }
            } else if (window.web3) {
                this.provider = window.web3.currentProvider;
            } else {
                alert('请安装MetaMask插件并解锁您的以太坊账户');
                // this.provider = new Web3.providers.HttpProvider("http://127.0.0.1:7545");
            }
            this.web3 = new Web3(this.provider);
            this.web3.eth.getAccounts().then(accounts  => {
                this.account = accounts[0]
            });
        },

        // 初始化合约实例
        async initContract() {
            const crowdContract = contract(crowd);
            crowdContract.setProvider(this.provider);
            this.crowdFund = await crowdContract.deployed();
        },

        // 获取合约的状态信息
        async getCrowdInfo() {
            // 获取合约的余额
            this.web3.eth.getBalance(this.crowdFund.address).then(result => {
                this.total = this.web3.utils.fromWei(result);
            });

            // 获取读者的参与金额
            this.crowdFund.joined(this.account).then(result => {
                if (result > 0) {
                    this.joined = true
                    this.joinPrice = this.web3.utils.fromWei(result)
                }
            });

            // 获取合约的关闭状态
            this.crowdFund.closed().then(result =>  this.closed = result);

            // 获取当前的众筹价格
            this.crowdFund.price().then(result => this.price = this.web3.utils.fromWei(result));

            // 获取众筹截止时间
            this.crowdFund.endTime().then(result => {
                console.log("截止时间", result)
                var endTime = new Date(result * 1000);
                // 把时间戳转化为本地时间
                this.endDate = endTime.toLocaleDateString().replace(/\//g, "-") + " " + endTime.toTimeString().substr(0, 8);
            });

            // 获取众筹创作者地址
            this.crowdFund.author().then(result => {
                if (this.account == result) {
                    this.isAuthor = true;
                } else {
                    this.isAuthor = false;
                }
            })
        },

        // 读者参与众筹
        join() {
            this.web3.eth.sendTransaction({
                from: this.account,
                to: this.crowdFund.address,
                value: this.web3.utils.toWei(this.price)
            }).then(() =>
                this.getCrowdInfo()
            );
        },

        // 读者赎回资金
        withdraw() {
            this.crowdFund.withdraw({
                from: this.account
            }).then(() => {
                this.getCrowdInfo()
            })
        },

        // 创作者提取资金
        withdrawFund() {
            this.crowdFund.withdrawFund({
                from: this.account
            }).then(() => {
                this.getCrowdInfo()
            });
        }
    }
}
</script>
