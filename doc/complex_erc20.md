## 一个复杂的ERC20合约
### By 古千峰 Jacky@BTCMedia、IPFSForce

ERC20标准合约非常简单，本教程介绍一个较为复杂的合约，通过此合约，可以执行以下工作：

* 锁定代币

* 解冻代币

* 查询账户冻结状态

* 充值以太坊自动发送代币

* 变更合约管理员

* 增发代币

* 按时间有计划冻结账户

* 按时间有计划解冻账户

* 批量发送代币

* 授权某账户代理操作授权人账户

* 变更授权信用额度

* 变更代币名称、代币符号

* 变更投资以太坊的下限和上限

* 自动判断达到硬顶后，将投资款原路返回

* 修改硬顶

* 修改已经销售的代币总量

* 设定并修改ETH与代币的兑换率

* 销毁合约与代币

### 具体的请看以下代码以及注释：

```
pragma solidity ^0.4.24;

import 'github.com/OpenZeppelin/zeppelin-solidity/contracts/token/ERC20/StandardToken.sol';

// ERC20 standard token
contract ERC20_B is StandardToken {
    address public admin; // 管理员
    string public name = "IPFS FORCE"; // 代币名称
    string public symbol = "IPFS"; // 代币符号
    uint8 public decimals = 18; // 代币精度
    uint256 public INITIAL_SUPPLY = 1000000000000000000000000000; // 总量10亿 *10^18

    // 同一个账户满足任意冻结条件均被冻结
    mapping (address => bool) public frozenAccount; //无限期冻结的账户
    mapping (address => uint256) public frozenTimestamp; // 有限期冻结的账户

    bool public exchangeFlag = true; // 代币兑换开启
    // 不满足条件或募集完成多出的eth均返回给原账户
    uint256 public minWei = 1;  //最低打 1 wei  1eth = 1*10^18 wei
    uint256 public maxWei = 2000000000000000000000; // 最多一次打 2000 eth
    uint256 public maxRaiseAmount = 20000000000000000000000; // 募集上限 20000 eth
    uint256 public raisedAmount = 0; // 已募集 0 eth
    uint256 public raiseRatio = 20000; // 兑换比例 1eth = 2万token
    // event 通知
    event Approval(address indexed owner, address indexed spender, uint256 value);
    event Transfer(address indexed from, address indexed to, uint256 value);

    // 构造函数
    constructor() public {
        //最常规的写法
        totalSupply_ = INITIAL_SUPPLY;
        admin = msg.sender;
        balances[msg.sender] = INITIAL_SUPPLY;
    }

    // fallback 向合约地址转账 or 调用非合约函数触发
    // 代币自动兑换eth
    function() public payable {
        require(msg.value > 0); //判断ETH金额是否大于0
        if (exchangeFlag) {     //是否开启兑换功能
            if (msg.value >= minWei && msg.value <= maxWei){                     //判断ETH金额是否在设定范围内
                if (raisedAmount < maxRaiseAmount) {                             //判断是否达到硬顶
                    uint256 valueNeed = msg.value;
                    raisedAmount = raisedAmount.add(msg.value);                  //添加已众筹金额
                    if (raisedAmount > maxRaiseAmount) {                         //如果该笔入账后，总融资额度达到硬顶
                        uint256 valueLeft = raisedAmount.sub(maxRaiseAmount);    //计算需要退回的金额
                        valueNeed = msg.value.sub(valueLeft);                    //valueNeed实际可入账
                        msg.sender.transfer(valueLeft);                          //将多打的资金退回给投资人
                        raisedAmount = maxRaiseAmount;                           //达到硬顶
                    }
                    if (raisedAmount >= maxRaiseAmount) {                        //如果达到硬顶，则关闭兑换功能
                        exchangeFlag = false;
                    }
                    // 已处理过精度 *10^18
                    uint256 _value = valueNeed.mul(raiseRatio);                  //计算代币额

                    require(_value <= balances[admin]);                          //判断发币方的代币是否都已经发完
                    balances[admin] = balances[admin].sub(_value);               //计算发币方账户余额
                    balances[msg.sender] = balances[msg.sender].add(_value);     //发送人的代币添加

                    emit Transfer(admin, msg.sender, _value);

                }
            } else {
                msg.sender.transfer(msg.value);                                  //不满足条件，原路返回
            }
        } else {
            msg.sender.transfer(msg.value);                                      //不满足条件，原路返回
        }
    }

    /**
    * 修改管理员
    */
    function changeAdmin(address _newAdmin) public returns (bool)  {
        require(msg.sender == admin);                                            //该操作必须是现在的admin，否则退出
        require(_newAdmin != address(0));                                        
        balances[_newAdmin] = balances[_newAdmin].add(balances[admin]);          //将admin账户的所有代币转给新的admin
        balances[admin] = 0;
        admin = _newAdmin;
        return true;
    }

    /**
    * 给指定账户增加代币，并做总量增发
    */
    function generateToken(address _target, uint256 _amount) public returns (bool)  {
        require(msg.sender == admin);                                            //该操作必须是现在的admin，否则退出
        require(_target != address(0));
        balances[_target] = balances[_target].add(_amount);                      //给指定账户发放代币
        totalSupply_ = totalSupply_.add(_amount);                                //总量增发
        INITIAL_SUPPLY = totalSupply_;
        return true;
    }

    // 从合约提现
    // 只能提给管理员
    function withdraw (uint256 _amount) public returns (bool) {
        require(msg.sender == admin);
        msg.sender.transfer(_amount);
        return true;
    }
    
    /**
    * 锁定账户
    */
    function freeze(address _target, bool _freeze) public returns (bool) {
        require(msg.sender == admin);
        require(_target != address(0));
        frozenAccount[_target] = _freeze;                                        //冻结账户
        return true;
    }

    /**
    * 通过时间戳锁定账户
    */
    function freezeWithTimestamp(address _target, uint256 _timestamp) public returns (bool) {
        require(msg.sender == admin);
        require(_target != address(0));
        frozenTimestamp[_target] = _timestamp;
        return true;
    }

    /**
    * 批量锁定账户
    */
    function multiFreeze(address[] _targets, bool[] _freezes) public returns (bool) {
        require(msg.sender == admin);
        require(_targets.length == _freezes.length);                             //确保账户数组与冻结状态数组长度一致
        uint256 len = _targets.length;
        require(len > 0);
        for (uint256 i = 0; i < len; i = i.add(1)) {
            address _target = _targets[i];
            require(_target != address(0));
            bool _freeze = _freezes[i];
            frozenAccount[_target] = _freeze;
        }
        return true;
    }
    
    /**
    * 批量通过时间戳锁定账户
    */
    function multiFreezeWithTimestamp(address[] _targets, uint256[] _timestamps) public returns (bool) {
        require(msg.sender == admin);
        require(_targets.length == _timestamps.length);
        uint256 len = _targets.length;
        require(len > 0);
        for (uint256 i = 0; i < len; i = i.add(1)) {
            address _target = _targets[i];
            require(_target != address(0));
            uint256 _timestamp = _timestamps[i];
            frozenTimestamp[_target] = _timestamp;
        }
        return true;
    }

    /**
    * 批量转账
    */
    function multiTransfer(address[] _tos, uint256[] _values) public returns (bool) {
        require(!frozenAccount[msg.sender]);                                     //发送人不在冻结账号中
        require(now > frozenTimestamp[msg.sender]);                              //当前时间已经超过按时间冻结的时间
        require(_tos.length == _values.length);
        uint256 len = _tos.length;
        require(len > 0);
        uint256 amount = 0;
        for (uint256 i = 0; i < len; i = i.add(1)) {
            amount = amount.add(_values[i]);                                     //累加需要转账的总额
        }
        require(amount <= balances[msg.sender]);                                 //转账人就是发起该指令的人
        for (uint256 j = 0; j < len; j = j.add(1)) {                             //为每个账户进行转账
            address _to = _tos[j];
            require(_to != address(0));
            balances[_to] = balances[_to].add(_values[j]);
            balances[msg.sender] = balances[msg.sender].sub(_values[j]);
            emit Transfer(msg.sender, _to, _values[j]);
        }
        return true;
    }

    /**
    * 从调用者转账至_to
    */
    function transfer(address _to, uint256 _value) public returns (bool) {
        require(!frozenAccount[msg.sender]);
        require(now > frozenTimestamp[msg.sender]);
        require(_to != address(0));
        require(_value <= balances[msg.sender]);

        balances[msg.sender] = balances[msg.sender].sub(_value);
        balances[_to] = balances[_to].add(_value);

        emit Transfer(msg.sender, _to, _value);
        return true;
    }
    
    /**
    * 从调用者作为from代理将from账户中的token转账至to
    * 调用者在from的许可额度中必须>=value
    */
    function transferFrom(address _from, address _to, uint256 _value) public returns (bool) {
        require(!frozenAccount[_from]);                                          //首先判断发送者账户没有被锁定
        require(now > frozenTimestamp[msg.sender]);                              //调用者账户已经解冻
        require(_to != address(0));                                              //不能是零账户
        require(_value <= balances[_from]);                                      //金额足够
        require(_value <= allowed[_from][msg.sender]);                           //_from账户授权给调用者的额度足够

        balances[_from] = balances[_from].sub(_value);
        balances[_to] = balances[_to].add(_value);
        allowed[_from][msg.sender] = allowed[_from][msg.sender].sub(_value);     //调整许可额度

        emit Transfer(_from, _to, _value);
        return true;
    }

    /**
    * 调整转账代理方spender的代理的许可额度
    */
    function approve(address _spender, uint256 _value) public returns (bool) {
        // 转账的时候会校验balances，该处require无意义
        // require(_value <= balances[msg.sender]);

        allowed[msg.sender][_spender] = _value;                                  //有调用人发起，授权_spender
        emit Approval(msg.sender, _spender, _value);                             //发起 Approval 事件
        return true;
    }
    
    /**
    * 增加转账代理方spender的代理的许可额度
    */
    function increaseApproval(address _spender, uint256 _addedValue) public returns (bool) {
        uint256 value_ = allowed[msg.sender][_spender].add(_addedValue);
        require(value_ <= balances[msg.sender]);
        allowed[msg.sender][_spender] = value_;

        emit Approval(msg.sender, _spender, value_);
        return true;
    }
    
    /**
    * 减少转账代理方spender的代理的许可额度
    */
    function decreaseApproval(address _spender, uint256 _subtractedValue) public returns (bool) {
        uint256 oldValue = allowed[msg.sender][_spender];
        if (_subtractedValue > oldValue) {
            allowed[msg.sender][_spender] = 0;
        } else {
            uint256 newValue = oldValue.sub(_subtractedValue);
            require(newValue <= balances[msg.sender]);
            allowed[msg.sender][_spender] = newValue;
        }

        emit Approval(msg.sender, _spender, allowed[msg.sender][_spender]);
        return true;
    }

    /**
    * 查询账户是否存在锁定时间戳
    */
    function getFrozenTimestamp(address _target) public view returns (uint256) {
        require(_target != address(0));
        return frozenTimestamp[_target];
    }

    /**
     * 查询账户是否被锁定
     */
    function getFrozenAccount(address _target) public view returns (bool) {
        require(_target != address(0));
        return frozenAccount[_target];
    }

    /**
     * 查询合约的余额
     */
    function getBalance() public view returns (uint256) {
        return address(this).balance;                                            //this是指合约，不是balances[admin]
    }

    /**
     * 修改代币名称name
     */
    function setName (string _value) public returns (bool) {
        require(msg.sender == admin);
        name = _value;
        return true;
    }
    
    /**
     * 修改代币symbol
     */
    function setSymbol (string _value) public returns (bool) {
        require(msg.sender == admin);
        symbol = _value;
        return true;
    }

    /**
     * 修改自动兑换状态
     */
    function setExchangeFlag (bool _flag) public returns (bool) {
        require(msg.sender == admin);
        exchangeFlag = _flag;
        return true;
    }

    /**
     * 修改单笔募集下限
     */
    function setMinWei (uint256 _value) public returns (bool) {
        require(msg.sender == admin);
        minWei = _value;
        return true;
    }
    
    /**
     * 修改单笔募集上限
     */
    function setMaxWei (uint256 _value) public returns (bool) {
        require(msg.sender == admin);
        maxWei = _value;
        return true;
    }
    
    /**
     * 修改总募集上限
     */
    function setMaxRaiseAmount (uint256 _value) public returns (bool) {
        require(msg.sender == admin);
        maxRaiseAmount = _value;
        return true;
    }

    /**
     * 修改已募集数
     */
    function setRaisedAmount (uint256 _value) public returns (bool) {
        require(msg.sender == admin);
        raisedAmount = _value;
        return true;
    }

    /**
     * 修改募集比例
     */
    function setRaiseRatio (uint256 _value) public returns (bool) {
        require(msg.sender == admin);
        raiseRatio = _value;
        return true;
    }

    /**
     * 销毁合约
     */
    function kill() public {
        require(msg.sender == admin);
        selfdestruct(admin);
    }
}
```

以上合约已经部署在Ropsten测试网络上，网址：https://ropsten.etherscan.io/address/0x039cc582ba306524c85989b583cfa23f8041329f

合约地址:0x039Cc582Ba306524c85989B583cFA23f8041329F
